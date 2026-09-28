# Layout

## Principle

Design every screen at **390px wide first** (a typical phone). Larger screens add space and side panels, but never new core features.

---

## Breakpoints

| Name | Width | Navigation | Content |
|---|---|---|---|
| `mobile` | < 640px | Bottom tab bar | Single column, 16px padding |
| `tablet` | 640–1023px | Left icon rail (72px) | Single column, max 640px, centered |
| `desktop` | ≥ 1024px | Left sidebar (248px) with labels | Centred grid (max 1800px): fluid main column + right panel 320–400px, 32–40px gaps |
| `wide` | ≥ 1440px | Same as desktop | Whole layout max 1320px, centered |

---

## App shell

### Mobile

```
┌─────────────────────────────┐
│ Top bar (56px)              │  title · streak chip · settings
├─────────────────────────────┤
│                             │
│   Scrollable content        │
│                             │
│                             │
├─────────────────────────────┤
│ Tab bar (68px + safe area)  │  Essentials · Quran · (TODAY) · Review · Profile
│   raised 64px Today button  │  sits in a curved notch, 20px above the bar
└─────────────────────────────┘
```

### Desktop

```
┌──────────┬───────────────────────────────┬──────────────┐
│ Sidebar  │ Top bar (title, streak)       │ Right panel  │
│ 240px    ├───────────────────────────────┤ 320px        │
│          │                               │ · Daily goal │
│ Logo     │   Main content (max 680px)    │ · Due today  │
│ Today    │                               │ · Fix next   │
│ Essent.  │                               │ · Streak     │
│ Quran    │                               │              │
│ Review   │                               │              │
│ Profile  │                               │              │
│ ──────   │                               │              │
│ Settings │                               │              │
└──────────┴───────────────────────────────┴──────────────┘
```

---

## Navigation

### Tabs (same five everywhere)

| Tab | Icon | Route | Mobile position |
|---|---|---|---|
| Essentials | duotone star | `/essentials` | 1 |
| Quran | duotone book | `/quran` | 2 |
| **Today** | live calendar (`components/icons/CalendarToday.tsx`) | `/today` | center, raised button |
| Review | duotone cycle | `/review` | 4 |
| Profile | duotone shield | `/profile` | 5 |

Tablet rail and desktop sidebar: **Today first**, then Essentials, Quran, Review, Profile. Desktop keys 1–5 jump to them (tooltips show the key).
The Learn tab is gone: `/learn` redirects to `/today`; the full learning path is at `/essentials/path`; lessons stay at `/learn/[lessonId]` (focus mode).

### Focus mode

Lesson, recite, and onboarding screens are **focus mode**: tab bar and sidebar hidden, only a close (×) button and a progress bar at the top. Closing a lesson mid-way asks: "Leave this lesson? Your progress in it won't be saved."

### Back behavior

- Nested screens (surah page, section detail) show a back arrow in the top bar.
- Android back button / browser back works on every screen.

---

## Routes

```
/                        → redirects to /today (or /welcome for first visit)
/welcome                 → onboarding flow (focus mode)
/today                   → Today: date, ayah/hadith, today's lesson + goal, prayer card, tool chips, sunnah, mood
/learn                   → redirects to /today
/essentials/path         → the full learning path
/learn/[lessonId]        → lesson player (focus mode)
/essentials              → Essentials dashboard
/essentials/[section]    → section detail (e.g. /essentials/salah)
/essentials/[section]/walkthrough → salah/janazah step-by-step
/item/[itemId]           → single dua/surah/ruling page
/item/[itemId]/recite    → recitation practice (focus mode)
/quran                   → surah list
/quran/[surah]           → surah reader
/quran/[surah]/[ayah]    → reader scrolled to ayah
/review                  → due today + weak items
/profile                 → score, streak, badges
/profile/settings        → all settings
/about/scholars          → scholar board + sources
/admin                   → admin (role-restricted)
```

---

## Grid and spacing

- Mobile: single column, 16px side padding, 16px gap between cards, 24px between sections.
- Card grids (Essentials sections): 2 columns on mobile, 3 on tablet, 3 in the desktop main column.
- Vertical rhythm: section title → 12px → content → 32px → next section.

---

## Safe areas and PWA

- Respect `env(safe-area-inset-*)` for the notch and home indicator.
- Viewport: `width=device-width, initial-scale=1, viewport-fit=cover`.
- The tab bar sits above the home indicator; the mic button sits above the tab bar or at the bottom in focus mode.
- Standalone PWA mode hides the browser UI, so the top bar must include everything needed (back, title).

---

## RTL support

- When the UI language is RTL (Arabic, Urdu), the whole layout mirrors: tab order, back arrow direction, sidebar on the right.
- Arabic Quran/dua text is always RTL regardless of UI language.
- Use CSS logical properties (`margin-inline-start`, `padding-inline-end`) everywhere so mirroring works automatically.

## Website routes (app/(site))

```
/ /features /features/[slug] /how-it-works /pronunciation /scholars /content-policy
/about /gallery /free /faq /contact /volunteer
/blog[/slug] /changelog[/slug] /press
/privacy /terms /cookies /children /voice-data /community-guidelines /accessibility /security /delete-account
```

Same paths under `/bn`, `/ar`, `/ur` (proxy rewrite). Own navbar + footer; the app shell is not used.
