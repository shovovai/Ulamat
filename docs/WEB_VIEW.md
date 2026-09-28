# Web view (tablet and desktop)

The web view adapts the mobile app. Same features, same flows, same components. Extra width is used for side panels and larger text, not new functions.

See LAYOUT.md for breakpoints and the app shell.

---

## General adaptation rules

| Mobile | Tablet (640–1023) | Desktop (≥ 1024) |
|---|---|---|
| Bottom tab bar with raised Today button | Left icon rail (76px), Today first as a 52px raised rounded square | Left sidebar (248px), Today first: 32px calendar icon, label and the full date |
| Full-width buttons | Buttons max 400px, centered | Buttons auto width, left-aligned in forms |
| Bottom sheets | Bottom sheets | Right-side drawers (400px) or centered dialogs |
| Single column | Single column, max 640px | Main column max 680px + right panel 340px |
| Tap / long-press | Tap / long-press | Click, hover states, right-click = ayah menu |
| 2-column card grid | 3 columns | 3 columns |

### Keyboard shortcuts (desktop)

| Key | Action |
|---|---|
| `Space` | Start / stop recording on recite screens; play/pause in the reader |
| `Enter` | Continue / Check |
| `1`–`4` | Choose quiz option |
| `L` | Listen to reference |
| `←` / `→` | Previous / next ayah or walkthrough step |
| `Esc` | Close sheet / leave lesson (with confirmation) |
| `/` | Focus search (Quran list) |

---

## 1. Learn (home)

```
┌──────────┬──────────────────────────────────┬────────────────────┐
│ ✦ [App]  │ Learn                    🔥 12   │ Daily goal         │
│          ├──────────────────────────────────┤ ▓▓▓▓▓▓░░░ 6/10 min │
│ ▸ Learn  │ Unit 2 · Salah recitations       │                    │
│   Essent.│                                  │ Due today (3)      │
│   Quran  │           ✦ Sana                 │ · Ruku tasbih      │
│   Review │        ✦    Al-Fatiha            │ · Al-Ikhlas        │
│   Profile│           ✧ Ruku tasbih          │ · Sana             │
│          │        ☆    Tashahhud            │ [Review]           │
│          │           ☆ Durood               │                    │
│          │                                  │ Fix this next      │
│ ──────── │ Unit 3 · Janazah                 │ Janazah duas       │
│ Settings │           ☆ ...                  │ [Start]            │
└──────────┴──────────────────────────────────┴────────────────────┘
```

The right panel replaces the mobile "Today" card.

---

## 2. Lesson player and recitation (focus mode)

Sidebar and right panel hidden. Content centered, max 640px wide, with larger Arabic (`arabic-quran` 34px).

```
┌──────────────────────────────────────────────────────────────────┐
│ ×                    ▓▓▓▓▓▓▓░░░░░                                │
│                                                                  │
│                         Now you say it                           │
│                                                                  │
│                 التَّحِيَّاتُ لِلَّهِ وَالصَّلَوَاتُ                     │
│                                                                  │
│               [🔊 Listen]   [🐢 Slow]                            │
│                                                                  │
│                          ( 🎤 )                                  │
│                 Press Space or click to record                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

Desktop microphones vary: show an input-device picker the first time and in settings.

---

## 3. Essentials dashboard

```
┌──────────┬──────────────────────────────────┬────────────────────┐
│ sidebar  │ Essentials                       │ Your Learning Score│
│          │ What every Muslim must know      │      ╭────╮        │
│          │                                  │      │ 58 │        │
│          │ ┌──────────────────────────────┐ │      ╰────╯        │
│          │ │ Fix this next: Janazah duas  │ │ Must know   70%    │
│          │ │ [Start]                      │ │ Life        25%    │
│          │ └──────────────────────────────┘ │ Growth      10%    │
│          │                                  │                    │
│          │ Must know                        │ Unlock for your    │
│          │ ┌────────┐┌────────┐┌────────┐   │ life               │
│          │ │Aqeedah ││Taharah ││ Salah  │   │ ☐ I have savings   │
│          │ │  60%   ││  45%   ││  25%   │   │ ☐ Planning Hajj    │
│          │ └────────┘└────────┘└────────┘   │ ☐ Married / soon   │
│          │ ┌────────┐┌────────┐┌────────┐   │ ☐ Business owner   │
│          │ │Janazah ││Quran   ││Fasting │   │ ☐ Parent           │
│          │ │   0%   ││  80%   ││  30%   │   │ ☐ Traveller        │
│          │ └────────┘└────────┘└────────┘   │                    │
│          │ ┌────────┐┌────────┐             │                    │
│          │ │Duas    ││Halal / │             │                    │
│          │ │  20%   ││haram   │             │                    │
│          │ └────────┘└────────┘             │                    │
└──────────┴──────────────────────────────────┴────────────────────┘
```

---

## 4. Section detail

Desktop uses two columns inside the main area: rulings on the left, recitations on the right. The "Pray step by step" walkthrough button spans the full width above them.

---

## 5. Salah walkthrough

```
┌──────────────────────────────────────────────────────────────────┐
│ ×   Salah step by step                        Step 4 of 12       │
├───────────────────────┬──────────────────────────────────────────┤
│ 1 Intention           │                                          │
│ 2 Takbir              │   [posture outline]                      │
│ 3 Standing (Qiyam)    │                                          │
│ ▸ 4 Ruku              │   Ruku                                   │
│ 5 Rising              │   Bow with your back straight...         │
│ 6 Sujood              │                                          │
│ 7 Sitting             │   سُبْحَانَ رَبِّيَ الْعَظِيمِ                 │
│ 8 Second sujood       │   subḥāna rabbiyal-ʿaẓīm                 │
│ ...                   │   Glory be to my Lord, the Most Great.   │
│                       │                                          │
│                       │   [🔊 Listen]  [🎤 Practice]              │
│                       │                    ‹ Prev    Next ›      │
└───────────────────────┴──────────────────────────────────────────┘
```

The step list on the left replaces the mobile dots; clicking any step jumps to it.

---

## 6. Quran reader

```
┌──────────┬──────────────────────────────────┬────────────────────┐
│ sidebar  │ ← Al-Mulk                 Aa  ⋯  │ Tafsir · 67:1      │
│          ├──────────────────────────────────┤ [Al-Mukhtasar ▾]   │
│          │   بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ        │                    │
│          │                                  │ (tafsir text for   │
│          │ تَبَارَكَ الَّذِي بِيَدِهِ الْمُلْكُ وَهُوَ   │  the selected      │
│          │ عَلَىٰ كُلِّ شَيْءٍ قَدِيرٌ ①           │  ayah)             │
│          │ Blessed is He in whose hand...   │                    │
│          │ ──────────────────────────────── │ Source: Tafsir     │
│          │ (next ayah)                      │ Center for Quranic │
│          │                                  │ Studies            │
│          ├──────────────────────────────────┤                    │
│          │ ▶ Husary ▾  ayah 1/30  1×  ⟳     │                    │
└──────────┴──────────────────────────────────┴────────────────────┘
```

- The right panel shows the tafsir (or word-by-word details) for the selected ayah, so the user never leaves the text.
- Hovering an Arabic word shows its meaning tooltip; clicking plays it.
- The surah list is available as a collapsible panel via a "Surahs" button next to the title.

---

## 7. Review, profile, settings

- **Review:** due list and "needs practice" list side by side (2 columns).
- **Profile:** score ring and section list on the left; streak calendar (month heatmap in lapis shades) and badges on the right.
- **Settings:** left list of groups, right pane with the selected group's settings (classic settings layout).

---

## 8. Landing page (logged-out visitors on desktop)

Visitors who aren't signed in and arrive on desktop see a landing page before the app:

```
┌──────────────────────────────────────────────────────────────────┐
│ ✦ [App name]                         Our scholars   [Open app]   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Learn what every Muslim must know,                              │
│  and say it correctly.                                           │
│                                                                  │
│  [ Try it: recite Al-Fatiha now ]   ← live demo of the checker   │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│  Must-know first · Checked by AI, reviewed by scholars · Free    │
├──────────────────────────────────────────────────────────────────┤
│  How it works (3 real screenshots from the mobile app)           │
├──────────────────────────────────────────────────────────────────┤
│  Our scholars and sources                                        │
├──────────────────────────────────────────────────────────────────┤
│  Install on your phone: scan QR / Add to home screen guide       │
└──────────────────────────────────────────────────────────────────┘
```

The hero's main action is the live recitation demo: the core feature, working in the first 10 seconds.

On mobile, the root URL goes straight to the app (welcome screen), not the landing page.

## Core features (Parts 14–18) on tablet and desktop

- Recite, answers and scholar pages use the centre column with the right panel; the scholar editor is a single-column form with two-column grids from `sm`.
- Sidebar tools: Prayer times, Qibla, Adhkar, Hijri calendar, Journal, Answers, Seasons; Settings pinned at the bottom.
- Keyboard: recite keeps Space = record, L = listen; the letter sheet closes with Esc.

## Today tab (desktop and tablet)

- Rail/sidebar order: Today, Essentials, Quran, Review, Profile. Hover: tinted background, icon lifts 2px, label brightens, tooltip "Label · Shortcut: n". Keys 1–5 navigate (ignored while typing). Focus-visible ring on every item.
- Desktop Today item: calendar icon (live weekday + date, changes at midnight) with "শনিবার, ২৬ সেপ্টেম্বর" under the label.
- The Today page centre holds the date line, tools, sunnah card and "Today's lesson"; hadith, prayer, circles and mood stay in the right panel (hadith first).

## Public website

- Navbar: sticky, transparent over the hero, solid + blur after 16px. Desktop: Features mega menu (2 columns + preview card), dropdowns, language select, Open app. Phones: full-screen sheet.
- Pages share `components/site/ui.tsx`: PageHero (breadcrumbs, star pattern), Section, CtaBand, FaqList, Toc (sticky on desktop), LiftCard.
- Scroll reveal: CSS + one IntersectionObserver (`RevealObserver`), once, off with reduced motion. Mini screens (`MiniScreen`) animate when revealed.
- Legal/blog: reading progress bar (CSS scroll timeline), print styles.

## Desktop grid (2026-09-26)

- One centred container that fills the screen up to 1800px. Grid `[sidebar 232px] [main, fluid] [right panel 320px]` at lg, 256/380 at xl, 272/400 at 2xl; 32–40px gaps. The Quran reader drops the right panel (its own tafsir panel).
- Sidebar: sticky, top 0, `height: 100dvh`. Logo and the five main items are fixed at the top, the tools list scrolls on its own (thin quiet scrollbar), Settings is pinned at the bottom. Main rows 48px (14px uppercase labels, 28px icons), tool rows 40px (14px), clear gaps between groups. Below 800px viewport height rows get compact (42px main rows, 36px tool rows, 20px icons). Scrollbars stay hidden until hover.
- Header row over main + right panel: the date with Hijri on the start side; season chip, goal ring, streak and the language menu grouped on the end side with 12px gaps. The page's own date line hides on desktop.
- Right panel: sticky at top 24px, `max-height: calc(100dvh - 48px)`, scrolls on its own with no visible scrollbar (only the page scrollbar shows).
- Today's lesson card: content centred (star, title, progress, goal ring, button up to 360px, path link); quick tools centred under it.
- Main cards use the full column width; the quick tools row wraps on desktop (scrolls sideways on phones).
- No status dot on the Today button.
- Checked at 1280, 1440 and 1920 wide and at 100/90/80/67% browser zoom.
