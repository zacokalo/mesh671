# Mesh671 Design System

Single-file static site (`index.html`). No build step. All theming lives in CSS custom properties defined on `:root` and overridden by `[data-theme]` attribute + `@media (prefers-color-scheme)`.

---

## Color palette

CSS variables declared on `:root` (dark default) and overridden in `[data-theme="light"]` / `@media (prefers-color-scheme:light)`:

| Token     | Dark              | Light             | Role                              |
|-----------|-------------------|-------------------|-----------------------------------|
| `--rock`  | `#111614`         | `#F0EBE0`         | Page / card background            |
| `--rock2` | `#161C1A`         | `#E8E0D0`         | Alternate section background      |
| `--rock3` | `#1C2420`         | `#DDD4C0`         | Hover / raised surface            |
| `--reef`  | `#2DC4A8`         | `#1A7A68`         | Primary accent (teal)             |
| `--reef2` | `#1FA090`         | `#145A50`         | Reef hover / deeper teal          |
| `--sand`  | `#C4A96A`         | `#8A6820`         | Secondary accent (gold)           |
| `--sand2` | `#A88C50`         | `#6A5010`         | Sand hover / deep gold            |
| `--mist`  | `#5A7068`         | `#7A6A50`         | Muted / subtext                   |
| `--text`  | `#E0EDE8`         | `#1C2018`         | Body copy                         |
| `--textd` | `#A8C0B8`         | `#4A5040`         | Dimmed body copy                  |
| `--dim`   | `#3A5048`         | `#A09070`         | Disabled / very muted             |
| `--border`| `rgba(196,169,106,0.14)` | same    | Subtle divider                    |
| `--borderb`| `rgba(196,169,106,0.22)`| same   | Stronger divider (weave pattern)  |
| `--navbg` | `rgba(17,22,20,0.96)` | `rgba(240,235,224,0.96)` | Sticky nav backdrop    |

### How theme switching works

`data-theme` is set on `<html>` by a small JS toggle (no framework):

```js
var root = document.documentElement;
root.setAttribute('data-theme', t);   // 'light' or 'dark'
localStorage.setItem('mesh671-theme', t);
```

On load it reads `localStorage` first, then falls back to `prefers-color-scheme`. Both the `[data-theme]` attribute selector and the `@media` block are declared in parallel so either mechanism works.

**Cascade note:** Light-mode overrides for elements with hardcoded dark backgrounds (hero, stats, `section.alt`) must come *after* the base rule in the stylesheet, or use higher specificity (`[data-theme="light"] .hero` vs `.hero`).

---

## Typography

```
EB Garamond   — headings, hero h1, stat numbers, footer wordmark
DM Sans       — body copy, nav, buttons (base font)
DM Mono       — eyebrow labels, chips, MQTT table, monospace callouts
```

Loaded via Google Fonts. All weights/styles preconnected at `<head>`.

Scale:
- Hero h1: `clamp(40px, 6vw, 58px)`, weight 700, EB Garamond, `line-height: 1.08`
- Section h2: EB Garamond, `font-size: clamp(22px, 3vw, 30px)`
- Body: 16px / 1.65

Italic accent (`<em>`) in the hero uses `color: var(--reef)` to pull the teal into the heading.

---

## Layout

Single-column sections with `padding: 72px 5vw`. Max content width ~780px set on `.section-inner` / prose blocks. Nav sticky at top with `backdrop-filter: blur(12px)`.

Hero is full-height (`min-height: 100vh`), flex row: left text column + right island SVG.

---

## Hero section

```
.hero
  background: layered radial-gradients (oceanic depth) → overridden in light mode to var(--rock)
  min-height: 100vh
  display: flex; align-items: flex-start; padding: 7vh 5vw 6vh

  .hero-island   (position: absolute, inset: 0)
    <svg id="guam-svg">  — inline SVG, coordinate-space matches real lat/lon
      outline path: stroke uses style="stroke:var(--reef)" (not the stroke attr)
      fill: rgba(45,196,168,0.06)  — subtle teal wash
      #guam-mesh-lines: JS-drawn <line> and <circle> inside clipPath
        colors overridden by CSS: stroke/fill use var(--sand) !important
      island glow: filter: drop-shadow on .hero-island svg

  .hero-content   (position: relative, z-index: 2)
    background: dark glass in dark mode → rgba(240,235,224,0.72) in light mode
    backdrop-filter: blur(6px)
```

The island SVG uses real geographic coordinates (`viewBox="144.615 13.22 0.345 0.46"`) and a Y-flip transform (`translate(0,26.9) scale(1,-1)`) to correct for SVG's inverted Y-axis vs latitude.

---

## Grain texture

Full-page noise overlay via `body::after`:

```css
body::after {
  position: fixed; inset: 0; pointer-events: none; z-index: 9999;
  opacity: 0.08;
  mix-blend-mode: overlay;        /* adapts to both light and dark backgrounds */
  background-image: url("data:image/svg+xml, <SVG feTurbulence fractalNoise …>");
  background-size: 260px 260px;
}
```

`mix-blend-mode: overlay` means the same noise texture works on dark backgrounds (adds brightness variation) and light backgrounds (subtracts, adds depth) without separate light/dark versions.

Parameters: `baseFrequency="0.72"`, `numOctaves="4"`, `stitchTiles="stitch"`, desaturated via `feColorMatrix`.

---

## Components

### Cards (why-cards)
- `background: var(--rock)`
- Left accent border: `border-left: 2px solid rgba(45,196,168,0.12)`
- Box-shadow: inner highlight + ambient drop
- Hover: border brightens, background shifts to `--rock3`

### Gear cards
- `background: var(--rock3)`, 1px border `var(--borderb)`
- Multi-layer shadow (near + ambient)
- Hover: lifts 3px, border switches to `var(--reef)`, ambient teal glow

### Stats bar
- `background: linear-gradient(to bottom, #0c1410, #111614)` in dark mode
- Overridden to `var(--rock2)` in light mode
- Numbers: EB Garamond, `color: var(--sand)`, gold text-shadow glow

### Chips / badges
- `background: rgba(45,196,168,0.07)`, `border: 1px solid rgba(45,196,168,0.18)`
- `color: var(--reef)`, DM Mono

### Weave divider
- CSS `repeating-linear-gradient` crosshatch pattern using `var(--borderb)`
- Height 28px, sandwiched between `border-top/bottom: 1px solid var(--borderb)`

---

## Sections that need dark-mode overrides

Any section with a hardcoded background must have a light-mode override placed *after* it in the CSS:

| Selector      | Dark bg                               | Light override     |
|---------------|---------------------------------------|--------------------|
| `.hero`       | multi-layer radial gradient           | `var(--rock)`      |
| `.hero-content` | `rgba(10,18,14,0.72)` glass          | `rgba(240,235,224,0.72)` |
| `.stats`      | `linear-gradient(#0c1410, #111614)`   | `var(--rock2)`     |
| `section.alt` | `linear-gradient(#0e1916, #131e1b)`   | `var(--rock2)`     |

---

## Footer

EB Garamond wordmark: `caldwell` in `var(--text)` + `.tech` in `var(--reef)`. Links back to caldwell.tech.
