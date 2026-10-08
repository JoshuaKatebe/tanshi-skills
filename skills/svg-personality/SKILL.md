---
name: svg-personality
description: Use SVG to give websites and web apps real visual personality instead of the default rounded-rectangles-and-gradients look. Use this skill whenever building, restyling, or polishing a landing page, marketing site, store, portfolio, dashboard, or app UI where character, brand feel, illustration, texture, icons, dividers, loaders, empty states, 404 pages, hover effects, or animation matter, and whenever the user says a design looks generic, bland, flat, templated, "AI-looking", or "all the same". Covers SVG filters as materials (paper, grain, ink, emboss), hand-drawn and wobbly lines, custom icons and marks, patterns and hatching, clip paths and masks for non-rectangular shapes, text on a path, path-draw and scroll animation, and the performance, accessibility and theming rules that keep it shippable. Use it alongside any frontend design skill, even if the user never says "SVG".
---

# SVG for Personality

Most websites look alike because they are built from the same few parts: rectangles with rounded corners, a soft gradient, an icon-library icon, a stock illustration. Those parts are fast to produce and say nothing about who made the site.

SVG is the cheapest way out. It gives you shapes that are not boxes, textures with no image files, lines that look drawn by a person, and motion that means something. It scales to any screen, themes with CSS variables, and weighs a few hundred bytes.

This skill is about using it with intent. Decoration is not the goal; the goal is that someone could recognise the site with the logo covered.

## The one rule

**Every SVG element should say something about this product or brand.** Test each one: could it appear unchanged on a competitor's site? If yes, redo it or delete it. A floating gradient blob fails this test. A wobbly underline on the one word the brand cares about passes.

## Workflow

### 1. Find where the site is generic

Look at the page (or the brief) and list the places that are default: hero background, section edges, icons, buttons, cards, borders, loading and empty states, the 404, the footer. Those are your candidates.

### 2. Write a personality brief (three to five lines)

Answer these before drawing anything. They stop you from decorating at random.

- **Material:** what is this brand made of? Paper, enamel, neon, chalk, felt, brushed metal, circuit board, tracing paper, tin toys?
- **Hand:** how steady is it? Machine-precise, relaxed, shaky, hurried, playful?
- **Motion verb:** how do things move? Snap, stamp, drift, unfold, bounce, scribble, slide?
- **Signature shape:** one recurring form (arch, circle, diagonal slash, notch, starburst)?
- **Palette:** three to five colours plus ink and background, as CSS variables.

Some examples of how different answers produce different sites:

| Site | Material and hand | SVG moves |
|---|---|---|
| Bakery | warm, hand-kneaded, slightly uneven | wobble filter on borders, flour-dust grain, hand-drawn underline on prices, steam wisps that drift |
| Developer tool | precise, blueprint | hairline grid pattern, dimension lines and callout arrows, diagrams that draw themselves on scroll |
| Kids' learning app | bouncy, gooey | goo filter on buttons, squishy blobs that respond to hover, characters built from circles |
| Record label | printed, misregistered | offset colour fills behind outlines, halftone patterns, stamp-style rotated text badges |
| Law firm or bank | calm, engraved | guilloche line patterns, fine hatching for depth, restrained animated rules |

### 3. Pick three to seven "personality moments"

Spend personality where users pause, not everywhere. Rank by how often people see it and how long they look:

1. Hero and logo mark
2. Section transitions and dividers
3. Emphasis on key words (underlines, circles, highlights, arrows)
4. Icons for primary features and navigation
5. Loading, empty, error, success and 404 states (the most neglected, so the biggest payoff)
6. Hover and press states on the primary button
7. Footer and sign-off
8. Backgrounds and surface textures

Leave the rest restrained. A site where everything is decorated has no focal point and reads as noisy, which is its own kind of generic.

### 4. Build the vocabulary first

Before placing anything, define the shared pieces once: filters, patterns, gradients, a symbol sprite, and CSS tokens such as `--stroke`, `--wobble`, `--ink`, `--accent`. Consistency of stroke width, corner style and wobble amount is what makes separate drawings feel like one hand. Mixed styles are the fastest way to look amateur.

### 5. Implement, then look at it

You cannot judge SVG from the code. Render it and view it. See "Verifying your work" below.

### 6. Cut

Remove the weakest third of what you added. Then do the performance and accessibility pass.

## Toolbox

### Filters as materials

SVG filters turn flat vector shapes into materials. `feTurbulence` generates noise, and what you do with the noise decides the material. One filter family with different frequencies can produce paper fibre, wood grain, cloth weave and stone.

**Wobbly, hand-drawn edges.** Displace a shape by noise. Keep `scale` small (2 to 5); large values look broken. Apply it to decorative layers (borders, shapes, pseudo-elements), not to body text, which gets fuzzy.

```html
<svg width="0" height="0" aria-hidden="true" style="position:absolute">
  <filter id="wobble" x="-5%" y="-5%" width="110%" height="110%">
    <feTurbulence type="fractalNoise" baseFrequency="0.02" numOctaves="2" seed="3" result="n"/>
    <feDisplacementMap in="SourceGraphic" in2="n" scale="4" xChannelSelector="R" yChannelSelector="G"/>
  </filter>
</svg>
<style>.hand { filter: url(#wobble); }</style>
```

**Line boil.** Real hand-drawn animation never holds perfectly still. Define three copies of the wobble filter with different `seed` values (`wobble1`, `wobble2`, `wobble3`) and step between them at about 8 fps. It is very cheap and instantly gives a drawn feel.

```css
.boil { animation: boil .375s steps(1) infinite; }
@keyframes boil {
  0%   { filter: url(#wobble1); }
  33%  { filter: url(#wobble2); }
  66%  { filter: url(#wobble3); }
}
@media (prefers-reduced-motion: reduce) { .boil { animation: none; filter: url(#wobble1); } }
```

**Grain and texture as a CSS background, no image file.** Encode `#` as `%23` (or run the string through `encodeURIComponent`). Keep tiles small and use `stitchTiles="stitch"` so they repeat without seams. `mix-blend-mode: multiply` makes the grain sit in the surface instead of on top of it.

```css
.grain { position: relative; }
.grain::after {
  content: ""; position: absolute; inset: 0; pointer-events: none;
  mix-blend-mode: multiply; opacity: .5;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='200' height='200'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='.8' numOctaves='3' stitchTiles='stitch'/><feColorMatrix values='0 0 0 0 .3  0 0 0 0 .2  0 0 0 0 .1  0 0 0 .9 -.3'/></filter><rect width='100%' height='100%' filter='url(%23n)'/></svg>");
}
```

Change `baseFrequency` to change the material: an elongated value like `.01 .3` streaks into wood grain; `.8` is paper fibre; `.05` is soft stone or cloud. The `feColorMatrix` rows choose the tint (first three rows) and how dense the speckle is (last row).

If the same texture is needed in a native app (Flutter, SwiftUI) or an email, render the filter once to a PNG tile at 1x, 2x and 3x and repeat it there, because those platforms do not run SVG filters.

**Emboss, deboss and bevel.** Lighting filters give foil, stamped and raised-letter looks. They are costly, so use them on small elements only.

```html
<filter id="deboss">
  <feGaussianBlur in="SourceAlpha" stdDeviation="1" result="b"/>
  <feSpecularLighting in="b" surfaceScale="3" specularConstant=".8" specularExponent="20" lighting-color="#fff" result="s">
    <feDistantLight azimuth="225" elevation="50"/>
  </feSpecularLighting>
  <feComposite in="s" in2="SourceAlpha" operator="in" result="s2"/>
  <feComposite in="SourceGraphic" in2="s2" operator="arithmetic" k1="0" k2="1" k3="1" k4="0"/>
</filter>
```

**Gooey merging.** Blur, then crush the alpha channel, and nearby shapes fuse like liquid. Good for playful toggles, menus and loaders.

```html
<filter id="goo">
  <feGaussianBlur in="SourceGraphic" stdDeviation="8" result="b"/>
  <feColorMatrix in="b" values="1 0 0 0 0  0 1 0 0 0  0 0 1 0 0  0 0 0 18 -7" result="g"/>
  <feComposite in="SourceGraphic" in2="g" operator="atop"/>
</filter>
```

Other useful ones: `feComponentTransfer` for duotone and posterise, `feMorphology` to thicken text into a sticker outline, `feDropShadow` for hard offset "print" shadows with zero blur.

### Hand-drawn emphasis

This is the highest-return trick on the list. Underline, circle, or arrow the one thing that matters, and animate it drawing on. `pathLength="1"` normalises the path so the dash maths is always 0 to 1 whatever the real length.

```html
<span class="mark">
  <span>handmade</span>
  <svg viewBox="0 0 100 12" aria-hidden="true" focusable="false">
    <path d="M2 8 C 20 2, 40 11, 60 6 S 90 4, 98 7" pathLength="1"/>
  </svg>
</span>
<style>
.mark { position: relative; display: inline-block; }
.mark svg { position: absolute; left: -2%; bottom: -.3em; width: 104%; height: auto; overflow: visible;
  fill: none; stroke: var(--accent); stroke-width: 3; stroke-linecap: round; }
.mark path { stroke-dasharray: 1; stroke-dashoffset: 1; animation: draw .8s .2s ease-out forwards; }
@keyframes draw { to { stroke-dashoffset: 0; } }
@media (prefers-reduced-motion: reduce) { .mark path { animation: none; stroke-dashoffset: 0; } }
</style>
```

Do not add `vector-effect: non-scaling-stroke` to a path you are dash-animating; it makes the dash lengths use screen units and breaks `pathLength` in several browsers.

### Generating wobbly paths in code

When you need many hand-drawn lines (borders, tally marks, scribbled fills), generate them. Seed the randomness from the content (a hash of the text, the item id) so each one is unique but identical on server and client. Unseeded `Math.random()` causes hydration mismatches and shapes that change on every reload.

```js
const rng = (a) => () => { a |= 0; a = a + 0x6D2B79F5 | 0; let t = Math.imul(a ^ a >>> 15, 1 | a);
  t = t + Math.imul(t ^ t >>> 7, 61 | t) ^ t; return ((t ^ t >>> 14) >>> 0) / 4294967296; };

// Hand-drawn line: jittered points smoothed with Catmull-Rom into cubic Béziers.
function wobblyLine(x1, y1, x2, y2, { seed = 1, amp = 2, steps = 8 } = {}) {
  const r = rng(seed), pts = [];
  for (let i = 0; i <= steps; i++) {
    const t = i / steps;
    pts.push([x1 + (x2 - x1) * t + (r() - .5) * amp, y1 + (y2 - y1) * t + (r() - .5) * amp]);
  }
  const f = (n) => n.toFixed(1);
  let d = `M${f(pts[0][0])},${f(pts[0][1])}`;
  for (let i = 0; i < pts.length - 1; i++) {
    const p0 = pts[i - 1] || pts[i], p1 = pts[i], p2 = pts[i + 1], p3 = pts[i + 2] || p2;
    d += ` C${f(p1[0] + (p2[0] - p0[0]) / 6)},${f(p1[1] + (p2[1] - p0[1]) / 6)}` +
         ` ${f(p2[0] - (p3[0] - p1[0]) / 6)},${f(p2[1] - (p3[1] - p1[1]) / 6)}` +
         ` ${f(p2[0])},${f(p2[1])}`;
  }
  return d;
}
```

### Shapes that are not boxes

**Section edges.** Pin a small SVG to the bottom of a section and fill it with the next section's colour. It scales with width and the edge stays crisp. Vary the curve per section; one repeated wave on every section is a cliché.

```html
<section class="hero">
  …
  <svg class="edge" viewBox="0 0 1200 40" preserveAspectRatio="none" aria-hidden="true" focusable="false">
    <path d="M0 24 C 200 4, 400 40, 600 20 S 1000 4, 1200 26 V40 H0 Z" fill="var(--next-bg)"/>
  </svg>
</section>
<style>
.hero { position: relative; }
.edge { position: absolute; left: 0; right: 0; bottom: -1px; width: 100%; height: 40px; display: block; }
</style>
```

**Clipping and masking.** `clipPathUnits="objectBoundingBox"` makes a clip path scale with the element (coordinates 0 to 1), unlike CSS `clip-path: path()` which has fixed pixel coordinates. Use it for photo frames, torn edges and cut corners. Use `<mask>` with a gradient for soft fades and reveal effects.

**Patterns.** `<pattern>` gives dots, grids, hatching and halftone-style fills. Hatching instead of a blurred drop shadow gives depth a distinctly printed or engraved look.

```html
<pattern id="hatch" width="6" height="6" patternUnits="userSpaceOnUse" patternTransform="rotate(45)">
  <line x1="0" y1="0" x2="0" y2="6" stroke="currentColor" stroke-width="1"/>
</pattern>
```

### Icons and marks with a point of view

Icon libraries are fine as a base, but a whole interface of untouched library icons is a sign of a template. Redraw or modify the five to ten that matter most, and keep the set consistent: one stroke width, one cap style, one corner treatment.

Ideas that cost almost nothing:

- Offset a fill shape 1 to 2px from the outline for a misregistered print feel.
- Use uneven terminals and slightly overshooting strokes instead of perfect geometry.
- Give the icon one accent colour element (the spark, the dot).
- Build icons from the brand's signature shape.
- Use `currentColor` for strokes so they follow text colour, and a CSS variable for the accent.

```html
<svg class="icon" viewBox="0 0 24 24" aria-hidden="true" focusable="false">
  <polygon class="fill" points="4,11 12,4 20,11 20,20 4,20" transform="translate(1.5 1.5)"/>
  <polyline class="line" points="4,11 12,4 20,11 20,20 4,20 4,11"/>
</svg>
<style>
.icon { width: 1.5em; height: 1.5em; overflow: visible; }
.icon .fill { fill: var(--accent); }
.icon .line { fill: none; stroke: currentColor; stroke-width: 1.75; stroke-linecap: round; stroke-linejoin: round; }
.icon:hover .line { transform: rotate(-4deg); transform-box: fill-box; transform-origin: center; transition: transform .2s; }
</style>
```

### Text on a path

Seals, stamps, badges and curved headlines. `textLength` set to the circumference (2πr) makes the text fill the ring exactly.

```html
<svg viewBox="0 0 200 200" class="badge" role="img" aria-label="Fresh daily">
  <defs><path id="ring" d="M100,100 m-75,0 a75,75 0 1,1 150,0 a75,75 0 1,1 -150,0"/></defs>
  <text font-size="16" letter-spacing="2" fill="currentColor">
    <textPath href="#ring" textLength="471" lengthAdjust="spacing">FRESH DAILY • FRESH DAILY • FRESH DAILY •</textPath>
  </text>
</svg>
<style>.badge { animation: spin 24s linear infinite; } @keyframes spin { to { transform: rotate(360deg); } }
@media (prefers-reduced-motion: reduce) { .badge { animation: none; } }</style>
```

`paint-order: stroke` with a thick stroke gives sticker-outlined text without extra markup.

### Motion that carries meaning

- **Draw-on** (shown above) for diagrams, routes, signatures, underlines.
- **Scroll-driven draw:** tie the same keyframes to scroll position. Wrap in `@supports` so unsupporting browsers show the finished drawing instead of an invisible one.

```css
@supports (animation-timeline: view()) {
  .route path { stroke-dasharray: 1; stroke-dashoffset: 1; animation: draw linear both;
    animation-timeline: view(); animation-range: entry 10% cover 55%; }
}
```

- **Follow a path:** `offset-path: path("M…")` with an animated `offset-distance` moves any HTML element along a route (a bee, a paper plane, a delivery pin).
- **Morphing:** paths with the same number and type of commands can interpolate (`d: path()` in Chromium, or a small library). Good for toggles and mood changes.
- **Character:** eyes that follow the cursor, a mascot that reacts to form errors, a loader that tells a tiny story. Small narrative beats beat spinners.

Animate `transform` and `opacity` when you can. Animating filter parameters on large areas is expensive.

### Charts and data with a voice

Charts are SVG already, so give them the brand's hand: the same wobble, hatching instead of flat fills, annotation arrows drawn in the accent style, direct labels instead of legends. Data pages are some of the blandest on the web, so this is where it stands out.

### Theming

Inline SVG inherits CSS variables and `currentColor`, so illustrations can follow dark mode and brand themes without separate files. SVG used as an `<img>` or CSS background cannot see page CSS; if it needs dark mode, put a `@media (prefers-color-scheme: dark)` block inside the SVG itself.

## Gotchas that waste hours

- **Hidden defs:** do not hide the SVG that holds your filters or gradients with `display:none`; some browsers then fail to resolve them. Use `width="0" height="0"` with `position:absolute` and `aria-hidden="true"`.
- **Duplicate ids:** if you inline the same SVG several times, ids for gradients, filters and clip paths collide and the first one wins. Define shared things once globally, or generate unique ids per instance (React `useId`).
- **Transforms on SVG children:** `transform-origin` defaults to the SVG's top-left. Add `transform-box: fill-box; transform-origin: center;` for rotate and scale around the shape's own centre.
- **Stroke scaling:** strokes scale with the drawing. Use `vector-effect: non-scaling-stroke` for constant-width lines, except on dash-animated paths (see above).
- **Filter regions:** filters clip to a default region (-10% to 120%). Widen `x`, `y`, `width`, `height` on the filter if edges get cut off, especially with displacement or blur.
- **Safari:** filters on `foreignObject` are not supported. Test `mix-blend-mode` with filtered layers.
- **Optimisers:** SVGO defaults can strip `viewBox`, rename or remove ids you reference, and remove `<title>`. Configure it to keep them.
- **Sizing:** set `viewBox` always, size with CSS, and use `preserveAspectRatio="none"` only for deliberately stretchy things like edges and rules.

## Performance

- Prefer inline SVG for things you style or animate, `<img>` or CSS background for static art that never changes, and a sprite (`<symbol>` + `<use>`) for repeated icons.
- `feTurbulence` is computed on the CPU and is the heaviest common filter. Fine for a tile or a small element. Avoid it on full-screen animated layers; render a static texture once instead.
- Filter cost scales with the filtered area, so apply it to the smallest element possible and set a tight filter region.
- Do not animate `baseFrequency`, `stdDeviation` or lighting parameters continuously. Step between a few prepared variants instead (see line boil).
- Keep path data lean: round to one decimal place and simplify traced paths.

## Accessibility

- Decorative SVG: `aria-hidden="true" focusable="false"` and `pointer-events: none` if it overlaps content.
- Meaningful SVG (a logo, a chart, an illustration that carries information): `role="img"` with `<title>` and `aria-labelledby`, or an `aria-label`. Provide a text alternative for charts nearby.
- Never put essential text only inside SVG paths or textures. Real text stays real text.
- Keep text contrast on top of textures and illustrations. Check it with the texture on.
- Respect `prefers-reduced-motion` for every animation, and make the end state the visible default (not an invisible start state).
- Nothing should flash faster than three times a second.

## Verifying your work

Reading SVG code does not tell you whether it looks good. Render it and look.

1. Build or open the page in a real browser (headless Chromium or Playwright screenshots work). Filters like `feTurbulence` and lighting are only reliably rendered by browsers; some command-line rasterisers ignore most filters, so a blank or flat result there does not mean your SVG is wrong.
2. View the screenshot at desktop and phone widths, light and dark, and with reduced motion on.
3. Zoom into details: stroke ends, seams where tiles repeat, edges clipped by filter regions.
4. Ask the brand question again for each element: does it say something about this product?

For standalone drawings (a mascot, a mark), build from simple primitives on a grid, keep to the palette, prefer abstract or object-based illustration over detailed figures and faces, and iterate on the rendered image rather than the code.

## Anti-patterns

- **The blob:** soft gradient circles floating behind the hero. It is the default look of generated sites.
- **Wave on everything:** the same wavy divider between every section.
- **Stock flat people:** the purple-skinned, no-face illustration style used by thousands of sites.
- **Library icons untouched** across the whole interface.
- **SVG only as a logo file.**
- **Decoration without a reason:** random shapes, stars and squiggles added for "fun" that relate to nothing.
- **Unseeded randomness**, which changes the art on each load.
- **Mixed styles:** three stroke widths, two wobble amounts, and a perfectly geometric icon beside a hand-drawn one.
- **Texture that fights the text:** if people squint, lower the opacity.

## Before you finish

- [ ] A brief exists (material, hand, motion verb, signature shape, palette) and the work follows it
- [ ] Three to seven moments, not twenty
- [ ] Stroke width, corners and wobble are consistent across everything
- [ ] Every element passes the "could be on a competitor's site" test
- [ ] Looked at in a real browser: desktop, phone, light and dark
- [ ] Reduced motion handled; decorative SVG hidden from assistive tech; meaningful SVG labelled
- [ ] Text contrast checked with textures on
- [ ] No duplicate ids, `viewBox` on every SVG, filters scoped to small areas
- [ ] Random shapes are seeded
