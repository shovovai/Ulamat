# Islamic Learning App — Product Specification

A mobile-first web app where Muslims learn what they must know (fard 'ayn) first, then grow further. Users learn duas, surahs, and rulings, and an AI checks whether their recitation and pronunciation are correct.

> Working title: **[App Name]** — replace everywhere once chosen.

## Core promise

"Learn what every Muslim must know — and know that you are saying it correctly."

## Product principles

1. **App first.** Every screen is designed for a phone held in one hand. The desktop web view is an adaptation of the mobile app, never the other way around.
2. **Must-know first.** Fard 'ayn content (belief, purity, salah, fasting, janazah) always comes before optional knowledge.
3. **Authentic or hidden.** Nothing reaches users without a named scholar's review and a visible source.
4. **Kind correction.** Mistakes are shown gently. Nobody should feel like a "bad Muslim" for not knowing something.
5. **Private by default.** Voice recordings are never stored. Progress is private unless the user shares it.
6. **One project.** A single Next.js app. Content lives in code files. Supabase holds only user data.

## Documents in this folder

| File | What it covers |
|---|---|
| [FEATURES.md](./FEATURES.md) | Every feature, grouped and prioritized (MVP, phase 2, phase 3) |
| [FUNCTIONALITY.md](./FUNCTIONALITY.md) | How each feature works step by step: logic, rules, edge cases |
| [DESIGN.md](./DESIGN.md) | Design system: colors, typography, Arabic fonts, components, motion, tone |
| [LAYOUT.md](./LAYOUT.md) | App shell, navigation, grid, spacing, breakpoints |
| [MOBILE_VIEW.md](./MOBILE_VIEW.md) | Screen-by-screen mobile wireframes (the primary experience) |
| [WEB_VIEW.md](./WEB_VIEW.md) | Tablet and desktop adaptations of every screen |
| [CONTENT.md](./CONTENT.md) | Content file format, review workflow, the full Level 1 content list |
| [DATA_AND_API.md](./DATA_AND_API.md) | Supabase schema, API routes, external Quran APIs, speech service |
| [ROADMAP.md](./ROADMAP.md) | Build order, timeline, launch checklist |

## Tech stack (summary)

| Part | Choice |
|---|---|
| Framework | Next.js (App Router) + TypeScript |
| Styling | Tailwind CSS with the tokens in DESIGN.md |
| Installable app | PWA (manifest + service worker), later wrapped for Play Store |
| Database + auth | Supabase (Postgres + Auth). No Supabase Storage. |
| Learning content | TypeScript files in `/content` |
| Quran text, translation, tafsir, audio | Quran Foundation API + QuranEnc, cached by Next.js |
| Speech checking | Hugging Face Inference Endpoint running `tarteel-ai/whisper-base-ar-quran` (compute only, nothing stored) |
| Hosting | Vercel |
| Monitoring | Sentry (errors), PostHog (product analytics) |

## Glossary

| Term | Meaning |
|---|---|
| Fard 'ayn | Knowledge every Muslim must personally learn |
| Level 1 / 2 / 3 | Must know / unlocks by life situation / growth |
| Item | One learnable unit: a dua, surah, ruling, or hadith |
| Recitable item | An item the AI can check by voice |
| Knowledge Score | Weighted progress score shown on the profile |
| Review | A spaced-repetition prompt to practice something again |

## Website and admin (parts 19–22)

- Public website: `app/(site)` (see WEB_VIEW.md). Copy in `messages/site/*.json`, legal in `content/site/legal`.
- Admin panel: `/admin` in this app (`app/(panel)/admin`), 404 for anyone without a `staff_roles` row.
