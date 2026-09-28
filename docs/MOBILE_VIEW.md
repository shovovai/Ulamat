# Mobile view

The primary experience. All wireframes are for a ~390px-wide screen. See DESIGN.md for tokens and LAYOUT.md for the shell.

---

## 1. Welcome

```
┌─────────────────────────────┐
│                             │
│            ✦ (star)         │
│                             │
│   بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ   │
│                             │
│  Learn what every Muslim    │
│  must know, and say it      │
│  correctly.                 │
│                             │
│  App language               │
│  [ English            ▾ ]   │
│                             │
│  [      Get started      ]  │
│  I already have an account  │
└─────────────────────────────┘
```

---

## 2. Onboarding question (pattern for all steps)

```
┌─────────────────────────────┐
│ ←   ▓▓▓▓░░░░░░  3/10   Skip │
│                             │
│ What's your main goal?      │
│ Choose up to 2.             │
│                             │
│ ┌─────────────────────────┐ │
│ │ ◉ Fix my salah          │ │  ← selected: lapis border + lapis-soft fill
│ └─────────────────────────┘ │
│ ┌─────────────────────────┐ │
│ │ ○ Learn daily duas      │ │
│ └─────────────────────────┘ │
│ ┌─────────────────────────┐ │
│ │ ○ Memorize Quran        │ │
│ └─────────────────────────┘ │
│ ┌─────────────────────────┐ │
│ │ ○ Prepare for Hajj      │ │
│ └─────────────────────────┘ │
│ ┌─────────────────────────┐ │
│ │ ○ Teach my kids         │ │
│ └─────────────────────────┘ │
│                             │
│ [        Continue         ] │  ← disabled until a choice is made
└─────────────────────────────┘
```

Script choice shows two image cards (Uthmani vs IndoPak sample of the same ayah) instead of text options.

---

## 3. Placement check — recite

```
┌─────────────────────────────┐
│ ×   ▓▓▓▓▓░░░░░  5/10   Skip │
│                             │
│ Recite Al-Fatiha            │
│ Take your time.             │
│                             │
│   بِسْمِ اللَّهِ الرَّحْمَٰنِ     │
│   الرَّحِيمِ ۝ الْحَمْدُ لِلَّهِ    │
│   رَبِّ الْعَالَمِينَ ۝ ...        │
│                             │
│ 🔒 Your voice is checked,   │
│    then discarded.          │
│                             │
│          ( 🎤 )             │  ← 72px mic button
│      Tap to start           │
└─────────────────────────────┘
```

Recording state: pulsing ring, live waveform under the text, timer "0:14", button becomes ■ Stop.

---

## 4. Your starting point

```
┌─────────────────────────────┐
│                             │
│   Your starting point       │
│                             │
│        ╭──────╮             │
│        │  34  │             │  ← Learning Score ring
│        ╰──────╯             │
│                             │
│ Aqeedah      ▓▓▓▓▓░░  60%   │
│ Salah        ▓▓░░░░░  25%   │
│ Janazah      ░░░░░░░   0%   │
│ Daily duas   ▓░░░░░░  10%   │
│                             │
│ We'll start with the salah  │
│ recitations.                │
│                             │
│ [   Start first lesson    ] │
└─────────────────────────────┘
```

---

## 5. Learn (home) — learning path

```
┌─────────────────────────────┐
│ Learn            🔥 12   ⚙  │
├─────────────────────────────┤
│ ┌─────────────────────────┐ │
│ │ Today: 6 / 10 min  ▓▓▓░ │ │
│ │ 3 reviews due  [Review] │ │
│ └─────────────────────────┘ │
│                             │
│ Unit 2 · Salah recitations  │
│                             │
│            ✦  Sana          │  ← complete (lapis + gold edge)
│         ✦     Al-Fatiha     │  ← complete
│            ✧  Ruku tasbih   │  ← in progress (partial fill)
│         ☆     Tashahhud     │  ← available (outline)
│            ☆  Durood        │  ← locked (grey line)
│                             │
│ Unit 3 · Janazah            │
│            ☆  ...           │
├─────────────────────────────┤
│  Learn  Essent. Quran Rev. Me│
└─────────────────────────────┘
```

Tapping a star opens a small popover: lesson name, number of steps, **Start** button.

---

## 6. Lesson player — step types

All steps share this frame:

```
┌─────────────────────────────┐
│ ×   ▓▓▓▓▓▓░░░░              │
│                             │
│   (step content)            │
│                             │
│ [        Continue         ] │
└─────────────────────────────┘
```

### 6a. Learn card

```
│ Tashahhud                   │
│ Recited while sitting in    │
│ every second and last rakah.│
│                             │
│  التَّحِيَّاتُ لِلَّهِ وَالصَّلَوَاتُ    │
│  وَالطَّيِّبَاتُ ...               │
│                             │
│ at-taḥiyyātu lillāhi ...    │  ← transliteration (toggle)
│                             │
│ All greetings, prayers and  │
│ pure words are for Allah... │
│                             │
│ Sahih al-Bukhari 831        │
│ Reviewed by [Scholar name]  │
```

### 6b. Listen

```
│ Listen carefully            │
│                             │
│  [التَّحِيَّاتُ] لِلَّهِ وَالصَّلَوَاتُ │  ← current word highlighted
│                             │
│    ◀◀    ( ▶ )    1× ▾      │
```

### 6c. Repeat / Recite

```
│ Now you say it              │
│                             │
│  التَّحِيَّاتُ لِلَّهِ               │
│                             │
│  [🔊 Listen]  [🐢 Slow]      │
│                             │
│          ( 🎤 )             │
```

### 6d. Quiz

```
│ When is the Tashahhud       │
│ recited?                    │
│                             │
│ ┌─────────────────────────┐ │
│ │ While standing          │ │
│ └─────────────────────────┘ │
│ ┌─────────────────────────┐ │
│ │ In the sitting position │ │
│ └─────────────────────────┘ │
│ ┌─────────────────────────┐ │
│ │ In ruku                 │ │
│ └─────────────────────────┘ │
│ [          Check          ] │
```

Answer feedback slides up from the bottom as a sheet: green check + "Correct" or amber + explanation, then **Continue**.

### 6e. Order the words

```
│ Put the words in order      │
│  ┌──────────────────────┐   │
│  │  (answer slots, RTL) │   │
│  └──────────────────────┘   │
│  [لِلَّهِ] [التَّحِيَّاتُ] [وَالصَّلَوَاتُ] │
```

---

## 7. Recitation result

```
┌─────────────────────────────┐
│ ×                           │
│                             │
│   Good effort. 2 words      │
│   need practice.            │
│                  Score 86   │
│                             │
│  التَّحِيَّاتُ لِلَّهِ وَالصَّلَوَاتُ    │
│             ‾‾‾‾‾ (green)   │
│  وَالطَّيِّبَاتُ  السَّلَامُ ...        │
│  ⌇⌇⌇⌇⌇ (amber = wrong)     │
│                             │
│ Tap an amber word to hear   │
│ it correctly.               │
│                             │
│ [🔊 Listen]   [↻ Try again] │
│ [        Continue         ] │
└─────────────────────────────┘
```

"Not sure" variant replaces the score with: "We couldn't hear clearly. Try again in a quieter place." and only shows **Try again**.

---

## 8. Essentials dashboard

```
┌─────────────────────────────┐
│ Essentials       🔥 12   ⚙  │
├─────────────────────────────┤
│ What every Muslim must know │
│                             │
│ ┌─────────────────────────┐ │
│ │ Fix this next           │ │
│ │ Janazah prayer duas     │ │
│ │ [ Start ]               │ │
│ └─────────────────────────┘ │
│                             │
│ Must know                   │
│ ┌──────────┐ ┌──────────┐   │
│ │  ◔ 60%   │ │  ◑ 45%   │   │
│ │ Aqeedah  │ │ Taharah  │   │
│ └──────────┘ └──────────┘   │
│ ┌──────────┐ ┌──────────┐   │
│ │  ◔ 25%   │ │  ○  0%   │   │
│ │ Salah    │ │ Janazah  │   │
│ └──────────┘ └──────────┘   │
│ ┌──────────┐ ┌──────────┐   │
│ │  ◕ 80%   │ │  ◔ 30%   │   │
│ │Quran min.│ │ Fasting  │   │
│ └──────────┘ └──────────┘   │
│ ┌──────────┐ ┌──────────┐   │
│ │  ◔ 20%   │ │  ◔ 15%   │   │
│ │Daily duas│ │Halal/har.│   │
│ └──────────┘ └──────────┘   │
│                             │
│ For your life               │
│ ┌─────────────────────────┐ │
│ │ 🔒 Zakat                │ │
│ │ Do you have savings?    │ │
│ │ [Unlock]                │ │
│ └─────────────────────────┘ │
│                             │
│ Grow further                │
│ 40 Hadith · Seerah · ...    │
├─────────────────────────────┤
│  Learn  Essent. Quran Rev. Me│
└─────────────────────────────┘
```

---

## 9. Section detail (e.g. Salah)

```
┌─────────────────────────────┐
│ ←  Salah               25%  │
├─────────────────────────────┤
│ ┌─────────────────────────┐ │
│ │ ▶ Pray step by step     │ │  ← opens walkthrough
│ └─────────────────────────┘ │
│                             │
│ Rulings                     │
│  ✓ Prayer times             │
│  ✓ Conditions of salah      │
│  ○ What breaks salah        │
│  ○ Sajdah sahw              │
│                             │
│ Recitations                 │
│  ✓ Sana              92     │  ← score
│  ✓ Al-Fatiha         88     │
│  ◐ Ruku tasbih       64     │
│  ○ Tashahhud          –     │
│  ○ Durood Ibrahim     –     │
│  ○ Dua before salam   –     │
│  ○ Dua Qunut          –     │
│                             │
│ Madhab: Hanafi (change)     │
└─────────────────────────────┘
```

---

## 10. Salah walkthrough

Horizontal swipe through positions; each card:

```
┌─────────────────────────────┐
│ ×   Step 4 of 12   ● ● ● ◉ ○│
│                             │
│   [simple line diagram of   │  ← abstract posture outline,
│    ruku posture, no face]   │     no facial features
│                             │
│ Ruku                        │
│ Bow with your back straight,│
│ hands on knees.             │
│                             │
│ Say 3 times:                │
│   سُبْحَانَ رَبِّيَ الْعَظِيمِ        │
│ subḥāna rabbiyal-ʿaẓīm      │
│ Glory be to my Lord, the    │
│ Most Great.                 │
│                             │
│ [🔊 Listen]  [🎤 Practice]  │
│                             │
│ ‹ Previous        Next ›    │
└─────────────────────────────┘
```

Note for the design team: posture diagrams should be minimal outlines without faces. Confirm with the scholar board whether illustrations are acceptable. Text-only cards are the fallback.

---

## 11. Quran — surah list

```
┌─────────────────────────────┐
│ Quran                   ⚙   │
├─────────────────────────────┤
│ [🔍 Search surah         ]  │
│                             │
│ ┌─────────────────────────┐ │
│ │ Continue: Al-Mulk 67:12 │ │
│ └─────────────────────────┘ │
│                             │
│ ① Al-Fatiha        الفاتحة   │
│   The Opener · 7 ayahs      │
│ ② Al-Baqarah       البقرة   │
│   The Cow · 286 ayahs       │
│ ③ Al-Imran         آل عمران │
│   ...                       │
└─────────────────────────────┘
```

Tabs at the top of the list: **Surah · Juz · Bookmarks**.

---

## 12. Quran — surah reader

```
┌─────────────────────────────┐
│ ←  Al-Mulk          Aa  ⋯   │
├─────────────────────────────┤
│   بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ   │
│                             │
│  تَبَارَكَ الَّذِي بِيَدِهِ الْمُلْكُ   │
│  وَهُوَ عَلَىٰ كُلِّ شَيْءٍ قَدِيرٌ ①  │
│                             │
│ Blessed is He in whose hand │
│ is the dominion, and He is  │
│ over all things competent.  │
│ Translation: [source name]  │
│                   ⋯         │
│ ─────────────────────────── │
│  (next ayah)                │
│                             │
├─────────────────────────────┤
│ ▶  Husary ▾   ayah 1/30  1× │  ← mini player (when audio on)
├─────────────────────────────┤
│  Learn  Essent. Quran Rev. Me│
└─────────────────────────────┘
```

`Aa` opens reading settings: script, Arabic size, translation size, translation on/off, word-by-word on/off, transliteration on/off.

### Ayah action sheet (tap ⋯ or long-press ayah)

```
┌─────────────────────────────┐
│ ─── (handle)                │
│ Al-Mulk 67:1                │
│ ▶  Play from here           │
│ 🔤 Word by word             │
│ 📖 Tafsir                   │
│ 🎤 Practice this ayah       │
│ 🔖 Bookmark                 │
│ ↗  Share                    │
└─────────────────────────────┘
```

### Tafsir sheet

```
┌─────────────────────────────┐
│ ───                         │
│ Tafsir · 67:1               │
│ [Al-Mukhtasar ▾]            │  ← source picker
│                             │
│ (tafsir text, scrollable)   │
│                             │
│ Source: Tafsir Center for   │
│ Quranic Studies             │
└─────────────────────────────┘
```

### Word tap popover

```
      ┌──────────────┐
      │   الْمُلْكُ    │
      │ al-mulk      │
      │ the dominion │
      │   🔊         │
      └──────────────┘
```

---

## 13. Review

```
┌─────────────────────────────┐
│ Review           🔥 12   ⚙  │
├─────────────────────────────┤
│ Due today (3)               │
│ ┌─────────────────────────┐ │
│ │ Ruku tasbih        64   │ │
│ │ Last practiced 3 days   │ │
│ └─────────────────────────┘ │
│ ┌─────────────────────────┐ │
│ │ Surah Al-Ikhlas    78   │ │
│ └─────────────────────────┘ │
│ ...                         │
│ [    Review all (3)     ]   │
│                             │
│ Memory test                 │
│ Hide the text and recite.   │
│ [ Choose a surah ]          │
│                             │
│ Needs practice              │
│  Dua Qunut          52      │
│  Ayatul Kursi       61      │
└─────────────────────────────┘
```

Empty state: star illustration + "Nothing to review today. Learn something new?" + **Go to Learn**.

---

## 14. Profile

```
┌─────────────────────────────┐
│ Profile                  ⚙  │
├─────────────────────────────┤
│  Abdullah                   │
│  Learning since Oct 2026    │
│                             │
│        ╭──────╮             │
│        │  58  │  Learning   │
│        ╰──────╯  Score      │
│                             │
│ Must know    ▓▓▓▓▓▓░░  70%  │
│ Life         ▓▓░░░░░░  25%  │
│ Growth       ▓░░░░░░░  10%  │
│ [See all sections]          │
│                             │
│ 🔥 12-day streak  (best 30) │
│                             │
│ Badges                      │
│  ✦ Salah complete           │
│  ✦ First surah memorized    │
│  ☆ Janazah ready (locked)   │
│                             │
│ This measures what you have │
│ learned in the app. Only    │
│ Allah knows the state of    │
│ anyone's faith.             │
└─────────────────────────────┘
```

---

## 15. Settings

Grouped list:

- **Learning:** madhab, daily goal, reminder time, transliteration, show translation
- **Quran:** script, reciter, translation source, tafsir source, text sizes
- **App:** language, theme (system/light/dark), sounds & haptics, honorifics style
- **Privacy:** what we store, download my data, delete account
- **About:** our scholars and sources, report a problem, version

---

## 16. States to design for every screen

| State | Pattern |
|---|---|
| Loading | Skeleton blocks in the shape of the content (no spinners for lists) |
| Empty | Star illustration + one sentence + one action |
| Offline | Top banner: "You're offline. Learning works; recitation checks need internet." |
| Error | Plain explanation + what to do + retry button |
| Guest | Profile tab shows "Create an account to save your progress" card |

---

## 17. Seasonal banner (Learn and Essentials, top)

```
┌─────────────────────────────┐
│ ☾  Ramadan · Day 12         │  ← seasonal accent background
│ Iftar in 2h 14m             │
│ Today: dua of iftar + 1 ayah│
│ [ Continue Ramadan path ]   │
│ Khatm: juz 11 of 30 ▓▓▓░░░  │
└─────────────────────────────┘
```

"Get ready" variant (before the season): "Ramadan starts in 12 days. Learn the fasting rulings now." + **Start**.

First banner of a season asks once: "Did Ramadan start today where you live?" → Yes / No, tomorrow / No, yesterday → sets the Hijri adjustment.

---

## 18. Home: hadith card and salah check-in

```
┌─────────────────────────────┐
│ Learn            🔥 12   ⚙  │
├─────────────────────────────┤
│ ┌─────────────────────────┐ │
│ │ ✦ Hadith of the moment  │ │  ← soft gradient, topic accent
│ │                         │ │
│ │  (Arabic, 2 lines)      │ │
│ │  Meaning in your        │ │
│ │  language...            │ │
│ │                         │ │
│ │ Sahih Muslim · #        │ │
│ │ [Read more]      ↗ Share│ │
│ └─────────────────────────┘ │
│                             │
│ Today's salah (private)     │
│  ● Fajr ● Dhuhr ○ Asr ○ ○   │
│                             │
│ [ Instead of scrolling →    │
│   1-minute dhikr ]          │
│                             │
│ (learning path below)       │
└─────────────────────────────┘
```

Card: swipe away = next card (max 3 per open). Share generates an image with the same design.

## 19. Daily tools (mobile first)

- **Learn order:** hadith card → next-prayer countdown → 5 circles → mood → quick tools row (Qibla, Tasbih, Feelings, Adhkar, Sleep; 44px+ chips, horizontal scroll) → sunnah/Jumu'ah/fasting cards → path. Desktop: the same order in the right panel.
- **Minimum mode:** only the 5 circles and one dua, with a small "back to normal" link.
- **Prayer:** today list (next prayer highlighted), amber masjid note, qibla link, settings card, monthly table in a collapsible section (scrolls sideways inside the card, never the page).
- **Qibla:** 256px dial centred; turns green within ±3°; tips under it.
- **Feelings:** 3-column grid of feeling tiles; tap opens duas and the comfort ayah.
- **Tasbih:** one big tap area (≥ 96px tall), count in the centre.
- **Zakat:** grouped fields (money, gold, silver, other), result card, "How this was worked out" collapsible, amber draft banner on top.
- **Calendar:** 7-column month grid, Hijri day large, Gregorian small, special days tinted.

## 20. Core features (Parts 14–18), mobile first

- **Recite result:** score ring, words (amber underline for needs work), "Tap an amber word…", **Fix this first** / **Polish later** cards, "Practice weak parts" (full-width). Letter sheet: letters at 48px in a row (RTL), tap a letter → explanation, "Play the word slowly", "Hear the letter" when the phone has an Arabic voice.
- **Learning Score (Profile):** ring + "x of y mastered", then per section two thin bars (Knowledge, Pronunciation) with "3/5" counts; footnote.
- **Invite to learn (Profile):** a card with your link (read-only field), a row of share buttons that wraps (Share, Copy link, WhatsApp, Facebook, Telegram, 44px tall), then a soft box with the private counts. Guests see a "Create account" button instead.
- **Today button:** no status dot.
- **Answers:** principle banner, search, category chips (scroll sideways), list cards, sticky-ish "Ask a question" button at the end.

## 21. Bottom bar with the raised Today button

```
        ┌────────────┐
        │  ┌──────┐  │   64px circle, 20px above the bar, 6px ring of page background
  ──────╯  │ শনি  │  ╰──────   curved notch (SVG path) — the bar wraps around the button
 জরুরি কুরআন│  ২৬  │ রিভিউ প্রোফাইল
           └──────┘
             আজ              label on the same line as the other labels
```

- Order: Essentials · Quran · **Today** · Review · Profile, two tabs on each side. RTL mirrors the side tabs; the notch, button and calendar icon stay centred and never flip.
- Button: diagonal gradient primary → lighter primary, inner top highlight, colored shadow (primary 35%, blur 20, y 8) when active, slightly less when not. Press: scale 0.92, shadow shrinks, 10ms haptic, spring back. On arrival at Today: one outer glow pulse.
- Amber dot (top-right, also in RTL) while today's salah check-in or a lesson is not done.
- Side tabs: a pill slides between them (`layoutId`), skipping the center; tap scale 0.92; active icon pops.
- Safe area: the bar's surface continues under `env(safe-area-inset-bottom)`; the button never overlaps the home indicator. Focus mode hides the whole bar.
- Reduced motion: no pops, slides or pulses.
- Today page header: "শনিবার, ২৬ সেপ্টেম্বর · ১৫ রবিউস সানি, ১৪৪৮ হিজরি" (tap → Hijri calendar); title "আজ" top-left.
- Tap Today while on Today: smooth scroll to top.

## 22. Today page, lesson first

```
┌──────────────────────────────┐
│ ✦ এই মুহূর্তের আয়াত            │  2 lines Arabic + 2 lines meaning
│ আরও পড়ুন        পরেরটি  শেয়ার │
├──────────────────────────────┤
│ আজকের পাঠ                     │  hero card
│ ★ প্রথম কালিমা · ৬টি ধাপ       │
│ ▓▓░░░░░░░░░░░                 │
│ ◯ আজ: ৩ / ৫ মিনিট              │
│ [      শুরু করুন      ]        │  56px 3D button
│        পুরো পথ দেখুন            │
├──────────────────────────────┤
│ ফজর · সকাল ৫:১২ · ২ ঘণ্টা বাকি    │  one prayer card
│ ◯ ◯ ◯ ◯ ◯                     │
│ গোপনীয়: শুধু আপনি দেখবেন         │
├──────────────────────────────┤
│ (কিবলা) (তাসবিহ) (অনুভূতি) → │  chip row, scrolls sideways
├──────────────────────────────┤
│ আজকের সুন্নাহ                  │
│ আজ আপনার ঈমান কেমন?            │
└──────────────────────────────┘
```

- Page bottom padding: 108px + safe area (bar + raised button), so the last card never sits under the button.
- Chips: 48px tall, full-colour icon circles, one-line labels, no scrollbar, edge-to-edge scroll with 16px inner gutter.
- Goal met: the button area becomes a calm success note and a secondary "আরেকটি পাঠ" button. No confetti here.
- Focus header: a small green "timer on" pill with a soft pulsing dot (static with reduced motion).
