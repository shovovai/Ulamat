# Roadmap

Build the riskiest part first (speech checking), then the app around it.

---

## Phase 0 — Validate (weeks 1–2)

**Goal:** prove the speech model works well enough on Quran and duas.

- [ ] Create accounts: GitHub, Vercel, Supabase, Hugging Face.
- [ ] Apply for Quran Foundation API access.
- [ ] Contact 2–3 scholars; agree on one reviewer for Level 1 content.
- [ ] Run `tarteel-ai/whisper-base-ar-quran` in a notebook on:
  - Al-Fatiha, Tashahhud, Durood Ibrahim, ruku tasbih, janazah dua
  - 10 voices: men, women, children, different accents
  - Correct recitations and recitations with deliberate mistakes
- [ ] Record the results: correct words detected, mistakes caught, false alarms.

**Decision at end of week 2:**

| Result | Plan |
|---|---|
| Works well on Quran and duas | Continue as planned |
| Works on Quran, weak on duas | Launch with Quran checking + listen-and-repeat for duas; start collecting dua recordings |
| Weak on both | Try Whisper large-v3; if still weak, launch learning features first, checking in phase 2 |

---

## Phase 1 — MVP build (weeks 3–10)

### Weeks 3–4: foundation

- [ ] Next.js project, Tailwind with DESIGN.md tokens, fonts, dark mode
- [ ] App shell: tab bar (mobile), rail (tablet), sidebar + right panel (desktop)
- [ ] Supabase: schema + RLS from DATA_AND_API.md, auth (magic link + Google)
- [ ] PWA: manifest, icons (8-point star), service worker
- [ ] Content system: types, `content/index.ts`, CI checks
- [ ] Deploy to Vercel from day one (preview URL for every branch)

### Weeks 5–6: learning

- [ ] Onboarding flow (all 10 steps) + guest mode
- [ ] Learning path screen with star nodes
- [ ] Lesson player: learn, listen, quiz, match, order steps
- [ ] First 20 content items written and reviewed (salah recitations first)
- [ ] Essentials dashboard + section detail pages

### Weeks 7–8: recitation + Quran

- [ ] Recording component (16 kHz WAV, waveform, timer)
- [ ] `/api/recite`: speech endpoint, normalization, alignment, scoring
- [ ] Result screen with word colors, tap-to-hear, not-sure state
- [ ] Quran reader: surah list, ayahs, translation, word-by-word, audio player, tafsir sheet, bookmarks, last read
- [ ] "Practice this ayah"

### Weeks 9–10: motivation + polish

- [ ] Knowledge Score + profile page
- [ ] Review queue (spaced repetition)
- [ ] Streaks, daily goal, badges
- [ ] Reminders (web push + Vercel Cron)
- [ ] Salah and janazah walkthroughs
- [ ] Settings, report a mistake, delete/export account
- [ ] All Level 1 content reviewed (see CONTENT.md)
- [ ] Empty, loading, offline, and error states on every screen

---

## Phase 2 — Beta (weeks 11–12)

- [ ] 20–50 testers: family, friends, a local masjid, madrasa students, at least 5 reverts
- [ ] Watch 5 people use it in person without helping them; note every confusion
- [ ] Track in PostHog: onboarding drop-off, first lesson completion, day-7 return rate
- [ ] Tune recitation confidence thresholds with real attempts
- [ ] Fix, then fix again

### Beta success targets

| Metric | Target |
|---|---|
| Onboarding completion | ≥ 70% |
| First lesson completed | ≥ 60% of sign-ups |
| Users returning on day 7 | ≥ 25% |
| "Not sure" results | ≤ 15% of attempts |
| Wrongly marked correct recitations (reported) | Near zero |

---

## Launch checklist

- [ ] Custom domain connected on Vercel
- [ ] Privacy policy + terms pages
- [ ] About our scholars page with names and sources
- [ ] Landing page with live Al-Fatiha demo
- [ ] Sentry alerts on; PostHog dashboards set up
- [ ] Speech endpoint scaling tested (20 users reciting at once)
- [ ] Quran Foundation production credentials in place
- [ ] Lighthouse: performance ≥ 90 on mobile, accessibility ≥ 95
- [ ] Tested on: low-end Android (Chrome), iPhone (Safari, installed as PWA), desktop Chrome/Firefox/Safari

---

## Phase 3 — After launch (months 4–6)

- Dua recordings dataset (20–30 qaris incl. women and children, with consent) → fine-tune the model
- Letter-level tajweed checks: madd length, confused letters, ghunnah
- Level 2 life-situation sections and Level 3 growth sections
- Teacher review (paid), with female teachers
- Kids mode + family accounts
- More UI languages
- Play Store release (Trusted Web Activity or Expo wrapper)

---

## Costs to expect (check current pricing before launch)

| Service | Start | When it grows |
|---|---|---|
| Vercel | Hobby (free, non-commercial) | Pro once you charge money |
| Supabase | Free tier | Pro when you pass free limits |
| Hugging Face endpoint | Pay per GPU hour, scale-to-zero | Biggest variable cost; watch it |
| Domain | ~yearly fee | |
| Quran Foundation API | Free access on approval | Check terms for commercial use |
