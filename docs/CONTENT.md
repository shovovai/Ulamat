# Content

All learning content lives in TypeScript files inside `/content`. No CMS, no separate storage. The Quran itself is fetched from APIs (see DATA_AND_API.md) and is not stored in content files.

---

## 1. Folder structure

```
content/
├── index.ts                 → exports all items, filters out unreviewed ones
├── sections.ts              → section list, levels, weights, order
├── units.ts                 → learning path: units → lessons → steps
├── aqeedah/
├── taharah/
├── salah/
│   ├── sana.ts
│   ├── tashahhud.ts
│   ├── durood-ibrahim.ts
│   ├── rulings/
│   │   ├── prayer-times.ts
│   │   └── what-breaks-salah.ts
│   └── walkthrough.ts       → ordered positions of the prayer
├── quran-minimum/           → references to surahs (surah + ayah range), not the text
├── sawm/
├── janazah/
├── daily-duas/
├── halal-haram/
└── quizzes/                 → quiz questions, grouped by section
public/audio/
├── salah/ tashahhud.mp3 ...
└── daily-duas/ ...
```

---

## 2. Types

```ts
// types/content.ts
export type Madhab = "hanafi" | "shafii" | "maliki" | "hanbali";
export type Level = 1 | 2 | 3;
export type LangCode = "en" | "bn" | "ur" | "ar" | "id";

export interface Review {
  reviewedBy: string;      // scholar name, must match /about/scholars
  reviewedAt: string;      // ISO date
}

interface BaseItem {
  id: string;              // stable, never changes (used in the database)
  section: string;         // e.g. "salah"
  level: Level;
  title: Record<LangCode, string>;
  madhab: Madhab[] | "all";
  gender?: "male" | "female";   // only if the ruling is gender-specific
  review?: Review;         // missing = hidden from users
}

export interface Dua extends BaseItem {
  type: "dua";
  arabic: string;
  transliteration: string;
  translations: Partial<Record<LangCode, string>>;
  audio: string;           // path under /public
  wordTimings?: number[];  // start time (ms) of each word, for highlighting
  when: Record<LangCode, string>;   // "Recited while sitting in ..."
  source: string;          // "Sahih al-Bukhari 831"
  grading: "Sahih" | "Hasan" | "Other";
  recitable: boolean;
  repeat?: number;         // e.g. 3 for ruku tasbih
}

export interface SurahRef extends BaseItem {
  type: "surah";
  surah: number;
  ayahs?: [number, number]; // range; omit for whole surah
  recitable: true;
}

export interface Ruling extends BaseItem {
  type: "ruling";
  body: Record<LangCode, string>;   // markdown allowed
  evidence: string[];               // Quran/hadith references
}

export interface QuizQuestion {
  id: string;
  itemId: string;
  question: Record<LangCode, string>;
  options: Record<LangCode, string>[];
  correct: number;
  explanation: Record<LangCode, string>;
}

export type Item = Dua | SurahRef | Ruling;
```

---

## 3. Review workflow

```
Draft (no `review` field)  →  Scholar reviews  →  `review` added  →  merged  →  live
```

1. A contributor writes or edits a content file on a new Git branch.
2. A pull request is opened. The PR template asks: source, grading, madhab applicability, audio check.
3. The scholar (or a trusted team member reading it to them) reviews the text, translation, source, and audio.
4. On approval, the `review` field is added with the scholar's name and date. Any later edit to `arabic`, `translations`, `source`, or `body` must remove the `review` field until re-approved (enforced by a CI check).
5. Merge to `main` → Vercel deploys automatically.

`content/index.ts` filters out any item without `review`, so unreviewed content can never appear in the app.

### CI checks (GitHub Actions)

- Every `id` is unique.
- Every `audio` path exists in `/public`.
- Every `reviewedBy` name exists in the scholar list.
- Every quiz `itemId` points to a real item.
- Every item has at least an English title and translation.

---

## 4. Level 1 content list (MVP)

Every source below is the commonly cited reference. **The scholar must verify each one before approval.** Some items differ by madhab; the scholar decides which versions to include.

### Aqeedah (knowledge, quiz-based)

| ID | Item | Type |
|---|---|---|
| `aqeedah-shahada` | The Shahada and its meaning | Dua + ruling |
| `aqeedah-iman-pillars` | Six pillars of Iman: belief in Allah, His angels, His books, His messengers, the Last Day, and qadr (divine decree), its good and its bad | Ruling |
| `aqeedah-islam-pillars` | Five pillars of Islam: Shahada, salah, zakat, fasting Ramadan, Hajj (for those able) | Ruling |
| `aqeedah-tawhid` | Tawhid basics | Ruling |
| `aqeedah-shirk` | What shirk is and how to avoid it | Ruling |
| `aqeedah-ihsan` | Ihsan: to worship Allah as if you see Him; and if you do not see Him, He sees you (Hadith Jibril, Sahih Muslim 8) | Ruling |

### Taharah

| ID | Item | Type |
|---|---|---|
| `taharah-wudu-steps` | How to perform wudu | Ruling + walkthrough |
| `taharah-wudu-breakers` | What breaks wudu | Ruling |
| `taharah-dua-before-wudu` | Bismillah before wudu | Dua |
| `taharah-dua-after-wudu` | Shahada after wudu | Dua |
| `taharah-ghusl` | When ghusl is required and how | Ruling |
| `taharah-tayammum` | Tayammum | Ruling |
| `taharah-istinja` | Istinja and toilet duas | Ruling + duas |

### Salah

| ID | Item | Type | Commonly cited source |
|---|---|---|---|
| `salah-times` | Prayer times and number of rakahs | Ruling | |
| `salah-conditions` | Conditions of salah | Ruling | |
| `salah-breakers` | What breaks salah | Ruling | |
| `salah-takbir` | Allahu Akbar (opening takbir) | Dua | |
| `salah-sana` | Opening supplication (Sana) | Dua | Verify version per madhab |
| `salah-taawwudh` | Ta'awwudh | Dua | Quran 16:98 |
| `salah-fatiha` | Al-Fatiha | Surah ref | Quran 1 |
| `salah-ruku` | Ruku tasbih | Dua | Verify |
| `salah-rising` | Sami'allahu liman hamidah / Rabbana lakal hamd | Dua | Verify |
| `salah-sujood` | Sujood tasbih | Dua | Verify |
| `salah-between-sujood` | Dua between two sajdahs | Dua | Verify |
| `salah-tashahhud` | At-Tahiyyat | Dua | Sahih al-Bukhari 831 (verify) |
| `salah-durood` | Durood Ibrahim | Dua | Sahih al-Bukhari 3370 (verify) |
| `salah-before-salam` | Dua before salam | Dua | Sahih al-Bukhari 834 (verify) |
| `salah-salam` | Closing salam | Dua | |
| `salah-qunut` | Dua Qunut (Witr) | Dua | Version differs by madhab |
| `salah-sahw` | Sajdah sahw | Ruling | |
| `salah-after-adhkar` | Adhkar after salah | Duas | Verify |
| `salah-jumuah` | Jumu'ah basics | Ruling | |

### Quran minimum

| ID | Item |
|---|---|
| `quran-fatiha` | Al-Fatiha (1) |
| `quran-ayatul-kursi` | Ayatul Kursi (2:255) |
| `quran-105` … `quran-114` | Last 10 surahs (Al-Fil to An-Nas) |

### Sawm

| ID | Item | Type |
|---|---|---|
| `sawm-rules` | Who must fast, niyyah | Ruling |
| `sawm-breakers` | What breaks the fast | Ruling |
| `sawm-exemptions` | Exemptions and making up | Ruling |
| `sawm-iftar-dua` | Dua at iftar | Dua (verify) |

### Janazah

| ID | Item | Type |
|---|---|---|
| `janazah-hearing-death` | Inna lillahi wa inna ilayhi raji'un | Dua (Quran 2:156) |
| `janazah-prayer-steps` | The 4 takbirs and what follows each | Ruling + walkthrough |
| `janazah-dua-adult` | Dua for the deceased (adult) | Dua (verify) |
| `janazah-dua-child` | Dua for a deceased child | Dua (verify) |
| `janazah-ghusl` | Washing the deceased (basics) | Ruling |
| `janazah-graves` | Dua when visiting graves | Dua (verify) |
| `janazah-condolence` | How to give condolence | Ruling + dua |

### Daily duas

| ID | Occasion |
|---|---|
| `daily-waking` | On waking up |
| `daily-sleeping` | Before sleeping |
| `daily-eating-before` / `daily-eating-after` | Before and after eating |
| `daily-toilet-enter` / `daily-toilet-exit` | Entering and leaving the toilet |
| `daily-home-leave` / `daily-home-enter` | Leaving and entering home |
| `daily-masjid-enter` / `daily-masjid-exit` | Entering and leaving the masjid |
| `daily-travel` | Travel dua |
| `daily-clothes` | Wearing clothes |
| `daily-morning-evening` | Core morning and evening adhkar |

### Halal and haram basics

| ID | Item |
|---|---|
| `halal-food` | Halal and haram food basics |
| `halal-major-sins` | The major sins |
| `halal-awrah` | Awrah for men and women |
| `halal-parents` | Rights of parents |
| `halal-others` | Rights of others (neighbours, relatives) |

---

## 5. Audio guidelines (dua reference recordings)

- One qari per language region at minimum; Husary-style slow, clear recitation.
- Recorded in a quiet room, MP3 at 64 kbps mono (small files, ~30 KB per 5 seconds).
- Keep each file under 200 KB so the project stays light.
- Include word timings (`wordTimings`) so the app can highlight words during playback. They can be generated automatically with forced alignment and checked by hand.
- The reciter must sign written permission for use in the app.

---

## 6. Translation guidelines

- Translations of duas are written by the team and approved by the scholar.
- Quran translations are NOT written by the team: they come from established translators via the APIs, with the source name shown.
- Keep translations simple and literal enough that a learner can match words to meaning.

---

## 7. Seasonal content

```
content/
├── seasons.ts            → season definitions (see FUNCTIONALITY.md §13)
└── seasonal/
    ├── ramadan/          → rulings, duas, 29/30-day path (follows the actual month), laylatul-qadr.ts, zakat-al-fitr.ts
    ├── eid/              → eid-prayer-walkthrough.ts, takbir.ts, sunnahs.ts
    ├── dhul-hijjah/      → first-10-days.ts, arafah.ts, udhiyah.ts, takbirat.ts
    ├── hajj-umrah/       → walkthrough.ts (each rite + dua, pronunciation, meaning)
    ├── ashura/           → fasting.ts
    └── jumuah/           → kahf-ref.ts, rulings.ts
```

Every seasonal item has `season: "ramadan" | "eid-fitr" | ...` and the same review rules as all content. Ramadan content must be reviewed **at least 3 weeks before Ramadan**.

| ID | Item | Type |
|---|---|---|
| `ramadan-niyyah` | Intention for fasting | Ruling |
| `ramadan-suhoor` | Suhoor sunnah and timing | Ruling |
| `ramadan-iftar-dua` | Dua at iftar | Dua (verify) |
| `ramadan-taraweeh` | Taraweeh guide (per madhab) | Ruling + walkthrough |
| `ramadan-laylatul-qadr-dua` | Dua for Laylatul Qadr | Dua (verify) |
| `ramadan-itikaf` | I'tikaf basics | Ruling |
| `ramadan-zakat-fitr` | Zakat al-Fitr | Ruling |
| `eid-prayer` | Eid prayer steps (per madhab) | Walkthrough |
| `eid-takbir` | Takbir of Eid | Dua |
| `dhulhijjah-arafah` | Fasting the day of Arafah | Ruling |
| `dhulhijjah-udhiyah` | Qurbani rules | Ruling |
| `hajj-umrah-steps` | Hajj and Umrah step by step | Walkthrough |
| `ashura-fasting` | Fasting 9th and 10th Muharram | Ruling |
| `jumuah-kahf` | Surah Al-Kahf on Friday | Surah ref (18) |

---

## 8. Faith content

```
content/
├── cards/              → home hadith/ayah cards (topics: salah, dua, patience, parents, ...)
├── new-muslim/         → shahada.ts, kalimas.ts, path.ts
├── salah-attachment/   → why.ts, understand.ts, steps.ts, heart.ts, falling-back.ts
├── digital/            → guarding-eyes.ts, value-of-time.ts, phone-guides.ts, dhikr-card.ts
└── seasonal/ramadan/highlights.ts → must / do-more / must-not groups
```

Rules:

- Cards: Sahih or Hasan only, with collection + number, scholar-reviewed.
- Kalimas: scholar decides wording, order, and explanatory note. Shahada marked as the essential one.
- Every item, same review workflow (§3).

## 9. Motivation and daily-life content

All drafts (no `review`), listed in `docs/REVIEW_QUEUE.md`.
- `content/motivation/`: `by-mood.ts` (mood → cards/duas), `prayer-why.ts` (pre-salah reasons), `nudges.ts` (15 adhan-time lines with sources).
- `content/sunnahs.ts`: 16 sunnahs of the day.
- `content/feelings.ts`: 10 `feel-*` duas and the 12-feeling map with comfort ayahs.
- `content/daily-life.ts`: praying at work/university, visiting the sick, ruqyah (Quran and Sunnah only).
- `content/zakat-config.ts`: nisab basis per madhab; set `reviewed: true` after sign-off.
- Istikhara walkthrough in `content/walkthroughs-more.ts`.
- Cards gain `publishedAt` for the "New" label.
