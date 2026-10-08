---
name: paper-ui-ux
description: >-
  Master styling, theming, and UI/UX design system guide for Lenga Store's skeuomorphic "Paper and Ink" tactile aesthetic.
  Use whenever creating, modifying, or reviewing pages, components, animations, or styling in this codebase to ensure
  complete visual consistency. Covers the wooden desk environment, hardcover book 3D illusion, sticky note categories,
  ruled exercise notebook pages, library checkout slips, torn paper edges, e-reader desk with spine shadows, and ledger stamps.
metadata:
  author: lenga
  version: "1.0.0"
---

# Lenga Store: Paper & Ink UI/UX Design System

Lenga Store uses a distinctive **tactile, skeuomorphic "Paper & Ink"** design system inspired by school exercise books, cloth-bound hardcover library books, handwritten Post-it notes with tape, and aged paper on a wooden study desk.

The primary goal of this skill is to ensure that all agents and developers maintain visual fidelity, tactile warmth, and physical craft across every screen.

---

## 1. Design Philosophy & Foundations

1. **Physicality & Tactility**: Elements represent tangible desk items — paper sheets, cardboard parcels, library lending slips, index cards, notebooks, and clothbound books.
2. **Warmth Over Sterile SaaS**: Avoid generic dark mode, glassmorphism-heavy tech templates, or cold monochrome minimalism. Use warm cream tones (`#FBF6E9`), subtle paper fiber grain, and fountain pen navy (`#1E2B4A`).
3. **Intentional Imperfection**: Elements feature slight analog rotations (`-rotate-1`, `rotate-0.5`, `rotate-2`), wobbly hand underlines (`.scribble`), and tape strips.
4. **Literary Typography**: Classical serifs for headlines and reading prose; casual handwriting for marginalia and stamps.

---

## 2. Color Palette & Theme Tokens

Defined in `app/globals.css` via Tailwind v4 `@theme`:

| Token | Hex Value | Semantic Purpose |
|---|---|---|
| `--color-paper` | `#FBF6E9` | Primary exercise book paper & sheet surfaces |
| `--color-paper-deep` | `#F3EBD6` | Aged parchment, board backing, cardstock |
| `--color-ink` | `#1E2B4A` | Primary fountain-pen navy ink (body text, dark buttons, board) |
| `--color-ink-soft` | `#4A5775` | Secondary text, subtitles, pencil marginalia |
| `--color-pen` | `#C23B4A` | Accent red (teacher's correction pen, price tags, stamps, seals) |
| `--color-rule` | `#BFD6E4` | Faint cyan/blue notebook rule lines |
| `--color-margin` | `#E4868A` | Exercise book vertical red margin line |
| `--color-kraft` | `#C8A87C` | Brown parcel paper, cardboard headers, packing slips |
| `--color-sticky-butter` | `#FFE680` | Classic yellow Post-it note |
| `--color-sticky-rose` | `#FFC2CF` | Pink Post-it note |
| `--color-sticky-mint` | `#BDEBCB` | Pale green Post-it note |
| `--color-sticky-sky` | `#BFE0FA` | Soft blue Post-it note |

### Shadows
- `--shadow-sheet`: `0 1px 0 #E6DCC0, 0 8px 18px -8px rgba(60, 45, 20, 0.35)` (Layered desk paper)
- `--shadow-sticky`: `0 1px 1px rgba(60, 45, 20, 0.25), 6px 10px 14px -8px rgba(60, 45, 20, 0.45)` (Lifted corner drop shadow)

---

## 3. Typography & Rhythms

Configured via Google Fonts in `app/layout.tsx`:

```tsx
const display = Young_Serif({ weight: "400", subsets: ["latin"], variable: "--font-display" });
const body = Newsreader({ subsets: ["latin"], variable: "--font-body" });
const hand = Caveat({ subsets: ["latin"], variable: "--font-hand" });
```

### Font Roles
- **`font-display` (`Young Serif`)**: Headings (`h1`, `h2`, `h3`), book titles, price tags, rubber stamp labels, section tags.
- **`font-body` (`Newsreader`)**: Book prose, descriptions, summaries, long-form editorial copy.
- **`font-hand` (`Caveat`)**: Sticky note annotations, taped notes, casual author notes, bookstore signs.

### Vertical Rhythm
- `--line: 32px`: The height of ruled notebook lines.
- Headings and paragraphs on notebook pages use line heights that sit directly on the rules: `leading-[32px]`, `leading-[48px]`, `leading-[64px]`.
- `.scribble`: `text-decoration: underline wavy var(--color-pen); text-underline-offset: 6px;`

---

## 4. Textures & Backgrounds

### 4.1 Paper Fibre Noise (`.grain`)
Generates realistic microscopic paper fibers using an inline SVG turbulence filter without downloading image assets:

```css
.grain::after {
  content: "";
  position: absolute;
  inset: 0;
  pointer-events: none;
  mix-blend-mode: multiply;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='220' height='220'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='.85' numOctaves='3' stitchTiles='stitch'/><feColorMatrix values='0 0 0 0 .45 0 0 0 0 .33 0 0 0 0 .2 0 0 0 .9 -.35'/></filter><rect width='100%' height='100%' filter='url(%23n)'/></svg>");
}
```

### 4.2 The Study Desk (`.store-bg`)
Light wooden study desk with fiber grain plus a faint radial dot grid:
```css
.store-bg {
  position: relative;
  isolation: isolate;
  background-color: #F1E9D3;
  background-image: radial-gradient(rgba(30, 43, 74, 0.09) 1px, transparent 1.3px);
  background-size: 24px 24px;
}
```

---

## 5. Signature Components & Patterns

### 5.1 Hardcover Book Component (`<Cover />` & `.book`)
A realistic 3D hardcover seen from the front with physical depth, boards, page block, and spine crease:

```tsx
import { Cover } from "@/components/store";

<Cover src={book.cover} title={book.title} priority sizes="320px" />
```

**Architecture (back to front):**
1. **Back Board (`.book-back`)**: Dark board (`#3B2F24`) peeking out along the right edge.
2. **Page Block (`.book-pages`)**: Cream sheets seen edge-on with fine vertical gradient lines.
3. **Front Board (`.book-front`)**: Slightly inset (`--board: 3.5%; --drop: 1.6%`) with cover art and a bevel highlight.
4. **Spine Hinge (`.book-hinge`)**: Linear gradient creating the vertical groove where the hardcover flexes.
5. **Surface Sheen (`.book-sheen`)**: Diagonal light reflection and print texture.
6. **Shelf Interaction**: On `.group:hover .book`, the book tilts and lifts up off the wooden desk:
   `transform: translateY(-4px) rotate(-1deg);` with dual drop-shadows.

### 5.2 Category Sticky Notes (`<StickyNote />`)
Post-it notes pinned to corkboards or desk shelves with translucent scotch tape:

```tsx
import { StickyNote } from "@/components/paper";

<StickyNote tone="butter" tilt={-2} tape={true}>
  <h3 className="text-2xl leading-8">Fiction</h3>
  <p className="text-base leading-6 text-ink-soft">Novels and short stories.</p>
</StickyNote>
```

**Features:**
- Tones: `"butter"`, `"rose"`, `"mint"`, `"sky"`.
- Variable tilts (`--tilt: -2deg` to `2deg`).
- Translucent tape strip at the top with `backdrop-blur-[1px]`.
- Curled bottom-right corner fold (`bg-[linear-gradient(135deg,transparent_50%,rgba(0,0,0,.12)_50%)]`).
- Hover animation: Straightens and lifts (`hover:rotate-0 hover:-translate-y-1`).

### 5.3 Exercise Notebook Page (`<NotebookPage />`)
School exercise book page with blue rules and red vertical margin:

```tsx
import { NotebookPage } from "@/components/paper";

<NotebookPage className="pb-16">
  <h1 className="text-5xl leading-[64px]">Books for every reader.</h1>
  <p>Novels, non-fiction, children's books and textbooks.</p>
</NotebookPage>
```

**Features:**
- `.ruled`: Blue lines every 32px (`calc(var(--line) - 1px)`).
- `.margin-line`: Red double rule at `4.25rem` (68px).
- Internal padding: `pl-24 pr-8 pt-8` on desktop. On narrow mobile viewports, use `!pl-[4.75rem] sm:!pl-24 !pr-4 sm:!pr-8` so text aligns past the margin line without overflowing.

### 5.4 Library Slip (`<Slip />`)
A library checkout card with dashed edges used for purchase summaries, pricing plans, and book specs:

```tsx
import { Slip } from "@/components/paper";

<Slip className="mt-8 rotate-1 max-w-full">
  <p className="flex items-baseline justify-between">
    <PriceTag price={book.price} className="text-4xl leading-[48px]" />
  </p>
  <BookPurchaseAction book={book} />
</Slip>
```

**Features:**
- `border-2 border-dashed border-ink/40`
- `bg-paper shadow-sheet`
- Slight tilt: `rotate-1` or `-rotate-0.5`.

### 5.5 Torn Edge & Torn Section (`<TornEdge />`, `<TornSection />`)
Cardboard / kraft bands with algorithmic jagged tear paths:

```tsx
import { TornSection } from "@/components/paper";

<TornSection tone="board" wide>
  <h2>Browse by genre</h2>
  ...
</TornSection>
```

**Tones:**
- `kraft`: Brown cardboard (`bg-kraft text-ink`).
- `board`: Cork pinboard (`bg-[#E6D6B2] text-ink`).
- `ink`: Deep navy banner (`bg-ink text-paper`).

### 5.6 Buttons & CTAs
Tangible paper cards with tactile press feedback:

```tsx
// Primary Ink Button
export const primaryBtn =
  "inline-block bg-ink px-6 py-2 font-display text-paper shadow-sticky transition-transform hover:rotate-0 hover:bg-pen active:translate-y-0.5 cursor-pointer";

// Secondary Paper Button
export const paperBtn =
  "inline-block bg-paper px-6 py-2 font-display text-ink shadow-sticky transition-transform hover:rotate-0 hover:bg-paper-deep active:translate-y-0.5 cursor-pointer";
```

### 5.7 Verified Reader Reviews Ledger
Archival ledger sheet with red ink stamps and reader marginalia:
- Section header wrapped in parchment container with faux paper tape.
- Average rating stamped in teacher's red ink seal (`border-2 border-pen/40 rotate-1 bg-paper-deep/40`).
- Star ratings using ★ and ☆ in font-display serif.
- Verified reader badge in green ledger stamp style (`bg-green-200/70 border border-green-700/20`).

---

## 6. The E-Reader Desk Experience (`/read/[slug]`)

The reader provides a physical distraction-free reading desk:

1. **3D Stage (`.reader-stage`)**:
   - `perspective: 2000px` for realistic paper page depth.
   - Supports single-page or two-page spread (`data-layout="double"`).
2. **Sheet Surface (`.reader-sheet`)**:
   - Real paper grain overlay (`.reader-sheet::after`).
   - Running header (`.reader-sheet-head`) and footer folio (`.reader-sheet-foot`).
   - Center binding shadow (`.reader-spine`) in 2-page mode; left spine crease (`.is-single::before`) in 1-page mode.
3. **Themes**:
   - **Day**: Clean cream sheet (`#FBF6E9`) on study desk (`#EFE7D2`).
   - **Lamp**: Warm reading lamp glow (`#F1E2C2` on `#DFCEA5`).
   - **Night**: Dark study desk (`#162038` on `#0C1220`) with pale gold ink (`#E4DAC2`).
4. **Floating Chrome**:
   - Translucent glassmorphism pill bars (`backdrop-blur-[10px]`) that smoothly hide when reading.

---

## 7. Responsiveness & Overflow Guidelines

When building or updating pages:
1. **Never let `white-space: nowrap` or `truncate` blow out CSS Grid columns**:
   Always add `min-w-0 max-w-full overflow-hidden` to grid items containing text.
2. **Always Portal Modals**:
   Modals rendered inside transformed ancestors (e.g. `<Slip className="rotate-1">`) will become trapped inside the parent box due to CSS `transform` containing block rules. Always use `createPortal(modal, document.body)`.
3. **Respect the Red Margin on Mobile**:
   On screen widths < 640px, adjust notebook padding to `!pl-[4.75rem]` to give ~8px clearance past the red margin line at `4.25rem`.
4. **Clip Parent Bounds When Using Rotation**:
   Add `overflow-x-clip` on main page wrappers when child cards or buttons use `-rotate-1` or `rotate-1` to prevent horizontal scrollbars on mobile.
