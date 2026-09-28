# Features

Priority key: **MVP** = launch version · **P2** = months 4–6 · **P3** = later

---

## 1. Account and onboarding

| Feature | Priority | Notes |
|---|---|---|
| Guest mode | MVP | Use the app before signing up; progress kept on device until sign-up |
| Sign up / log in | MVP | Email magic link + Google. Phone OTP in P2 |
| Onboarding flow | MVP | Language, user type, goal, Arabic level, placement check, madhab, script, gender, daily time |
| Placement check | MVP | Recite Al-Fatiha + 5 quiz questions → starting Knowledge Score |
| Profile settings | MVP | Change any onboarding answer later |
| Delete account + all data | MVP | One tap, required for trust and privacy law |

## 2. Essentials (the must-know dashboard)

| Feature | Priority | Notes |
|---|---|---|
| Level 1 sections | MVP | Aqeedah, Taharah, Salah, Quran minimum, Sawm, Janazah, Daily duas, Halal & haram basics |
| Section progress rings | MVP | Each section shows its % |
| "Fix this next" card | MVP | Always suggests the weakest Level 1 item |
| Salah walkthrough | MVP | Every position of the prayer in order, with the recitation for each |
| Janazah walkthrough | MVP | The 4 takbirs and what is recited after each |
| Madhab-specific versions | MVP | Hanafi, Shafi'i, Maliki, Hanbali; "not sure" shows common ground |
| Level 2 life-situation unlocks | P2 | Zakat, Hajj/Umrah, marriage, business, parenting, travel |
| Level 3 growth sections | P2 | 40 Hadith Nawawi, Seerah, 99 Names, prophets, adab |

## 3. Learning path (Duolingo-style)

| Feature | Priority | Notes |
|---|---|---|
| Path of units and lessons | MVP | Units follow Essentials order |
| Lesson types | MVP | Learn card, listen & repeat, recite check, quiz, match meaning, order the words |
| Arabic letters course (Qaida) | MVP | Letters, harakat, joining, makharij basics |
| Tajweed lessons | P2 | One rule per lesson |
| Hearts / lives | ✗ Not planned | Punishing mistakes in worship learning is the wrong feeling |

## 4. Recitation and pronunciation checking

| Feature | Priority | Notes |
|---|---|---|
| Word-level check | MVP | Correct / missed / extra / wrong per word |
| "Not sure" result | MVP | Low confidence → ask to retry, never guess |
| Listen to reference first | MVP | Qari audio before each attempt |
| Slow playback | MVP | 0.75× and 0.5× |
| Word-tap pronunciation | MVP | Tap any word to hear it alone |
| Attempt history per item | MVP | Scores only, no audio |
| Dua-specific model | P2 | Fine-tuned on collected dua recordings |
| Letter-level tajweed check | P2 | Madd length, confused letters, ghunnah |
| Real-time (while reciting) feedback | P3 | Streaming recognition |

## 5. Quran reader

| Feature | Priority | Notes |
|---|---|---|
| Surah list + search | MVP | Name, number, Makki/Madani, ayah count |
| Uthmani / IndoPak script | MVP | Per-user setting |
| Translation (user's language) | MVP | Source shown under every translation |
| Word-by-word meaning + audio | MVP | Tap a word |
| Ayah audio with reciter choice | MVP | Current ayah highlighted |
| Tafsir panel | MVP | Short tafsir (e.g. al-Mukhtasar) first; longer tafsir optional |
| "Practice this ayah" | MVP | Opens recitation check |
| Bookmarks + last read | MVP | Stored in Supabase |
| Tajweed color-coded text | P2 | |
| Repeat ayah N times | P2 | For memorization |

## 6. Hifz and revision

| Feature | Priority | Notes |
|---|---|---|
| Spaced repetition queue | MVP | "Due for review today" list |
| Hide-text test | MVP | Recite from memory, words revealed as recited |
| Weak items list | MVP | Lowest recent scores |
| Hifz planner | P2 | Goal: surah X by date Y |

## 7. Profile, score, motivation

| Feature | Priority | Notes |
|---|---|---|
| Knowledge Score | MVP | Weighted: Level 1 = 70%, Level 2 = 20%, Level 3 = 10% |
| Section breakdown | MVP | Rings per section |
| Streak + daily goal | MVP | Streak freeze 2×/month |
| Badges | MVP | Milestones only (e.g. "Salah complete"), no competition |
| Reminders | MVP | Linked to prayer times: "after Fajr", "after Isha" |

## 8. People

| Feature | Priority | Notes |
|---|---|---|
| Kids mode | P2 | Simpler UI, parent dashboard, no voice data retained |

## 9. Trust, settings, admin

| Feature | Priority | Notes |
|---|---|---|
| Source + reviewer on every item | MVP | "Reviewed by …" line |
| Report a mistake | MVP | Sends item ID + note to admin |
| About our scholars page | MVP | Names, qualifications |
| Dark mode | MVP | Follows system, override in settings |
| Text size control | MVP | Arabic and translation sized separately |
| Offline (PWA) | MVP | Content files + visited surahs cached |
| Admin dashboard | MVP | Reports inbox, basic stats (read from Supabase) |
| Multiple UI languages | MVP (2 languages) → P2 (more) | |

---

## 10. Seasonal modes

| Feature | Priority | Notes |
|---|---|---|
| Season engine | MVP+ (ready before Ramadan) | Hijri calendar detection + user date offset (−2 to +2 days) for local moon sighting |
| Ramadan mode | MVP+ | Fasting rulings, suhoor/iftar duas, taraweeh guide, last 10 nights, Laylatul Qadr dua, Zakat al-Fitr, daily Ramadan path, Quran khatm planner, qada fasts tracker |
| Suhoor / iftar times | MVP+ | Calculated from city + calculation method; "follow your local masjid" note |
| Dhul Hijjah mode | P2 | First 10 days, takbirat, fasting Arafah, Udhiyah (qurbani) rules |
| Hajj / Umrah guide | P2 | Step-by-step rites with duas, pronunciation, meaning |
| Eid mode (both Eids) | MVP+ | Eid prayer steps, takbir, sunnahs of Eid |
| Muharram / Ashura | P2 | Fasting 9th and 10th |
| Jumu'ah mode (weekly) | MVP+ | Every Friday: Surah Al-Kahf, durood, Jumu'ah rulings |
| Seasonal badges | MVP+ | e.g. "Ramadan 1448 complete" |
| Seasonal theme accent | MVP+ | Small accent change (night blue + crescent) while a season is active |

---

## 11. Faith features

| Feature | Priority | Notes |
|---|---|---|
| Home hadith/ayah card | MVP | New card every app open, no repeats until all shown, sahih/hasan only, source + grading, context-aware (Friday, Ramadan, night), tap for explanation + "act on it today", share as image |
| New Muslim path | MVP | Shahada first, then kalimas (presented as the scholar decides), wudu, salah, basics of Iman and Islam |
| Salah attachment path | MVP | Why salah → meaning of every word → small steps → heart (Seerah, Sahaba) → falling back up (tawbah, no shame) |
| Private salah check-in | MVP | 5 daily taps, never public, never in the Learning Score |
| Digital distraction section | MVP | Guarding the eyes, value of time, phone Focus/Digital Wellbeing guides, "scroll urge → dhikr card", weekly private reflection |
| Ramadan: Must / Do more / Must not | MVP+ | Three color-highlighted groups, each item with ruling + source |

## 12. Motivation

| Feature | Priority | Notes |
|---|---|---|
| Iman mood check | MVP | 3 moods, answer picks a card or dua; private (`iman_checkins`) |
| Minimum mode | MVP | "I'm struggling": only 5 circles + one dua; streak kept |
| Adhan-time nudges | MVP | Push per prayer, rotating text, quiet hours |
| Pre-salah why card | MVP | 20 min after adhan, "I'm going to pray" |
| Consistency graph | MVP | Private 8-week grid on Profile |
| Gentle return | MVP | Welcome back after 3+ days, no guilt |
| "New" label | MVP | `publishedAt` on cards, 14 days |

## 13. Daily tools

| Feature | Priority | Notes |
|---|---|---|
| Prayer times | MVP | Method, Asr, high-lat, offsets, city search/geolocation, monthly table |
| Qibla | MVP | Compass with ±3° haptic, fallback text |
| Duas for feelings | MVP | 12 feelings, comfort ayah, search |
| Adhkar + tasbih | MVP | Listen, auto-advance; counter page |
| Sunnah of the day, Jumu'ah checklist, sleep checklist | MVP | Home cards and `/sleep` |
| Istikhara | MVP | Walkthrough |
| Fasting reminders | MVP+ | Mon/Thu, white days |
| Hijri calendar | MVP | `/calendar` |
| Travel ask | MVP+ | Asks, never decides |
| Sadaqah + gratitude journal | MVP+ | Private `/journal` |
| Zakat calculator | MVP | Madhab-config nisab, explanation, draft |
| Daily-life guidance | MVP | Work/university prayer, visiting the sick, ruqyah |
| Widgets, Fajr alarm, masjid finder, halal scanner | Native | See `docs/NATIVE_PHASE.md` |

## 14. Core: pronunciation, mastery, verified salah

| Feature | Priority | Notes |
|---|---|---|
| Pronunciation check | MVP | Letter-level result, serious vs minor, letters view, practice weak parts. Phase 2 letter engine behind a flag |
| Learning Score (mastery) | MVP | Must-know items mastered; Knowledge + Pronunciation bars; Fix next |

## 15. Website and admin

| Feature | Priority | Notes |
|---|---|---|
| Public website in 4 languages | MVP | `/`, `/features/*`, how it works, pronunciation, scholars, about, gallery, FAQ, contact, volunteer, blog, changelog, press |
| Legal pages + versioning | MVP | 9 documents, DRAFT until lawyer review, acceptances |
| Cookie consent | MVP | Analytics opt-in |
| Admin panel | MVP | TOTP, roles, audit log, CMS, reviews, inbox, flags, broadcasts, invite totals per day |
| Invite to learn | MVP | Personal link `/i/<code>`; share sheet, copy, WhatsApp/Facebook/Telegram. Private Profile card: "N people started learning through you" + first lessons. No points, rankings, rewards or public counts |

**Removed (2026-09-26):** Salah Verified (levels, teacher recording, certificates, `/verify`), family and classes (child profiles, child mode, parent dashboard, `/teach`, maktab classes, `/for-families`, `/for-maktabs`) and prayer buddies. Their database tables are kept but deprecated (see DATA_AND_API.md).
