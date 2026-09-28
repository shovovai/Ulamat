# Design

## Design direction (v2)

**Bright, friendly and tactile — like Duolingo — while staying respectful for Islamic content.**

- Solid, saturated colors with a clear lapis-blue primary. Surfaces are clean white on a soft paper background.
- Everything tappable feels physical: a darker bottom "depth" edge that presses down when tapped.
- Rounded shapes (16–24px radii), bold rounded type (Nunito).
- The Quran and duas are always shown calmly: large, well-spaced Arabic, never animated, never on busy backgrounds.

**Deliberately avoided:**

- The green-and-gold look used by most Islamic apps.
- Mascots or any character with a face, people or animals. The app's "character" is the **8-pointed star (khatam)**.
- Music or melodic sounds. Confetti rectangles.
- Red on recitation mistakes (always amber).

**The memorable element:** the 8-pointed star. Lesson nodes are stars that fill with lapis and gain a gold edge when completed. Stars are also used for the app icon, empty states, milestone celebrations (small star particles) and subtle decoration on the landing page.

---

## Color tokens

Defined as CSS variables in `app/globals.css` and mapped to Tailwind utilities with `@theme` (Tailwind v4 has no `tailwind.config`). Every color has a fill (`bg-primary`), a depth shade for 3D edges, a soft tint for selected/tinted backgrounds, a text-safe **ink** version (≥ 4.5:1 on paper and surface), and an `on-*` color for text on the fill.

### Light

| Token | Fill | Depth (shadow) | Soft tint | Ink (text) | On-fill text |
|---|---|---|---|---|---|
| `primary` (lapis) | `#2B59C3` (hover `#244BA8`) | `#1B3A85` | `#E8EEFB` | `#2B59C3` | `#FFFFFF` |
| `success` | `#3DB46D` | `#2A8C52` | `#E3F6EA` | `#1E7A45` | `#0B3B20` |
| `streak` (sunflower) | `#FFB020` | `#D98E00` | `#FFF4DC` | `#7A5A00` | `#3D2600` |
| `attention` (amber, recitation mistakes) | `#F08A24` | `#C46A10` | `#FDEEDD` | `#A85300` | `#3A1D00` |
| `gold` (achievements only) | `#E3B341` | — | `#FBF1D6` | `#7A5A00` | — |
| `danger` (system errors, destructive) | `#E5484D` (button fill `#CE2C31`) | `#9E1F23` | `#FDE7E8` | `#C62F35` | `#FFFFFF` |
| `sky` (Quran accent) | `#1CB0F6` | | | `#0B6F9E` | |
| `violet` (Essentials accent) | `#8B5CF6` | | | `#6D3FE0` | |

Neutrals: `ink #1F2A44`, `ink-muted #646E89`, `paper #F7F8FB`, `surface #FFFFFF`, `line #E3E7EF`, `line-strong #CFD5E2`, `disabled #E3E7EF` / `on-disabled #8A93AA`.

Deviations from the requested palette, made for contrast (WCAG AA, 4.5:1 for body text):

- `ink-muted` is `#646E89` instead of `#6B7590` (4.32:1 on paper → 4.78:1).
- White text fails on the success, streak and amber fills, so those buttons use dark on-fill text.
- The danger *button* uses `#CE2C31` (white text 5.2:1); `#E5484D` is kept for icons, borders and outlines.

### Dark

| Token | Fill | Depth | Soft | Ink |
|---|---|---|---|---|
| `primary` | `#3A67D6` | `#22418F` | `#1F2C52` | `#8FB0FF` |
| `success` | `#4CC47E` | `#2A8C52` | `#16352A` | `#6FD69A` |
| `streak` | `#FFB020` | `#B37600` | `#3A2D10` | `#FFC555` |
| `attention` | `#F59A3C` | `#B5600C` | `#3B2615` | `#FFB86B` |
| `gold` | `#E8BF5C` | | `#3A3016` | `#F0CF7E` |
| `danger` | `#FF6B70` (button `#CE2C31`) | `#8A1B1F` | `#3B1A1D` | `#FF8A8E` |

Neutrals: `ink #EEF1F8`, `ink-muted #A3ACC4`, `paper #101624`, `surface #18213A`, `line #2A3552`, `line-strong #3A4768`.

Theme follows the system and can be overridden in settings. Any subtree can be forced with `data-theme="light|dark"` (used by `/dev/ui`).

### Section accents (Essentials)

| Section | Token | Color |
|---|---|---|
| Aqeedah | `--sec-aqeedah` | violet `#8B5CF6` |
| Taharah | `--sec-taharah` | water cyan `#14B8C4` |
| Salah | `--sec-salah` | lapis `#2B59C3` |
| Quran minimum | `--sec-quran` | sky `#1CB0F6` |
| Sawm | `--sec-sawm` | indigo `#6366F1` |
| Janazah | `--sec-janazah` | calm sage `#5B9A7C` |
| Daily duas | `--sec-duas` | rose `#E86A92` |
| Halal & haram | `--sec-halal` | teal `#0E9F8E` |
| Arabic letters | `--sec-qaida` | leaf `#7CB342` |
| Zakat / Hajj (phase 2) | `--sec-zakat` / `--sec-hajj` | `#D4A017` / `#475569` |

Accents color icons, progress rings and card edges. Text on accent backgrounds always uses `ink`.

### Color rules

- **Recitation mistakes are amber, never red.** Red is only for system errors and destructive actions.
- **Gold is earned**: completed stars, badges, milestones. The app icon's gold edge is the only brand use.
- Never rely on color alone: mistakes also get an underline style (wavy / dotted) and icons.

---

## Typography

| Role | Font | Loading |
|---|---|---|
| Latin UI (en, and Latin inside ar/ur) | **Nunito** 400/600/700/800 | `next/font/google`, preloaded |
| Bengali UI | **Hind Siliguri** 400/500/600/700 | `next/font/google`, loaded when Bengali is on screen |
| Arabic / Urdu UI, duas | **Noto Naskh Arabic** 400/500/700 | `next/font/google` |
| Urdu translations | **Noto Nastaliq Urdu** | `next/font/google` |
| Quran — Uthmani | **KFGQPC Uthmanic Script HAFS** | `next/font/local`, shipped unmodified (`app/fonts/KFGQPC-LICENSE.txt`) |
| Quran — IndoPak | Placeholder: Noto Naskh Arabic | IndoPak Nastaleeq font blocked on licence (see BLOCKERS.md) |

The UI font switches automatically with `lang`: the `<html lang>` and any `[lang]` subtree. Line height grows for Bengali (1.65) and Arabic script (1.8).

### Type scale

| Token | Size / line height | Weight |
|---|---|---|
| `t-display` | 28/36 (desktop 36/44) | 800 |
| `t-title` | 22/30 | 800 |
| `t-heading` | 18/26 | 700 |
| body | 16/24 | 400 |
| `t-small` | 14/20 | 400 |
| `t-tiny` | 12/16 | 400 |
| `ar-quran` | 30 (desktop 36) / 2.1 | 400 |
| `ar-dua` | 24 / 1.9 | 400 |
| `ar-word` | 32 / 1.75 | 400 |

Sentence case everywhere. Arabic is always `dir="rtl" lang="ar" translate="no"`. Arabic and translation text scale separately (80–160%).

---

## Shape, depth and spacing

- Spacing scale: 4, 8, 12, 16, 24, 32, 48. Screen padding 16 (mobile), 24 (tablet+).
- Radii: buttons 16, inputs 16, cards 20, sheets 28 (top corners), dialogs 24, chips full.
- Borders: 2px `line` on cards, inputs and chips.
- **Depth:** tappable things have a 4px bottom edge in the darker shade (`box-shadow: 0 4px 0 var(--depth)` via the `.tactile` class). Pressing moves the element down 4px and the edge shrinks to 0 in 80ms. A box-shadow is used instead of a real border so pressing never shifts the layout.
- Floating elements (sheets, dialogs, toasts) get a soft shadow.

---

## Components (`components/ui/`)

| Component | Description |
|---|---|
| **Button** | Variants `primary`, `success`, `streak`, `secondary` (white, line border + line depth), `ghost`, `danger`. Sizes `sm` 44px, `md` 52px, `lg` 60px. Loading = spinner, keeps width. Disabled = grey, no depth. Haptic tap (`navigator.vibrate(10)`), can be turned off in settings. `ButtonLink` for navigation. |
| **Card / TapCard** | Card: 2px border, no depth. TapCard: depth + press; selected = primary border + tint. |
| **Chip / ChoiceChip** | Tones neutral, primary, success, attention, streak, gold. ChoiceChip is selectable with depth. |
| **Input / Select / Textarea / Field** | 52px, 2px border, primary border on focus, danger border when invalid. |
| **Toggle** | 52×32 switch, `role="switch"`. |
| **ProgressBar** | 16px, rounded, glossy highlight and a moving stripe on the filled part. `sm` 10px variant. |
| **ProgressRing** | Accent-colored stroke on a line track, value in the center. |
| **Sheet** | Bottom sheet (mobile) / 400px side drawer (desktop). Optional success/attention tint. |
| **Dialog / useConfirm** | Centered confirmation; replaces `window.confirm`. |
| **Toast** | Pill with icon, above the tab bar, 3s. |
| **Skeleton** | Shimmering placeholders shaped like the content. No spinners except inside buttons. |
| **EmptyState** | Illustration + one sentence + one action. |
| **Star node** | 8-point star: locked, available, in progress (partial fill), complete (lapis + gold edge). |

All components are shown in every state, light and dark, in English, Bengali and Arabic at **`/dev/ui`** (development only; enable in a production build with `NEXT_PUBLIC_ENABLE_DEV_UI=true`).

---

## Today button and live calendar icon

- `CalendarToday`: rounded-square page, two binder rings, a strip with the short weekday in the reader's language (bn শনি, en SAT, ar السبت), and the date in the reader's digits (২৬ / 26 / ٢٦). `tone="light"` draws it white on the button. Updates at local midnight (`lib/useToday.ts`).
- `.today-button` (globals.css): gradient `primary → color-mix(primary 72%, white)` at 135°, inset 1.5px white highlight, colored drop shadow `0 8px 20px primary/35%` when active (`0 6px 16px` otherwise, `0 3px 8px` while pressed). `.today-glow`: a single ring pulse on arrival; off with reduced motion.
- Notch: SVG path in a 120×68 segment (fill `--surface`, 2px `--line` stroke following the curve) plus a 76px circle of `--paper` behind the button, so the cutout matches the page in light and dark themes.

## Motion

- Quick and physical: presses 80ms, fades 150ms, sheets 180ms; springs for pops (stiffness 400, damping 30).
- Allowed: tab icon pop, bobbing "Start" bubble on the next lesson, progress sparkle, word-by-word coloring of recitation results (40ms stagger, reading order), count-ups, star-particle celebrations.
- Not allowed: confetti rectangles, looping motion near Quran text, animating the Quran text itself.
- `prefers-reduced-motion`: all animation off, final states shown immediately.

---

## Sound and haptics

- **No music anywhere.** Every sound is synthesised in `lib/sounds.ts` — there are no audio files —
  and none of it is a tune, none of it loops, and none of it plays by itself.
- **Sound answers an outcome, never a tap.** Buttons, navigation, toggles, the mood check: silent.
  A chime on every tap is noise, and the whole thing then gets switched off.

  | Sound | When |
  | --- | --- |
  | `correct()` — a fifth up | A right answer; a recitation at or above its pass mark |
  | `again()` — lower, quieter, falling | A wrong answer; a recitation below it. It says "not that one", never "bad" |
  | `complete()` — three rising notes | A finished lesson, the day's goal, a badge, the review queue emptied, a tasbih or dhikr target reached |
  | `tick()` — dry, pitchless tap | Each count, each thing ticked off: the tasbih, the adhkar, the salah check-in, the daily checklist, the qibla lining up |

- On by default; Settings → Sounds turns all of it off. A sound nobody switches on is not feedback.
- Haptics ride alongside: 10 ms on buttons, `[15, 40, 15]` for a celebration. Same toggle idea, own setting.
- The only audio *content* is recitation and the user's own voice.

---

## Tone of voice

Warm, plain, respectful. Short sentences. Encourage without flattery, correct without shame.

| Situation | Write | Avoid |
|---|---|---|
| Recitation with mistakes | "Good effort. 3 words need practice." | "Wrong! 3 mistakes." |
| Streak lost | "Welcome back. Start a new streak today." | "You lost your streak!" |
| Low score | "Let's strengthen this one." | "You failed." |
| Empty review list | "Nothing to review today. Learn something new?" | "No items." |

---

## Accessibility

- Text contrast ≥ 4.5:1 (checked for every token pair above), icons and large text ≥ 3:1.
- Tap targets ≥ 44×44px. Visible focus ring (2px primary ink, 2px offset).
- Screen readers: Arabic items have labels with transliteration and meaning.
- Dark mode, text scaling, reduced motion.
