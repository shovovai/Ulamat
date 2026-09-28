# Data and API

## 1. What is stored where

| Data | Location | Stored permanently? |
|---|---|---|
| Duas, lessons, rulings, quizzes | `/content` TypeScript files | Yes, in the code |
| Dua reference audio | `/public/audio` | Yes, in the code |
| Quran text, translations, tafsir | Quran Foundation API / QuranEnc | No, cached by Next.js for 30 days |
| Quran audio | Quran CDN (streamed) | No |
| Users, progress, scores | Supabase Postgres | Yes |
| User voice recordings | Nowhere | **Never stored** |

---

## 2. Supabase schema

```sql
-- profiles: one row per user (extends auth.users)
create table profiles (
  id uuid primary key references auth.users on delete cascade,
  display_name text,
  ui_language text not null default 'en',
  madhab text check (madhab in ('hanafi','shafii','maliki','hanbali')), -- null = not sure
  script text not null default 'uthmani' check (script in ('uthmani','indopak')),
  gender text check (gender in ('male','female')),                       -- null = prefer not to say
  user_type text not null default 'born_muslim'
    check (user_type in ('born_muslim','revert','parent','exploring')),
  goals text[] not null default '{}',
  arabic_level text not null default 'none' check (arabic_level in ('none','slow','fluent')),
  daily_goal_min int not null default 10,
  reminder_slot text default 'after_isha',
  timezone text not null default 'UTC',
  unlocked_level2 text[] not null default '{}',   -- e.g. {'zakat','hajj'}
  reciter_id int,
  translation_id int,
  tafsir_id int,
  role text not null default 'user' check (role in ('user','admin')),
  created_at timestamptz not null default now()
);

-- progress per content item
create table item_progress (
  user_id uuid references profiles on delete cascade,
  item_id text not null,                 -- matches content file `id`
  quiz_score int,                        -- 0-100, best recent
  recite_score int,                      -- 0-100, best recent
  best_recite_score int,
  attempts int not null default 0,
  placement_passed boolean not null default false,
  last_practiced timestamptz,
  next_review date,
  interval_days int not null default 1,
  primary key (user_id, item_id)
);

-- lessons completed
create table lesson_progress (
  user_id uuid references profiles on delete cascade,
  lesson_id text not null,
  completed_at timestamptz not null default now(),
  primary key (user_id, lesson_id)
);

-- recitation attempts (NO audio)
create table recitation_attempts (
  id bigint generated always as identity primary key,
  user_id uuid references profiles on delete cascade,
  item_id text not null,
  score int,                             -- null when "not sure"
  confidence text check (confidence in ('high','medium','low')),
  word_results jsonb not null,           -- [{ "word": "...", "status": "correct" }]
  created_at timestamptz not null default now()
);

-- daily activity for streaks and goals
create table daily_activity (
  user_id uuid references profiles on delete cascade,
  day date not null,
  minutes int not null default 0,
  goal_met boolean not null default false,
  primary key (user_id, day)
);

create table streaks (
  user_id uuid primary key references profiles on delete cascade,
  current int not null default 0,
  longest int not null default 0,
  last_active_day date,
  freezes_left int not null default 2,
  freezes_reset_on date
);

create table quran_bookmarks (
  user_id uuid references profiles on delete cascade,
  surah int not null,
  ayah int not null,
  created_at timestamptz not null default now(),
  primary key (user_id, surah, ayah)
);

create table quran_last_read (
  user_id uuid primary key references profiles on delete cascade,
  surah int not null,
  ayah int not null,
  updated_at timestamptz not null default now()
);

create table badges (
  user_id uuid references profiles on delete cascade,
  badge_id text not null,
  earned_at timestamptz not null default now(),
  primary key (user_id, badge_id)
);

create table content_reports (
  id bigint generated always as identity primary key,
  user_id uuid references profiles on delete set null,
  item_id text,
  quran_ref text,                        -- e.g. "67:1" for reader reports
  kind text not null check (kind in ('text','audio','translation','ruling','other')),
  note text,
  app_version text,
  status text not null default 'open' check (status in ('open','fixed','rejected')),
  created_at timestamptz not null default now()
);

create table push_subscriptions (
  user_id uuid references profiles on delete cascade,
  endpoint text primary key,
  keys jsonb not null,
  created_at timestamptz not null default now()
);
```

### Row Level Security

Enable RLS on every table. Users can only read and write their own rows:

```sql
alter table item_progress enable row level security;
create policy "own rows" on item_progress
  for all using (auth.uid() = user_id) with check (auth.uid() = user_id);
-- repeat for every user table (profiles uses id instead of user_id)

-- content_reports: users can insert, only admins can read
create policy "insert own report" on content_reports
  for insert with check (auth.uid() = user_id);
create policy "admins read reports" on content_reports
  for select using (exists (select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'));
```

### Knowledge Score

Computed on the server from `item_progress` + `/content` (weights live in `content/sections.ts`). Not stored as a table; computed on request and cached per user for 5 minutes.

---

## 3. Next.js API routes

| Method | Route | Purpose |
|---|---|---|
| `POST` | `/api/recite` | Receive audio + `itemId` (or `surah`+`ayah`), call speech endpoint, align, score, save attempt, return results |
| `POST` | `/api/progress/lesson` | Mark lesson complete, update item progress, streak, daily activity |
| `POST` | `/api/progress/quiz` | Save quiz result for an item |
| `GET` | `/api/score` | Knowledge Score + section breakdown |
| `GET` | `/api/review` | Items due today + weak items |
| `POST` | `/api/onboarding` | Save onboarding answers (+ merge guest progress) |
| `POST` | `/api/report` | Create a content report |
| `POST` | `/api/push/subscribe` | Save push subscription |
| `GET` | `/api/cron/reminders` | Hourly GitHub Actions trigger (`reminders.yml`): send due reminders |
| `DELETE` | `/api/account` | Delete user and all data |
| `GET` | `/api/account/export` | Download all personal data as JSON |

Quran data is loaded directly in **server components** (no public API route needed), using `lib/quran-api.ts`.

### `/api/recite` contract

Request: `multipart/form-data`

| Field | Type | Notes |
|---|---|---|
| `audio` | WAV, 16 kHz mono | ≤ 60 s, ≤ 2 MB |
| `itemId` | string | For duas/surah refs from content |
| `surah`, `ayah` | number | For Quran reader practice |

Response:

```json
{
  "score": 86,
  "confidence": "high",
  "words": [
    { "expected": "التَّحِيَّاتُ", "status": "correct" },
    { "expected": "لِلَّهِ", "status": "correct" },
    { "expected": "وَالصَّلَوَاتُ", "status": "wrong", "heard": "والصلات" },
    { "expected": "وَالطَّيِّبَاتُ", "status": "missed" }
  ],
  "extra": [{ "afterIndex": 1, "heard": "..." }],
  "tips": ["Listen to وَالصَّلَوَاتُ slowly, then try again."]
}
```

If confidence is low: `{ "score": null, "confidence": "low", "message": "not_clear" }`.

Rate limit: 30 recitation checks per user per hour (protects speech endpoint costs).

---

## 4. Speech service

- **Model:** `tarteel-ai/whisper-base-ar-quran` (Apache-2.0), later a version fine-tuned on dua recordings.
- **Hosting:** Hugging Face Inference Endpoint (GPU, scale-to-zero). Compute only: no audio is stored.
- **Call:** server-side from `/api/recite` with `HF_ENDPOINT_URL` and `HF_TOKEN`.
- **Cold start:** scale-to-zero means the first request after idle can take 10–30+ seconds. Options: keep minimum 1 replica during peak hours, or show "Warming up…" on the first check.
- **Confidence:** derived from the model's average token log-probability; thresholds tuned during beta.

---

## 5. External Quran APIs

### Quran Foundation Content API (primary)

- Official API behind Quran.com: verses, translations, tafsir, recitations, word-by-word data.
- Auth: OAuth2 client credentials (`scope=content`), token cached until expiry. Headers `x-auth-token` + `x-client-id`.
- Server-only: credentials must never be sent to the browser.
- Word audio: each word returns a relative path (e.g. `wbw/001_001_001.mp3`) served from `https://audio.qurancdn.com/`.
- Read the Developer Terms before enabling offline audio caching or any commercial plan.
- The old unauthenticated `api.quran.com/api/v4` is being replaced; use the new base URL.

### QuranEnc (translations and short tafsir)

- Scholar-supervised translations in 60+ languages, each labeled with translator/institution.
- Al-Mukhtasar (abridged tafsir) in multiple languages; good default for learners.
- Available via API and downloadable formats. Check terms and credit requirements.

### AlQuran.cloud (prototype only)

- Free, no key. Use while waiting for Quran Foundation access. Replace before launch.

### Source selection

The scholar board chooses which translations and tafsirs are enabled per language. The enabled list lives in `content/quran-sources.ts`:

```ts
export const quranSources = {
  translations: { en: [/* ids */], bn: [/* ids */] },
  tafsirs: { en: [/* ids */] },
  reciters: [/* ids: Husary, Minshawi, ... */],
};
```

---

## 6. Caching

| Data | Cache |
|---|---|
| Quran text/translation/tafsir | `fetch` with `revalidate: 2592000` (30 days) |
| API token | In memory, until expiry − 60 s |
| Content files | Built into the app at deploy |
| Knowledge Score | 5 minutes per user |
| PWA | Service worker caches app shell, content, fonts, played dua audio, visited surah pages |

---

## 7. Environment variables

`.env.example` is the full list, with a line on what each one does. The ones a deployment cannot
do without:

| | Without it |
| --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY` | No accounts, no sync, no admin panel; the app runs on the device alone |
| `HF_ENDPOINT_URL`, `HF_TOKEN` | Recitation is never scored: the screen says checking is not set up |
| `NEXT_PUBLIC_VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, `VAPID_SUBJECT` | No reminders in a browser |
| `FCM_SERVICE_ACCOUNT` | No reminders in the apps (`docs/PUSH.md`) |
| `CRON_SECRET` | The reminder jobs cannot authenticate; also a GitHub secret, with `APP_URL` |
| `REVALIDATE_SECRET` | Publishing from the admin panel does not refresh the site |
| `MAINTENANCE_BYPASS_SECRET` | Maintenance mode locks staff out too |
| `IP_HASH_SALT` | Form abuse counting falls back to `CRON_SECRET` as its salt |
| `QF_CLIENT_ID`, `QF_CLIENT_SECRET` | Quran text comes from the public API, then the bundled copy |
| `SENTRY_DSN`, `NEXT_PUBLIC_POSTHOG_KEY` | Errors go to the console; no analytics |
| `ANDROID_CERT_FINGERPRINTS`, `ANDROID_PACKAGE`, `APPLE_APP_ID` | Links open in the browser, not the app |

---

## 8. Privacy and security checklist

- [ ] Audio never logged (disable body logging on `/api/recite`, including in Sentry).
- [ ] RLS enabled on every table; tested with a second user account.
- [ ] Service role key used only in server code.
- [ ] Account deletion removes all rows (cascade) and push subscriptions.
- [ ] Privacy policy states: what is stored, what is not (voice), third parties (Supabase, Hugging Face, Vercel, Quran Foundation).
- [ ] Kids profiles: no recitation attempt history kept beyond scores.

## 9. Motivation and daily tools (migration 0012)

| Table | Columns | RLS |
|---|---|---|
| `iman_checkins` | user_id, day, mood (strong/okay/struggling) | own rows |
| `buddies` **(deprecated)** | user_a, user_b, status, created_at | own rows (either side reads) |
| `buddy_invites` **(deprecated)** | token, inviter, used_by, created_at | inviter; accept via `accept_buddy_invite(t)` (security definer, same gender) |
| `buddy_nudges` **(deprecated)** | id, from_user, to_user, preset (1–3), created_at | sender inserts, receiver reads |
| `sadaqah_entries` | id, user_id, day, amount, note (≤ 300 chars) | own rows |
| `gratitude_notes` | user_id, day, notes (≤ 3) | own rows |
| `salah_checkins` | (existing) | + buddy may read today's rows only (the buddy policy is deprecated with the buddy tables) |

New columns: `profiles.minimum_mode`, `profiles.minimum_since`; `push_subscriptions.lat, lng, prayer_settings, nudge_prayers, quiet_start, quiet_end, nudge_cursor, adhan_sent, fast_prefs, fast_sent`; `slot` may be `'none'` (adhan-only).

**Routes:** `GET /api/cron/adhan` (Bearer `CRON_SECRET`, every 5 min from `.github/workflows/adhan.yml`, needs GitHub secrets `APP_URL` and `CRON_SECRET`). `POST /api/push/subscribe` also takes prayer prefs.

## 10. Pronunciation (Part 14, migration 0013)

- `recitation_attempts` gains `serious int`, `letter_issues text[]` (codes like `H>h`, `v:a>u`, `madd`, `missing`) and `engine` (`word`/`letter`). Still never audio.
- `item_progress.pron_mastered` — an attempt scored ≥ 90 with no serious issue.
- `POST /api/recite` returns `{ score, confidence, words, extra, tips, pron }` where `pron` is the shared
  `PronunciationResult` (`lib/pronunciation/types.ts`).
- Env: `PRONUNCIATION_ENGINE` (`word` default, `letter` for Phase 2), `PHONEME_ENDPOINT_URL`,
  `PRONUNCIATION_LETTER_ITEMS` (optional allow-list).

## 11. Salah Verified (Part 16, migration 0014) — DEPRECATED

> **Deprecated 2026-09-26.** The feature was removed from the app, website and admin; its API routes were deleted. The tables below are **kept, not dropped** (no data is deleted). Nothing reads or writes them. Drop them only after a separate, explicit decision.


| Table | Columns | RLS |
|---|---|---|
| `teachers` | user_id, display_name, gender, languages, approved_by, active | teacher reads own row; admins manage |
| `verifications` | id, user_id, teacher_id, level, status, verdict, notes jsonb, first_name, learner_gender, madhab, parts, teacher_name, revoked, created_at, completed_at | learner reads own; teachers read assigned + unassigned of their gender; teachers update review columns only |
| `review_recordings` | id, verification_id, user_id, part, mime, audio bytea (≤ 1.5 MB), created_at, expires_at (+14 d) | learner inserts/reads/deletes own; assigned teacher reads/deletes |

Functions: `public_verification(id)` (anon: first name, level, date if valid and not revoked), `revoke_verification`, `cancel_verification`, `delete_expired_recordings`.

Routes: `POST /api/verify/submit` (consent + parts), `GET /api/verify/mine`, `POST /api/verify/revoke`, `GET /api/teach/audio?v=&part=` (assigned teacher, no-store), `GET /api/certificate/[id]` (owner PNG), `GET /api/cron/recordings` (daily, `CRON_SECRET`).

## 12. Maktab and children (Part 17, migration 0015) — DEPRECATED

> **Deprecated 2026-09-26.** Family & classes (child profiles, child mode, parent dashboard, teacher panel, maktab classes) were removed. Tables kept, not dropped; nothing reads or writes them. The weekly family cron and `/api/family/*` routes were deleted.


| Table | Columns | RLS |
|---|---|---|
| `child_profiles` | id, parent_id, name, age, script, language, mastery, created_at, updated_at | parent owns; teacher of a class with the child reads |
| `classes` | id, teacher_id, name, join_code | teacher owns; parents of members read |
| `class_members` | class_id, child_id, parent_id, joined_at | parent / teacher read; insert only via `join_class(code, child)` (parent's approval) |
| `assignments` | + class_id, item_id (lesson_id or item_id) | teacher manages class ones; parents of members read |
| `assignment_progress` | assignment_id, child_id, status, score | parent writes; teacher reads |
| `child_attempts` | child_id, item_id, score, serious, letter_issues | parent writes; teacher reads (never audio) |

Routes: `POST /api/family/child-sync` (attempt, mastery, assignment status), `GET /api/cron/family-weekly` (Fridays).
Local only: children list (`ilm-children-v1`), each child's progress (`ilm-state-v1:child:<id>`), the parent PIN hash.

## 13. Invite to learn (migration 0020)

| Table / column | Columns | RLS |
|---|---|---|
| `profiles.referral_code` | text, unique, `^[a-z0-9]{8}$` | own profile row |
| `referrals` | referrer_id, referred_id (PK), created_at, first_lesson_at | RLS on, **no policies**: nobody reads rows directly |

Functions (security definer, `authenticated` only):
- `my_referral_code()` → your code, created on first use.
- `claim_referral(code)` → called once after sign-in with the `ilm-ref` cookie; counts only accounts created in the last 7 days, once, never yourself.
- `my_referral_counts()` → `{ joined, first_lesson }` for the caller. **Counts only — the referrer never sees who.**
- Trigger `referral_first_lesson` on `lesson_progress` insert sets `first_lesson_at`.
- `admin_referrals(days)` → per-day totals (joined, first lessons) for analysts.

Route: `GET /i/[code]` sets the `ilm-ref` cookie (30 days) and redirects to `/`.

## 15. Security review fixes (migration 0021)

From the 2026-09-26 review:

- `profiles` is no longer `FOR ALL`: separate select/insert/update policies, no delete (accounts go through `/api/account`, which cascades from `auth.users`). A `profiles_server_columns` trigger keeps `created_at` and `referral_code` server-owned, so neither `claim_referral`'s 7-day window nor the admin metrics can be rewritten by the account itself.
- `my_referral_code()` returns null when the caller has no profiles row and retries at most 10 times (it could loop forever before).
- `scholars`: a scholar edits only their bio, and only while active. `active`, `name`, `qualifications` and `madhab` are admin-owned (`scholars_admin_columns` trigger). Editing an answer now also requires `is_answering_scholar()`, so deactivating a scholar removes write access to published content.
- `push_subscriptions.endpoint` must be a real push service (`push_endpoint_allowed`), enforced again in the route by `lib/push-endpoint.ts`: the cron jobs POST to whatever is stored, so an arbitrary URL would have let anyone aim the server at a host of their choosing. `/api/push/subscribe` is rate limited, and DELETE removes only the caller's own row (or an unclaimed one).

## 16. Native push (migration 0024)

An Android web view has no Web Push, so the apps register an FCM token instead of a subscription.

- `device_tokens` — `token` (primary key), `user_id`, `platform` (`android`/`ios`), and then the same
  columns as `push_subscriptions` minus `keys`. RLS is on with **no policies**: only the service role
  touches it, so a token cannot be read or guessed from the client.
- `POST /api/push/device` `{ token, platform, locale, slot, tz, … }` — signed in only, rate limited
  to 30 an hour per account. `DELETE` removes only the caller's own token.
- The two cron jobs each run twice: once over `push_subscriptions` through `web-push`, once over
  `device_tokens` through FCM HTTP v1 (`lib/fcm.ts`). Because the columns match, the queries alias
  `endpoint:token` and `reminderDue` / `adhanDue` / `fastDue` are unchanged. A token FCM reports as
  `UNREGISTERED` is deleted, the same as a `410` from a browser's push service.
- Env: `FCM_SERVICE_ACCOUNT` — the service-account JSON from the Firebase console, raw or base64.
  Without it the native half is skipped entirely and the browser half is unaffected. Setup:
  `docs/PUSH.md`.
- Shared validation for both routes lives in `lib/push-prefs.ts`.

## 14. Deprecated tables (kept, not dropped)

`teachers`, `verifications`, `review_recordings` (0014); `child_profiles`, `classes`, `class_members`, `assignments` (class columns), `assignment_progress`, `child_attempts` (0015); `buddies`, `buddy_invites`, `buddy_nudges` (0012). Their functions (`public_verification`, `join_class`, `lookup_class`, `accept_buddy_invite`, …) stay in the database but nothing calls them.

## Website and admin API

| Route | What |
|---|---|
| `POST /api/site/contact`, `/api/site/volunteer` | Forms (honeypot, fill time, rate limit, IP hash) → `contact_messages`, `volunteer_applications` |
| `GET /api/site/image/[id]` | Admin-uploaded WebP from `site_images` |
| `GET /api/og?t&l&k` | Open Graph image (HarfBuzz-shaped titles) |
| `GET /api/legal/status`, `POST /api/legal/accept` | Changed legal documents; record acceptance |
| `POST /api/revalidate` | Admin → drop website content cache (Bearer `REVALIDATE_SECRET`) |
| `/sitemap.xml`, `/robots.txt` | All website pages × locales |

Tables: migrations `0018_site_cms.sql`, `0019_staff_admin.sql` (see UPGRADE_REPORT). Staff RLS requires `aal2` + role via `has_staff_role()`.
