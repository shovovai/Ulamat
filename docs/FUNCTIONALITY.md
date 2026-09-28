# Functionality

How each feature works: steps, rules, and edge cases.

---

## 1. Onboarding

### Flow

```
Welcome → Language → Who are you? → Goal → Arabic level → Placement check
→ Madhab → Script → Gender → Daily time + reminder → Your starting point
→ First lesson → Sign-up prompt
```

### Rules

- Every screen after "Language" has a **Skip** that applies the default.
- A progress bar shows position (10 steps).
- All answers are saved to local storage immediately, then to `profiles` after sign-up.
- The whole flow must be completable in under 2 minutes.

| Step | Options | Default if skipped | Effect |
|---|---|---|---|
| Language | English, Bengali, Urdu, Arabic… | Device language | UI + translations |
| Who are you? | Born Muslim / New to Islam / Learning for my child / Exploring Islam | Born Muslim | Revert → start from zero, extra explanations. Parent → prompt to create child profile. Exploring → Aqeedah unit first |
| Goal (max 2) | Fix my salah / Learn daily duas / Memorize Quran / Prepare for Hajj / Teach my kids | Fix my salah | Orders the first units |
| Arabic level | Can't read / Read slowly / Read fluently | Can't read | Can't read → Qaida course first + transliteration on |
| Placement check | Recite Al-Fatiha + 5 questions | Skipped = score 0, no penalty | Sets starting Knowledge Score and skips known lessons |
| Madhab | Hanafi / Shafi'i / Maliki / Hanbali / Not sure | Not sure | Filters madhab-specific items |
| Script | Uthmani / IndoPak (shown as images) | Uthmani | Quran + dua rendering |
| Gender | Male / Female / Prefer not to say | Prefer not to say | Shows gender-specific rulings; "prefer not to say" shows both |
| Daily time | 5 / 10 / 15 / 20 min | 10 | Daily goal in XP-free minutes |
| Reminder | After Fajr / Morning / After Asr / After Isha / None | After Isha | Notification schedule |

### Placement check details

1. Show the microphone privacy note: "Your voice is checked and then discarded. We never save recordings."
2. Request microphone permission. If denied → skip recitation part, show quiz only.
3. User recites Al-Fatiha → word-level result.
4. Feedback copy is always encouraging: "Beautiful. Here are 3 things to polish." Never "12 mistakes."
5. 5 multiple-choice questions (wudu, salah basics).
6. Items answered/recited correctly are marked `placement_passed` and skipped in the path (user can still open them).

---

## 2. Learning path and lesson engine

### Structure

```
Unit (e.g. "Salah: the recitations")
 └─ Lesson (e.g. "Tashahhud")
     └─ Steps (5–10 per lesson)
```

### Step types

| Type | What the user does | Pass condition |
|---|---|---|
| `learn` | Reads a card: Arabic, meaning, source | Tap Continue |
| `listen` | Plays qari audio, word highlighting follows | Audio played once |
| `repeat` | Listens to a phrase, then recites that phrase | Word score ≥ 70% |
| `recite` | Recites the full item | Word score ≥ 80% (≥ 90% for Al-Fatiha) |
| `quiz` | Multiple choice | Correct answer |
| `match` | Match Arabic phrase ↔ meaning | All pairs correct |
| `order` | Tap words into the correct order | Correct order |

### Rules

- A wrong quiz answer shows the explanation, then the question returns at the end of the lesson.
- A failed recitation offers: **Listen again**, **Try again**, **Slow mode**. After 3 attempts, **Continue anyway** appears. The item gets flagged as weak and enters the review queue.
- No hearts or lives. Mistakes never block learning.
- Lesson complete → progress saved, streak updated, next lesson unlocks.
- Items the user's madhab doesn't use are hidden. With "Not sure", shared items show and differing items show a small "differs by madhab" note.

---

## 3. Recitation check

> **Part 14 — pronunciation check.** The check answers only "can this user say this Arabic correctly?"
> Every result now has letters: `lib/pronunciation/` (see `docs/PRONUNCIATION_ENGINE.md`). Flow: pick text →
> listen (1× / 0.75× / 0.5×) → say it **once** → result. Tap an amber word → letter-by-letter view (green
> correct, amber needs work, grey not clear) with the issue in simple words, "Play the word slowly" and (if the
> device has an Arabic voice) "Hear the letter". Serious issues ("Fix this first") are listed before minor ones
> ("Polish later"). "Practice weak parts" runs a mini drill of only the weak words, serious first. Confidence
> under 0.4 → "We couldn't hear clearly", nothing saved. Counts ("say it 3 times") are guidance only, never checked.

### Pipeline

```
[Browser] record → convert to 16 kHz mono WAV → POST /api/recite (audio + itemId)
[Next.js API] validate (≤ 60 s, ≤ 2 MB) → forward audio to speech endpoint
[Speech endpoint] Whisper Quran model → Arabic text + confidence
[Next.js API] normalize both texts → align words → score → save attempt (no audio)
[Browser] color each word, show score + feedback
```

### Text normalization (applied to expected text and transcript)

- Remove tatweel (ـ) and Quranic annotation marks.
- Unify alif forms (أ إ آ ٱ → ا) for matching; keep the original for display.
- Unify ى/ي and ة/ه at word end for matching.
- Tashkeel: ignored in MVP matching (word level). Used in P2 letter-level checks.

### Word alignment

Levenshtein alignment over word arrays. Each expected word gets one status:

| Status | Meaning | Display |
|---|---|---|
| `correct` | Matched | Normal text, soft green underline |
| `wrong` | Substituted by a different word | Amber underline + tap to hear correct |
| `missed` | Not recited | Amber dotted outline |
| `extra` | Recited but not expected | Small amber marker between words |

**Score** = correct words ÷ expected words × 100 (extra words subtract 2 points each, minimum 0).

### Confidence handling

| Model confidence | Result shown |
|---|---|
| High | Normal word-colored result |
| Medium | Result + "Double-check with the qari audio" |
| Low, or audio too quiet/noisy | "We couldn't hear clearly. Try again in a quieter place." No score saved |

### Limits and errors

| Case | Behavior |
|---|---|
| Mic permission denied | Explain how to enable; offer listen-only mode |
| Recording > 60 s | Stop automatically, check what was recorded |
| No speech detected | "We didn't hear anything. Tap the mic and start reciting." |
| Speech endpoint down/slow (> 10 s) | "Checking is busy right now. Your practice still counts." Mark step as practiced, unscored |
| Offline | Recording disabled, listen + learn still work |

### Privacy

- Audio lives only in memory during the request. Never written to disk, database, or logs.
- `recitation_attempts` stores word statuses and score only.

---

## 4. Knowledge Score

> **Part 15:** the Profile now shows the **Learning Score** = must-know (fard 'ayn) items mastered ÷ measurable items (`lib/mastery.ts`). Knowledge = quiz ≥ 80%; Arabic = pronunciation check ≥ 90 with no serious issue; an item is mastered when every part it has is mastered. Two bars per section (Knowledge, Pronunciation). "Fix next": serious pronunciation issues first. Private; measures learning, never faith. The weighted score below still exists for placement and the monthly re-check.

### Per item

| Item type | Formula |
|---|---|
| Recitable (dua, surah) | 50% best recent quiz + 50% best recent recite score |
| Knowledge-only (ruling) | 100% quiz |

### Decay

Recite scores lose 1 point per week without practice after 30 days, down to a floor of 50% of the best score. Practicing restores it immediately. This keeps the score honest without punishing heavily.

### Aggregation

```
section score  = average of its item scores (unstarted items = 0)
level score    = average of its section scores
overall score  = 0.70 × Level 1 + 0.20 × Level 2 + 0.10 × Level 3
```

Level 2 sections that are not unlocked for the user are excluded, and their weight moves to Level 1.

### Display rules

- Named **"Learning Score"** in the UI, never "Muslim score".
- Profile footnote: "This measures what you have learned in the app. Only Allah knows the state of anyone's faith."

---

## 5. Spaced repetition (reviews)

Simple interval system per item:

| Last result | Next review |
|---|---|
| First time learned | 1 day |
| Passed | Previous interval × 2.5 (max 90 days) |
| Weak (score < threshold) | 1 day |
| Failed 3 times | Same day, later |

"Due today" list = items where `next_review <= today`, weakest first, max 10 per day.

---

## 6. Streaks and daily goal

- Daily goal met = minutes of active learning ≥ goal.

### What counts as learning time

- Counts only on learning screens: lesson player, recitation check, review quiz, re-check, walkthroughs, Quran reader (reading or audio), adhkar and tasbih counters, together mode, hifz.
- Never counts on Today, Essentials, the review list, Profile or Settings.
- Pauses when the tab is hidden or the phone locks, after 60 s with no tap, key, scroll, audio or recording, and when the learner leaves the screen.
- `lib/learning-timer.ts` (`useLearningTimer`): 1 s tick while active; every 15 s of active time it adds to today's local minutes and sends a beat to `POST /api/progress/activity` (signed-in only).
- Server (`add_learning_seconds`, migration 0017): at most 20 s per beat; a beat less than 12 s after the last one is rejected. Old minutes stay as they were.
- A small green "timer on" pill in the focus header shows when it is counting.

### Today page order

1. Ayah/hadith card: 2 lines Arabic, 2 lines meaning, "Read more" opens the full sheet.
2. Today's lesson (hero): title, steps, progress bar, daily-goal ring "Today: 3 / 5 min", big Start/Continue button, "See full path" link. Goal met → calm "Today's goal done, Alhamdulillah" and an "Another lesson" button. The separate goal card is gone (it stays in the desktop right panel on other pages).
3. One prayer card: next prayer, time, countdown; the 5 salah circles below; a privacy line.
4. Quick tools: one scrolling row of chips (Qibla, Tasbih, Feelings, Adhkar, Before sleep).
5. Sunnah of the day.
6. Iman check-in.
- Tapping the Today button while on Today scrolls smoothly to the top (instant with reduced motion).

### Time and source display

- `formatClock` (`lib/clock.ts`): bn "সকাল ১১:৫১" (দুপুর, বিকাল, সন্ধ্যা, রাত); ur, ar localized periods and digits; never AM/PM outside English. Settings → Prayer → "24-hour clock".
- `formatSource(source, locale)` (`lib/source-i18n.ts`, `<Source/>`): every hadith/ayah source line is shown in the UI language and digits.
- Streak increases once per calendar day in the user's timezone.
- 2 automatic streak freezes per month.
- Missing a day with no freeze resets to 0 with a kind message: "Welcome back. Start a new streak today."

---

## 7. Quran reader

### Flow

```
Surah list → Surah page (ayahs) → tap ayah → action sheet:
  Play · Word by word · Translation · Tafsir · Practice this ayah · Bookmark · Share
```

### Rules

- Text, translation, tafsir, and audio URLs are fetched on the server and cached (30 days).
- The reader remembers the last read position (Supabase for signed-in users, local storage for guests).
- Audio: continuous play with the current ayah highlighted and auto-scrolled into view.
- Word tap: shows meaning + plays word audio.
- Every translation and tafsir shows its source name (translator/institution). Only sources enabled by the scholar board appear.
- Browser auto-translate is disabled on Quran text and translations (`translate="no"`).

---

## 8. Reminders

- Web Push through the PWA service worker (requires the app to be installed on iOS).
- Times based on the chosen slot. Prayer times are approximated from the user's timezone and optional city.
- Maximum 1 reminder per day. If the goal is already met, no reminder.

---

## 9. Offline (PWA)

| Available offline | Not available offline |
|---|---|
| All content files (duas, lessons, rulings) | Recitation check |
| Dua audio already played | Sign-in |
| Surahs opened before | Uncached surahs |
| Progress (queued, synced later) | Invite counts |
| Prayer times, qibla, tasbih, adhkar, zakat, hijri calendar, journal | Verified answers not read before |
| Anything saved from **Settings → Offline downloads** | |

### Precache

On install the service worker fetches the shell and every tool that is arithmetic or stored text:
`/today`, `/essentials`, `/quran`, `/review`, `/profile`, `/welcome`, `/offline`, `/downloads`,
`/prayer`, `/prayer-times`, `/qibla`, `/tasbih`, `/adhkar`, `/duas`, `/sleep`, `/calendar`,
`/journal`, `/zakat`, `/hifz`, `/tajwid`, `/seasons`. So they work on the first flight, not only
after someone happened to open them.

### Downloads (`/downloads`)

Chosen in advance, for a bus with no signal or a plan with no data left. Three kinds of pack:

| Pack | What it saves |
|---|---|
| A unit | Its lesson pages and the recitation for every ayah in them |
| A juz (1–30) | The surah pages it spans and the recitation for its ayahs |
| Daily duas | The dua pages and their recordings |

- Every pack shows its size **before** it is tapped (`lib/downloads.ts`, ~120 kB per recited ayah).
  A download nobody can size is one nobody trusts; on these phones a surprise 100 MB is somebody's
  month of data.
- The page asks `navigator.storage.persist()` so the browser does not evict a pack when the phone
  runs low — which is exactly when it is needed — and shows `navigator.storage.estimate()` usage.
- The app talks to the service worker by `postMessage`: `download-pack` → `pack-progress` /
  `pack-done`, `delete-pack` → `pack-deleted`. Saved packs are listed in `offlinePacks` in local
  state, so a delete gives the space back.

---

## 10. Guest to account

1. Guest progress is stored in local storage.
2. On sign-up, local progress is uploaded and merged (keep the higher score per item).
3. Local copy is cleared after a successful merge.

---

## 11. Report a mistake

1. The flag icon on any item opens a short form: type (text, audio, translation, ruling, other) + note.
2. Saved to `content_reports` with the item ID and app version.
3. Admin dashboard shows open reports. The fix happens in the content file, reviewed by the scholar, and is deployed.

---

## 12. Admin

- Route `/admin`, access limited to users with `role = 'admin'` in `profiles`.
- Pages: reports inbox, user counts, daily active users, recitation attempts per day, average score per item (shows which items are hardest).
- Content is not edited here; it is edited in `/content` files.

---

## 13. Seasonal modes

### Detection

- Hijri date from `Intl.DateTimeFormat` with the `islamic-umalqura` calendar.
- Moon sighting differs by country, so settings include **Hijri date adjustment: −2 to +2 days**. Default 0. Onboarding does not ask; the first seasonal banner does ("Did Ramadan start today where you live?").
- Seasons are defined in `content/seasons.ts`:

```ts
export const seasons = [
  { id: "ramadan",     start: { month: 9,  day: 1 },  end: { month: 9,  day: 30 } /* month may end on day 29: season ends when Shawwal begins */, preDays: 14 },
  { id: "eid-fitr",    start: { month: 10, day: 1 },  end: { month: 10, day: 3 } },
  { id: "dhul-hijjah", start: { month: 12, day: 1 },  end: { month: 12, day: 13 }, preDays: 7 },
  { id: "eid-adha",    start: { month: 12, day: 10 }, end: { month: 12, day: 13 } },
  { id: "ashura",      start: { month: 1,  day: 7 },  end: { month: 1,  day: 10 } },
  { id: "jumuah",      weekday: 5 },  // every Friday
];
```

- `preDays` = a "Get ready" banner shows that many days before (e.g. 14 days before Ramadan: learn fasting rulings now).

### What changes when a season is active

1. A seasonal banner card at the top of Learn and Essentials.
2. A temporary seasonal unit appears at the top of the learning path (daily lessons for that season).
3. Seasonal items count toward the Learning Score only while relevant rulings are Level 1 (fasting is already Level 1).
4. Theme accent shifts slightly (seasonal color + crescent icon in the top bar).
5. Seasonal badge when the seasonal path is completed.
6. After the season: the unit moves into Essentials as a normal section, progress kept.

### Ramadan specifics

- **Suhoor ends / iftar times:** calculated with the `adhan` library from the user's city and chosen calculation method (settings). Always shows: "Follow your local masjid's timetable if it differs."
- **Daily Ramadan path:** one short lesson per day of Ramadan (29 or 30 days, following the actual month; day 30 is optional content) — one ruling, one dua, one ayah meaning, one review.
- **Quran khatm planner:** choose finish date → daily pages/juz target → progress in the Quran reader.
- **Qada fasts tracker:** count missed fasts, mark made-up days.
- **Last 10 nights:** reminder card each night + Laylatul Qadr dua practice.
- **Zakat al-Fitr:** ruling + "pay before Eid prayer" reminder (amount rules shown per madhab; no fixed currency amount unless the scholar approves a local figure).

### Jumu'ah (weekly)

- Friday banner: Surah Al-Kahf (opens reader), durood reminder, Jumu'ah rulings.
- Streak-friendly: reading Al-Kahf on Friday counts toward the daily goal.

---

## 14. Faith features

### Home hadith/ayah card

- Source: `content/cards/*.ts`, each card `{ id, type: "hadith"|"ayah", arabic, meaning, explanation, action, source, grading, topics[], contexts[], review }`.
- Only `grading` Sahih or Hasan. Only reviewed cards.
- Selection on each app open:
  1. Filter cards matching current context: `friday`, `ramadan`, `night` (after Isha), `fajr`, `default`.
  2. Remove cards already seen in the current cycle (seen list in local storage + profile).
  3. Pick randomly, weighted toward the user's weak topics (e.g. salah if salah check-in is low).
  4. When all seen → reset cycle.
- Tap → sheet: full Arabic, meaning in user's language, short explanation, "Act on it today" line, source.
- Share → generates a styled image (card design, source included, app name small). No user data.
- Start with 100–200 cards. Topics: salah, dua, patience, parents, character, akhirah, Quran, mercy, tawbah, time.

### New Muslim path

- Chosen when onboarding answer = "New to Islam".
- Order: Shahada (meaning + recitation check) → kalimas → wudu → salah step by step → pillars of Iman → pillars of Islam → halal/haram basics.
- Kalimas: presentation decided by the scholar. The app must make clear the Shahada is what enters a person into Islam.
- Tone: extra gentle, extra explanation, no assumed knowledge.

### Salah attachment path

| Stage | Content |
|---|---|
| Why | What salah is, its place in Islam, what is promised, what leaving it means |
| Understand | Meaning of every word said in salah, with quizzes |
| Small steps | Goal 1: never miss fard. Goal 2: add sunnah. Tracked privately |
| Heart | Khushu tips, Prophet ﷺ and Sahaba with salah, dua for steadfastness (Quran 14:40) |
| Falling back up | Missed salah → gentle tawbah message + qada guidance, never shame |

### Private salah check-in

- Home shows 5 small prayer circles for today. Tap = prayed.
- Private only. Not in the Learning Score, not shared, not in badges.
- Missed → next open shows a gentle card, not a warning.

### Digital distraction section

- Lessons: guarding the eyes, value of time (Sahih Bukhari: two blessings many people waste, health and free time), phone habits and the heart.
- Guides: step-by-step for iPhone Focus and Android Digital Wellbeing / Bedtime mode, set around salah times.
- Honest limit shown: the app cannot block other apps; it guides the user to phone tools.
- "Instead of scrolling" button on home → 1-minute dhikr card (SubhanAllah, Alhamdulillah, Allahu Akbar counter).
- Weekly private reflection (Friday): "This week: time on phone vs time with Quran?" user answers, only they see it.

### Ramadan highlights

| Group | Color | Examples |
|---|---|---|
| Must (fard) | Green | Fast every day of Ramadan, intention, keep 5 salah, avoid what breaks the fast |
| Do more | Blue (sky) | Quran, taraweeh, sadaqah, dua, Laylatul Qadr, i'tikaf, feeding others at iftar |
| Must not | Amber | Eating, drinking, intimacy while fasting; lying, backbiting, arguing (take away the reward) |

Shown as the top of the Ramadan mode; each item tap → ruling + source; madhab differences noted.

## 15. Motivation

- **Mood check:** once a day on Learn. Saved to `imanCheckins` (synced to `iman_checkins`). The mood picks content from `content/motivation/by-mood.ts`.
- **Minimum mode:** `profile.minimumMode = { since }`. Offered after 3 days with no check-in (asked once). While on, `settleStreak(prev, today, graceSince)` does not reset the streak, and the daily goal is paused.
- **Adhan nudges:** `/api/cron/adhan` (Bearer `CRON_SECRET`) runs every 5 minutes. For each subscription: compute today's times from its saved lat/lng and settings; if a chosen prayer started in the last 15 minutes, not in quiet hours and not already sent, send the next nudge (`nudge_cursor` rotates). The same run sends fasting reminders the evening before.
- **Pre-salah card:** shown for `WHY_WINDOW_MIN` (20) minutes after the current prayer began, until "I'm going to pray" is tapped.
- **Welcome back:** if the previous open was 3+ days ago, one card, dismissible.

## 16. Daily tools

- **Prayer times:** `lib/prayer.ts` with `PrayerOptions { asr, highLat, offsets }`. Location from geolocation (reason shown first) or city search. The first location becomes home.
- **Travel:** a new location ≥ 50 km (`TRAVEL_ASK_KM`) from home sets `travelAsk`; the prayer page asks. "Yes" shows qasr/jam' guidance with madhab notes; the app never changes prayer counts itself.
- **Qibla:** `bearingTo(loc, KAABA)`. `deviceorientationabsolute` (Android) or `webkitCompassHeading` (iOS, after `requestPermission`). Aligned within 3°, vibrates once on entering.
- **Tasbih:** local state only; preset 33/33/34 or custom target.
- **Checklists:** `checklists[<id>][<day>]` in the store (sleep, jumu'ah).
- **Sadaqah / gratitude:** local, synced to `sadaqah_entries` / `gratitude_notes` when signed in.
- **Zakat:** `lib/zakat.ts`; nisab basis from `content/zakat-config.ts` per madhab (draft); gold valued as grams × karat/24 × 24k price. Nothing is saved.

## 17. Invite to learn

- `/i/<code>` (route handler): a valid 8-character code is kept in the `ilm-ref` cookie for 30 days, then the visitor lands on the home page.
- After sign-in, account sync calls `claim_referral(code)` once and clears the cookie. It counts only accounts created in the last 7 days, once per account, never your own code.
- The first finished lesson (a `lesson_progress` insert) sets `first_lesson_at` via a trigger; a lesson done before claiming is picked up by `claim_referral`.
- Profile → **Invite to learn**: `my_referral_code()` (created on first use) and `my_referral_counts()` → "N people started learning through you" and how many finished a first lesson. Only counts, never who. No points, rankings, rewards or public counts.
- Share: the native share sheet (when available), copy link, WhatsApp, Facebook and Telegram links.

## 18. Removed features (2026-09-26)

- Salah Verified, family & classes (child profiles, child mode, parent dashboard, teacher panel, maktab classes) and prayer buddies were removed from the app, website and admin. Their API routes were deleted, so they return 404.
- Devices that were left in child mode return to the owner's state (the old `ilm-active-child` key is cleared on load).

## 19. Verified answers (Part 18)

- `/answers`: published answers (scholar, madhab, sources, reviewed date), search and categories; "Report an issue" uses the existing report flow (`item_id = answer:<id>`).
- `/answers/ask`: shows similar published answers while typing; private; max 3 a day (API + database trigger). Published answers never link to the asker.
- `/scholar`: queue → write per-locale question/answer + sources (publishing requires a source) → publish; merge duplicates; decline. `/about/scholars` lists answering scholars from the database.
- No AI-generated religious content exists anywhere in the app (checked: no AI SDKs or chat).
