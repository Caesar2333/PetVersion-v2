# design.md — petExtension-v2 UI/UX Design Specification

## 1. Product Temperament & Design Goals

### Dual Identity

petExtension-v2 sits at the intersection of two worlds:

- **Developer tool**: precision, clarity, information density, monospace-friendly, tabular data, terminal aesthetics
- **Browser pet toy**: warmth, delight, animation, personality, approachability

The design must honor both. The default posture is **modern-minimal precision** — crisp typography, hairline borders, restrained chrome — but with a warm accent and deliberate moments of playful animation that remind the user this isn't just another devtool.

### North Star Principles

1. **Clarity over decoration.** Every pixel must earn its place. Whitespace is the primary separator.
2. **One accent, used sparingly.** A single warm rose/coral accent appears at most twice per screen — once as the primary CTA, once as a status indicator or brand moment.
3. **Motion with purpose.** Animations are fast (150–250ms), eased, and directional. No floaty-fade-in everywhere; motion signals state change.
4. **Developer-grade density where it matters.** Player and import pages embrace information density — tables, grids, monospace numerics. Popup stays spacious and scannable.
5. **Pet is the hero.** UI chrome recedes; the pet sprite is the emotional center. Give it room to breathe in the player, and let it peek through in the popup.

---

## 2. Visual System

### 2.1 Color Tokens

All tokens expressed in OKLch for perceptual uniformity.

```css
:root {
  /* Neutral foundation */
  --bg:        oklch(99% 0.002 250);
  --surface:   oklch(100% 0 0);
  --fg:        oklch(18% 0.012 255);
  --muted:     oklch(50% 0.014 255);
  --border:    oklch(90% 0.006 255);

  /* Brand accent — warm rose, bridges dev-tool precision with pet warmth */
  --accent:           oklch(58% 0.17 20);
  --accent-foreground: oklch(100% 0 0);
  --accent-muted:     oklch(95% 0.03 20);
  --accent-hover:     oklch(52% 0.18 20);

  /* Semantic status */
  --success: oklch(62% 0.16 150);
  --warning: oklch(70% 0.15 85);
  --danger:  oklch(52% 0.18 25);

  /* Typography */
  --font-display: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'Inter', system-ui, sans-serif;
  --font-body:    -apple-system, BlinkMacSystemFont, 'SF Pro Text', 'Inter', system-ui, sans-serif;
  --font-mono:    'JetBrains Mono', 'IBM Plex Mono', ui-monospace, Menlo, monospace;
}

.dark {
  --bg:        oklch(14% 0.008 255);
  --surface:   oklch(18% 0.012 255);
  --fg:        oklch(95% 0.002 250);
  --muted:     oklch(62% 0.012 255);
  --border:    oklch(28% 0.012 255);
  --accent:    oklch(62% 0.16 20);
  --accent-muted: oklch(22% 0.06 20);
}
```

### 2.2 Typography Scale

| Step | Size | Line-height | Letter-spacing | Usage |
|------|------|-------------|----------------|-------|
| xs | 12px | 1.5 | 0 | Captions, mono labels, table footnotes |
| sm | 14px | 1.5 | 0 | Body compact, form labels, secondary nav |
| base | 16px | 1.5 | 0 | Default body, inputs, descriptions |
| lg | 20px | 1.4 | -0.005em | Card titles, section headers, step labels |
| xl | 24px | 1.3 | -0.01em | Page titles (player/import), modal headers |
| 2xl | 32px | 1.2 | -0.015em | Hero titles, popup header |
| 3xl | 48px | 1.1 | -0.02em | Landing only — not used in-page |

**Rules:**
- Display sizes (≥24px): `font-weight: 600`, `letter-spacing: -0.01em`
- Body: `font-weight: 400`
- Mono: used exclusively for code, IDs, spritesheet params, frame counts, technical labels
- All numerics in data tables use `font-variant-numeric: tabular-nums`
- Sentence case for all headings; title case only for pet display names

### 2.3 Spacing Scale

Base unit: 4px grid.

| Token | Value | Usage |
|-------|-------|-------|
| xs | 4px | Icon-to-label gap, inline status dots |
| sm | 8px | Compact padding, list gaps |
| md | 12px | Card internal gaps, form row spacing |
| lg | 16px | Card padding, section padding (phone) |
| xl | 24px | Section padding (desktop), content gaps |
| 2xl | 32px | Major section dividers |
| 3xl | 48px | Page-top spacing, hero padding |
| 4xl | 64px | Only for full-page section breathing |

### 2.4 Depth & Elevation

Two levels only, following the modern-minimal posture:

- **Level 0 (flat):** Default. Background color `--bg`. No shadow, no border-radius beyond 8px.
- **Level 1 (raised):** Dropdowns, modals, floating panels, tooltips. Uses `--surface` background, 1px `--border`, `box-shadow: 0 2px 8px oklch(0% 0 0 / 0.08)`.

No level 2+. No glassmorphism. No neumorphism.

### 2.5 Border & Radius

- **Border:** hairline `1px solid var(--border)` everywhere. No 2px borders on cards.
- **Radius:**
  - Cards, panels, inputs: `8px`
  - Buttons, badges, tags, pills: `6px`
  - Modals, dialogs: `12px`
  - Pet thumbnails in selection grid: `10px`
  - Fully rounded (pills, status dots): `999px`

### 2.6 Iconography

- **Library:** lucide-react (outline style, 1.5px stroke weight)
- **Sizes:** 16px (inline), 20px (button/control), 24px (standalone), 32px (hero/decorative)
- **Color:** Inherit from text color. Never color icons independently unless they are status indicators.
- **No emoji** as functional icons. Emoji may appear only as pet display names if the user provides them.

---

## 3. Page Architecture

### 3.1 Popup — Daily Control Panel

**Canvas:** Chrome extension popup window, ~400×520px (browser-controlled, variable).

**Structure (top → bottom):**

```
┌─────────────────────────────────┐
│  Header bar                     │
│  [Pet icon] pet name  [Toggle]  │
├─────────────────────────────────┤
│  Tab bar: [Controls] [Pets]     │
├─────────────────────────────────┤
│  Tab content area (flex: 1)     │
│                                 │
│  ── Controls tab ──             │
│  Scale slider    ─●──────── 1×  │
│  Speed slider    ──●─────── 1×  │
│  [ Reset Position ]             │
│                                 │
│  ── Pets tab ──                 │
│  ┌────┐ ┌────┐ ┌────┐          │
│  │    │ │    │ │    │ ...       │
│  │ P1 │ │ P2 │ │ P3 │          │
│  └────┘ └────┘ └────┘          │
├─────────────────────────────────┤
│  Footer actions                 │
│  [Open Player]   [Import Pet]   │
└─────────────────────────────────┘
```

**Key behaviors:**
- Toggle switch animates with a 150ms ease-out transition; the pet icon responds to enabled/disabled state (slight scale pulse on toggle)
- Sliders show current numeric value on the right, formatted with monospace numerals
- Selected pet card has a 2px accent ring + subtle scale (1.02)
- Tab switch uses a sliding underline indicator (200ms ease-out)
- Footer buttons are full-width on narrow popup, side-by-side on wider

**States:**
- **Loading:** Pet list area shows 3 skeleton cards (pulsing grey rectangles)
- **Empty (no pets):** Centered illustration + "No pets installed" + "Import your first pet" CTA
- **Error:** Inline red banner "Failed to load pets" with retry link
- **Disabled:** Entire panel gets a subtle 60% opacity overlay, toggle shows "off"

### 3.2 Player — Animation Debug Console

**Canvas:** Full browser tab, minimum 900px usable width.

**Structure (left → right, two-column):**

```
┌──────────────────────────────────────────────────────────┐
│  Top bar                                                  │
│  [← Back]  pet-selector ▾  [Scale: 1×]  [Speed: 1×]     │
│  [Bg: checkerboard ▾]                          [Dark □]  │
├──────────────────────┬───────────────────────────────────┤
│  PET VIEWPORT        │  DEBUG PANELS                     │
│  (60% width)         │                                   │
│                      │  ── State Machine ──              │
│   ┌──────────────┐   │  Current role: idle               │
│   │              │   │  Last hook: userActivity           │
│   │   [pet       │   │                                   │
│   │    sprite    │   │  ── Role Triggers (9 btns) ──     │
│   │    animation]│   │  [idle] [move-L] [move-R] [greet] │
│   │              │   │  [work] [wait] [success] [fail]   │
│   │              │   │  [special]                         │
│   └──────────────┘   │                                   │
│                      │  ── Hook Triggers ──              │
│  Background:         │  [userActivity] [settled]         │
│  checkerboard ▤▥▦▧   │  [inactive] [clickPet]            │
│                      │  [dragStart] [dragMove] [dragEnd] │
│                      │  [blur] [focus] [resize]          │
│                      │                                   │
│                      │  ── Spritesheet Info ──           │
│                      │  cols  rows  cellW  cellH         │
│                      │    9     9    256    288          │
│                      │                                   │
│                      │  ── Current Action ──             │
│                      │  role: idle → action: idle        │
│                      │  row: 0  frames: 9  loop: true    │
│                      │                                   │
│                      │  ── Action Map ──                 │
│                      │  idle      → idle                 │
│                      │  move-left → move-left            │
│                      │  move-right→ move-right           │
│                      │  greet     → greet                │
│                      │  ...                              │
└──────────────────────┴───────────────────────────────────┘
```

**Key behaviors:**
- Pet viewport has a subtle inner shadow to feel like a "stage"
- Background toggle cycles through: transparent → checkerboard (12px grid) → light solid → dark solid
- Role trigger buttons are 2×2 grid of compact pill buttons; clicking triggers the role and the button briefly flashes with the accent color
- Hook trigger buttons are smaller, grouped by source (Activity / Pointer / Drag)
- Spritesheet info displayed as a 4-column stat grid with monospace values
- ActionMap rendered as a compact table with mono action names
- All panels scroll independently; pet viewport stays fixed

**States:**
- **Loading:** Pet viewport shows skeleton rectangle with pulsing border; debug panels show skeleton rows
- **Empty (no pet selected):** Pet viewport shows "Select a pet to preview" placeholder; panels are greyed out
- **Role active:** Triggered role button stays highlighted (accent bg, white text) until animation complete or another role triggered
- **Frame tick:** Current frame number updates with a subtle number-flip animation

### 3.3 Import — Pet Import Wizard

**Canvas:** Full browser tab, centered content (max-width 720px).

**Structure (vertical step flow):**

```
┌──────────────────────────────────────────────┐
│  Step indicator                               │
│  ● Upload  ○ Review  ○ Confirm               │
├──────────────────────────────────────────────┤
│                                               │
│  ── Step 1: Upload ──                         │
│                                               │
│  ┌─────────────────────────────────┐          │
│  │       Drop files here           │          │
│  │   pet.json + spritesheet.webp   │          │
│  │        or click to browse       │          │
│  └─────────────────────────────────┘          │
│                                               │
│  ── Step 2: Review ──                         │
│                                               │
│  Detected spritesheet:                        │
│  ┌─────────────────────────────────┐          │
│  │  Preset: codex-9x9 (auto)       │          │
│  │  Size: 2304×2592  Cells: 256×288│          │
│  │  Grid: 9 cols × 9 rows          │          │
│  │  Actions found: 9                │          │
│  └─────────────────────────────────┘          │
│                                               │
│  Action Map:                                  │
│  Role          →  Action        Status        │
│  idle          →  idle          ✓ auto        │
│  move-left     →  move-left     ✓ auto        │
│  greet         →  waving        ↻ alias       │
│  special       →  —             ✗ unmapped    │
│  ...                                          │
│                                               │
│  ── Step 3: Confirm ──                        │
│                                               │
│  Summary + [Import] button                    │
│                                               │
├──────────────────────────────────────────────┤
│  [← Back]                          [Next →]   │
└──────────────────────────────────────────────┘
```

**Key behaviors:**
- Step indicator: 3 connected dots with labels. Completed steps show checkmark, current step shows accent fill, future steps are muted.
- Drop zone: dashed border (2px, accent), hover state changes to solid accent border + subtle bg tint
- File chips appear after upload showing filename + size + remove button
- ActionMap table: each row shows Role (left, bold), Action (center, mono), Status badge (right). Unmapped roles show a dropdown selector for manual mapping.
- Status badges: `✓ auto` (green bg, 10% opacity), `↻ alias` (amber), `✗ unmapped` (red), `✎ manual` (blue)
- Step transitions animate with a horizontal slide (250ms ease-in-out, fade + translateX 20px)

**States:**
- **Loading (detection):** Spritesheet preview area shows scanning animation (spinning indicator + "Analyzing spritesheet…")
- **Error (invalid file):** Drop zone turns red, shows specific error ("pet.json missing required field: id", "Spritesheet dimensions don't match any known preset")
- **Error (upload fail):** Inline red toast at top
- **Success (import complete):** Green banner + auto-navigate to step indicator showing all three steps with green checks

---

## 4. Component Catalog

### 4.1 Toggle Switch

```
[ ●────── ] off          [ ──────● ] on
```

- Width: 44px, height: 24px, thumb: 20px circle
- Track: `--border` when off, `--accent` when on
- Thumb: `--surface` with 1px shadow
- Transition: 150ms ease-out, thumb slides + track color fades
- Label: 14px, `--muted`, 8px gap to the right
- **States:** off, on, disabled (40% opacity), focused (2px accent outline, keyboard only)

### 4.2 Slider

```
Scale   ●──────────────  1.5×
```

- Track: 4px height, `--border` fill, 999px radius
- Active track: `--accent` fill up to thumb position
- Thumb: 16px circle, `--surface` with 1px border and 2px 8% shadow
- Label on left (14px, `--fg`), value on right (14px, `--muted`, mono)
- **States:** default, hover (thumb shadow deepens), active/dragging (thumb scales to 1.15, accent ring), disabled, focused
- Range labels: min and max shown below track in 11px `--muted`

### 4.3 Primary Button

```
[  Import Pet  ]
```

- Height: 36px, padding: 0 16px, radius: 6px
- Fill: `--accent`, label: `--accent-foreground`, weight: 500, size: 14px
- Hover: `--accent-hover`, 150ms color transition
- Active: scale 0.97
- **States:** default, hover, active, disabled (40% opacity), loading (spinner replaces label)
- Icon variant: 20px lucide icon, 8px gap before label

### 4.4 Secondary Button

```
[  Open Player  ]
```

- Height: 36px, padding: 0 16px, radius: 6px
- Fill: transparent, border: 1px `--border`, label: `--fg`, weight: 500, size: 14px
- Hover: bg `--accent-muted` (the 5% tint), border `--accent`
- Active: scale 0.97
- Same states as primary

### 4.5 Ghost Button (compact, icon-only or icon+label)

- Height: 28px, padding: 0 8px (icon-only: square 28px)
- No border, no bg. Label: `--muted`, 13px
- Hover: bg `--border`, label `--fg`

### 4.6 Pet Card (selection grid)

```
┌──────────────┐
│              │
│   [sprite]   │  ← 72×72 preview area, bg: checkerboard or transparent
│              │
│  pet name    │  ← 13px, centered, weight 500
└──────────────┘
```

- Size: 96×112px, radius: 10px, border: 1px `--border`, bg: `--surface`
- Hover: border `--accent`, subtle lift (translateY -1px, shadow Level 1)
- Selected: 2px `--accent` ring, bg: `--accent-muted`
- **States:** default, hover, selected, disabled (pet not loaded yet)
- Animate selection change: ring appears with 150ms scale-in

### 4.7 Tab Bar

```
 [Controls]   Pets
 ━━━━━━━━━
```

- Active tab: `--fg` text, 2px `--accent` underline
- Inactive tab: `--muted` text, no underline
- Hover (inactive): `--fg` text
- Underline slides between tabs with 200ms ease-out (use `transform: translateX` on the indicator element)
- Tab labels: 14px, weight 500 (active) / 400 (inactive)

### 4.8 Step Indicator (import wizard)

```
 ● ──── ○ ──── ○
Upload  Review  Confirm
```

- Step dots: 28px circle, connected by 2px lines
- Completed: green fill + white checkmark
- Current: `--accent` fill, white number, subtle pulsing ring (2s infinite, opacity 1→0)
- Future: `--border` fill, `--muted` text
- Labels below dots: 12px, `--muted` (future) / `--fg` (current) / `--success` (completed)

### 4.9 Status Badge

```
 ✓ auto    ↻ alias    ✗ unmapped    ✎ manual
```

- Padding: 2px 8px, radius: 999px, size: 11px, weight: 500
- Variants: success (green bg 10% + green text), warning (amber), danger (red), info (accent)
- Mono icon prefix, 4px gap
- No hover state — badges are static indicators

### 4.10 Drop Zone

```
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
│                                  │
│         ┌──────┐                │
│         │  ↑   │                │
│         └──────┘                │
│    Drop files here or click     │
│    pet.json + spritesheet.webp  │
│                                  │
└ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

- Border: 2px dashed `--border`, radius: 12px, padding: 48px
- Hover (dragover): border solid `--accent`, bg: `--accent-muted`, icon scales to 1.1
- Upload icon: 32px, `--muted`
- Label: 14px `--muted`, secondary line: 12px `--muted` (60% opacity)
- Inside: hidden `<input type="file">` triggered by click

### 4.11 Toast / Notification

```
┌──────────────────────────────────┐
│  ✓  Pet imported successfully    │
└──────────────────────────────────┘
```

- Slides in from top-right, 300ms ease-out
- Auto-dismiss after 3s with fade-out
- Variants: success (green left border accent), error (red), info (accent)
- Height: 44px, padding: 12px 16px, radius: 8px, bg: `--surface`, Level 1 shadow
- Icon + message (14px) + optional action link ("Undo")

### 4.12 Skeleton Loader

- Grey rectangles matching the shape of the eventual content
- Pulse animation: opacity cycles 0.4 → 0.6 → 0.4, 1.5s duration
- Card skeletons: rectangle for thumbnail + 2 smaller rectangles for text lines
- Table skeletons: 4–6 rows of 80% width rectangles

### 4.13 Frame Counter (player)

```
Frame 3 / 9
```

- Mono font, 13px
- Current frame number is `--accent`, total is `--muted`
- Frame advance: number briefly scales up (1.1) and snaps back, 100ms
- Displayed in a compact pill: 2px 8px padding, `--border` bg, 999px radius

### 4.14 Select Dropdown

```
[ zhengke-youyu         ▾ ]
```

- Height: 36px, padding: 0 12px, radius: 8px
- Border: 1px `--border`, bg: `--surface`
- Open: border `--accent`, Level 1 shadow on dropdown panel
- Options: 36px height each, 12px padding, hover bg `--accent-muted`
- Selected option: `--accent` text, checkmark icon on right
- Transition: panel slides down 4px + fades in, 150ms

---

## 5. Interaction Rules

### 5.1 Motion Language

| Event | Duration | Easing | Effect |
|-------|----------|--------|--------|
| Toggle switch | 150ms | ease-out | Thumb slide + track color |
| Tab switch | 200ms | ease-out | Underline translateX |
| Step transition (import) | 250ms | ease-in-out | Fade + translateX(20px) |
| Card select | 150ms | ease-out | Ring scale-in + lift |
| Button press | 100ms | ease-out | Scale 0.97 |
| Toast enter | 300ms | ease-out | Slide down + fade in |
| Toast exit | 200ms | ease-in | Fade out |
| Dropdown open | 150ms | ease-out | Slide down 4px + fade |
| Modal open | 200ms | ease-out | Scale 0.95→1 + fade |
| Skeleton pulse | 1.5s | ease-in-out | Opacity 0.4↔0.6 (infinite) |
| Number flip (frame counter) | 100ms | ease-out | Scale 1→1.1→1 |
| Step dot pulse (current) | 2s | ease-out | Ring opacity 1→0 (infinite) |

### 5.2 Hover / Focus / Active States

- All interactive elements must have a **visible hover state** (color shift, border change, or subtle lift)
- All interactive elements must have a **visible focus state** for keyboard navigation: 2px `--accent` outline, 2px offset. Use `:focus-visible`, not `:focus`.
- **Active (press)** state: slight scale down (0.97) or darker color. Fast (100ms).
- **Disabled** state: 40% opacity across the entire element + children, `cursor: not-allowed`, no hover/focus effects.

### 5.3 Keyboard Navigation

- **Popup:** Tab through toggle → sliders → buttons in natural order. Enter/Space to activate.
- **Player:** Full keyboard support — Tab through all controls. Arrow keys adjust sliders (±0.05 step). Number keys 1–9 trigger corresponding role.
- **Import:** Tab through form fields. Enter to advance to next step. Escape to go back.
- **Focus trapping:** Inside modals, focus cycles within the modal.

### 5.4 Loading Strategy

- **Instant paint:** Every page shows its chrome (header, tabs, layout) immediately
- **Progressive load:** Content areas fill in as data arrives
- **Skeleton preference:** Use skeleton loaders (pulsing grey shapes matching layout) rather than spinners for content areas
- **Spinner use:** Only for actions in progress (button loading states, file upload analysis)

### 5.5 Micro-interactions (the "灵动" layer)

These are the small delightful moments that separate this product from generic devtools:

1. **Pet thumbnail hover:** On hover, the pet sprite in the selection grid does a tiny bounce (translateY -4px + ease-out-bounce)
2. **Toggle animation:** The pet icon next to the toggle shrinks slightly when disabled, pops back when enabled
3. **Slider snap:** When the slider value hits exactly 1×, a subtle haptic-like animation (thumb pulses once)
4. **Step completion:** When a step completes in the import wizard, the connecting line between dots fills with a colored sweep (left → right, 400ms)
5. **Role trigger glow:** When clicking a role trigger button in the player, the pet viewport border briefly flashes (200ms glow, `--accent`)
6. **Import success confetti-lite:** On successful import, 8–12 small colored dots burst from the confirm button and fade out (no full confetti library — just a few keyframe-animated divs)
7. **Pet name in header:** The pet name in the popup header has a subtle text-shadow glow in the accent color when the pet is "active"

---

## 6. Responsive Layout Rules

### 6.1 Popup

- Chrome controls the popup window size (typically ~400×520px, but variable)
- Layout uses `display: flex; flex-direction: column; height: 100vh` to fill available space
- Pet grid adapts columns: 3 columns at ≥380px, 2 columns at <380px
- Sliders and buttons are full-width
- Minimum usable width: 320px

### 6.2 Player

- **≥1024px (desktop):** Two-column layout. Pet viewport 60%, debug panels 40%. Sidebars scroll independently.
- **640–1023px (tablet):** Stacked layout. Pet viewport on top (40vh min-height), debug panels below in a 2-column grid.
- **<640px (phone):** Single column. Pet viewport 30vh. Debug panels stacked, each collapsible (accordion style). Role trigger buttons form a 3×3 grid.

### 6.3 Import

- **≥640px:** Centered card, max-width 720px, generous padding
- **<640px:** Full-width, padding 16px, step indicator compresses to dots only (no labels)
- Drop zone padding reduces proportionally
- ActionMap table stacks to card rows on mobile (label: value pairs)

### 6.4 Fluid Type

Use `clamp()` for key type sizes to avoid breakpoint-specific overrides:

```css
h1 { font-size: clamp(24px, 4vw, 32px); }
h2 { font-size: clamp(20px, 3vw, 24px); }
```

### 6.5 Dark Mode

- All components must work in both light and dark themes
- Use the `.dark` class on `<html>` or a container to switch
- System preference detected via `prefers-color-scheme`, with manual toggle override stored in `localStorage`
- Player background options (checkerboard, solid) are independent of the light/dark theme

---

## 7. Implementation Recommendations

### 7.1 Tech Stack

```
React 19 + TypeScript + Tailwind CSS v3 + shadcn/ui + Motion for React + lucide-react
```

Already defined in the PRD. The design system maps directly to Tailwind tokens:

| Design Token | Tailwind Config |
|-------------|-----------------|
| `--bg` | `colors.neutral.50` (light) / `colors.neutral.900` (dark) |
| `--surface` | `colors.white` / `colors.neutral.800` |
| `--fg` | `colors.neutral.900` / `colors.neutral.50` |
| `--muted` | `colors.neutral.500` / `colors.neutral.400` |
| `--border` | `colors.neutral.200` / `colors.neutral.700` |
| `--accent` | Custom rose/coral scale |

### 7.2 Component Mapping to shadcn/ui

| Design Component | shadcn/ui Base |
|-----------------|----------------|
| Toggle Switch | `Switch` |
| Slider | `Slider` |
| Primary/Secondary Button | `Button` (variant: default / outline / ghost) |
| Tab Bar | `Tabs` |
| Select Dropdown | `Select` (or custom `Command` for pet search) |
| Toast | `Sonner` (toast library) |
| Skeleton | `Skeleton` |
| Badge | `Badge` |
| Card | `Card` |
| Dialog/Modal | `Dialog` |
| Drop Zone | Custom (no shadcn primitive — use react-dropzone or native) |

### 7.3 Motion Strategy

- **UI transitions:** Use Motion for React (`motion.div`) for:
  - Tab underline sliding
  - Step transitions in import
  - Toast enter/exit
  - Dropdown open/close
  - Panel expand/collapse
- **Sprite animation:** Handled by `SpritePlayer` (CSS background-position cycling). Motion library not used for sprite playback — too much overhead for 60fps frame animation.
- **Micro-interactions:** Pure CSS keyframe animations + Tailwind `animate-*` utilities where possible. Reserve Motion for layout changes and enter/exit transitions.

### 7.4 CSS Architecture

```
globals.css
  ├─ @tailwind base/components/utilities
  ├─ :root { color tokens (OKLch) }
  ├─ .dark { dark color tokens }
  ├─ @layer base { typography defaults, focus styles }
  └─ @layer components { skeleton pulse, toast, step-indicator }
```

Per-page styles live in their respective component files via Tailwind utility classes. No per-page `.css` files.

### 7.5 Accessibility Baseline

- All interactive elements: focus-visible outline (2px, `--accent`, 2px offset)
- Color contrast: minimum 4.5:1 for body text, 3:1 for large text (≥24px)
- Form inputs: associated `<label>` elements
- Toggle: `role="switch"` + `aria-checked`
- Slider: `role="slider"` + `aria-valuenow/aria-valuemin/aria-valuemax`
- Tab list: `role="tablist"` + `role="tab"` + `aria-selected`
- Toast: `role="status"` + `aria-live="polite"`
- Step indicator: `role="progressbar"`-like semantics via `aria-valuenow`

### 7.6 Performance Notes

- Pet sprite thumbnails in the popup grid: lazy-load via IntersectionObserver, 72×72 @1x (144×144 @2x max)
- Player spritesheet display: the preview uses the same object URL as the content script injection — no duplicate blob loading
- Import detection: run spritesheet analysis in a `requestIdleCallback` or chunked async to avoid blocking the UI
- Motion: prefer `transform` and `opacity` animations only (GPU-composited). Never animate `width`, `height`, `top`, `left`.

---

## 8. Key State Matrix

| Page | Loading | Empty | Error | Success | Disabled |
|------|---------|-------|-------|---------|----------|
| Popup (Controls) | — | — | — | — | 60% opacity overlay when disabled |
| Popup (Pets) | 3 skeleton cards | "No pets" CTA | Red banner + retry | — | Cards greyed, unclickable |
| Player (viewport) | Pulsing border skeleton | "Select a pet" placeholder | "Failed to load pet" | — | — |
| Player (panels) | Skeleton rows | Greyed panels | — | — | — |
| Import (Step 1) | — | — | Red border on drop zone + error text | File chips shown | Next button disabled until files valid |
| Import (Step 2) | Spinning "Analyzing…" state | "No actions found" | "Preset detection failed" | ActionMap populated | Next disabled if unmapped roles exist |
| Import (Step 3) | — | — | — | Green success toast | Import button disabled while saving |
