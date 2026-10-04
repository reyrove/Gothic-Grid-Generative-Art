# Gothic-Grid-Generative-Art

A seed-based generative art catalogue for **Gothic Grid**, a computational mosaic system. Built as a single-file HTML page with p5.js, styled as a gallery-ready portfolio and licensing surface.

**Live:** `https://reyrove.github.io/Gothic-Grid-Generative-Art/`

---

## Overview

**Gothic Grid** renders an 80 × 80 grid of irregular cells from numeric seeds. Each cell is lit or dark depending on a radial distance function modulated by a sine wave — and each lit cell draws itself as a small square in one of 15–25 gothic colours, over a dim background wash with a scattered glow behind.

The page functions as a **catalogue and licensing surface** for fashion houses, textile studios, and surface designers. Every composition is reproducible by seed, meaning the same number always produces the identical grid.

---

## Features

### Generative Engine (p5.js)

- **80 × 80 grid** — 6,400 cells per composition, each independently lit or dark
- **Radial threshold field** — cell state decided by `distance · 2` modulated by `sin(distance · π · 3)` — producing concentric rings of density that read as hidden structure
- **Gothic palettes** — a base hue drawn from nine anchors (reds, purples, blues, greens), then 15–25 companion colours generated with wide hue spread (±60°), constrained saturation (20–70), and constrained brightness (5–60)
- **Per-cell modulation** — size varies via `sin(i·0.1 + j·0.15)`, saturation via `sin(i·0.1)·cos(j·0.1)`, brightness via `sin(i·0.05)·cos(j·0.05)`, hue via `sin(i·j·0.0001)`
- **Scattered glow** — every second lit cell spawns a soft ellipse at 10% alpha, accumulating into a diffuse stained-glass wash

### Same Seed → Many Surfaces

A single seed drives every instance on the page:

| Instance | Purpose | Tiling |
|----------|---------|--------|
| **Cover** | Hero composition | 1 × 1 |
| **Plate** | Main work (master) | 1 × 1 |
| **Frame** | Framed archival print | 1 × 1 |
| **Print** | Archival print surface | 1 × 1 |
| **Scarf** | Silk scarf (3:1) | 3 × 1 |
| **Textile** | Fabric yardage (4:3) | 2 × 2 |
| **Wall** | Wallpaper (2:3) | 1 × 2 |
| **Archive** | 8 unique seeds | 1 × 1 each |

### Controls

- **New Seed** button / `R` key — regenerate with a fresh seed
- **Download** button / `S` key — export the plate canvas as PNG
- **Click any archive plate** — load that seed into the main view

### Performance

- Debounced resize handling (250ms)
- `IntersectionObserver` pauses off-screen sketches
- `visibilitychange` pauses rendering when the tab is hidden
- `pixelDensity(1)` for a lighter canvas footprint
- Two-frame render (`maxFrames: 2`) so tiles draw once and stop

---

## File Structure

```
Gothic-Grid-Generative-Art/
├── index.html          # Single-file page (structure + styles + generative engine)
├── README.md
└── images/
    ├── fav.svg         # Favicon
    ├── tote.png        # Mockup: tote bag
    ├── tee.png         # Mockup: t-shirt
    └── cushion.png     # Mockup: cushion
```

No build step. No dependencies to install. p5.js is loaded from a CDN.

---

## Getting Started

1. Clone the repo:
   ```bash
   git clone https://github.com/reyrove/Gothic-Grid-Generative-Art.git
   cd Gothic-Grid-Generative-Art
   ```
2. Add the three mockup images to `images/` (optional — they appear in the Commission section).
3. Open `index.html` in a browser.

That's it. The page self-initialises on `DOMContentLoaded`.

---

## How the Generative System Works

### Seeded Random

A linear congruential generator replaces p5's native randomness inside each sketch:

```js
class SeededRandom {
  constructor(s) { this.state = (Math.abs(s) % 2147483647) || 1; }
  rand() { this.state = (this.state * 9301 + 49297) % 233280; return this.state / 233280; }
  range(min, max) { return this.rand() * (max - min) + min; }
  int(min, max) { return Math.floor(this.range(min, max + 1)); }
  pick(arr) { return arr[Math.floor(this.rand() * arr.length)]; }
}
```

Every aesthetic decision — base hue, palette size, grid state, cell colour, glow placement — is drawn from this generator in a fixed order. Same seed, same sequence, same composition.

### Parameter Space (per seed)

| Parameter | Range |
|-----------|-------|
| Base hue | 9 gothic anchors (reds / purples / blues / greens) |
| Background saturation | 5 – 20 |
| Background brightness | 5 – 20 |
| Palette size | 15 – 25 colours |
| Companion hue spread | ±60° from base |
| Companion saturation | 20 – 70 (clamped 10–70) |
| Companion brightness | 5 – 60 (clamped 5–60) |

### The Threshold Field

Each cell's lit/dark state comes from a radial distance function:

```js
const dx = i / COLS - 0.5;
const dy = j / ROWS - 0.5;
const distance = Math.sqrt(dx*dx + dy*dy) * 2;
const threshold = 0.4 + Math.sin(distance * Math.PI * 3) * 0.3;
grid[i][j] = sr.rand() > threshold ? 1 : 0;
```

The `sin(distance · π · 3)` term creates three concentric rings — cells closer to a ring's threshold flip between lit and dark more frequently, producing the density bands visible in every composition.

### The Glow Effect

Behind the cells, every second lit cell (`i += 2`, `j += 2`) spawns a soft ellipse:

```js
p.drawingContext.globalAlpha = 0.1;
// for each even-indexed lit cell:
const shadowHue = ((bgHue + sr.range(-60, 60)) % 360 + 360) % 360;
const shadowSat = 20 + sr.range(0, 20);
const shadowBright = 20 + sr.range(0, 20);
p.fill(shadowHue, shadowSat, shadowBright);
p.ellipse(i*cellSize + cellSize/2, j*cellSize + cellSize/2, cellSize*2);
p.drawingContext.globalAlpha = 1;
```

Overlapping ellipses accumulate into a diffuse wash that reads as out-of-focus light passing through stained glass.

### Tiling Logic

Rectangular surfaces (scarf, textile, wall) tile the square composition N×M. Each tile is drawn into its own translated coordinate space, centred within its cell:

```js
const tileW = canvasW / tileCols;
const tileH = canvasH / tileRows;
const tileSize = Math.min(tileW, tileH);
// draw square of side tileSize at (ox, oy) for each cell
```

This preserves the square composition's proportions regardless of surface aspect ratio.

---

## Customisation

### Change the default seed

Edit the init function:

```js
function init() {
  seed = 123456789;  // fixed seed instead of random
  // ...
}
```

### Adjust the grid resolution

Edit the constants near the top of the script:

```js
const COLS = 80;
const ROWS = 80;
```

Higher values produce finer, denser mosaics. Lower values produce chunky, more architectural blocks. Both dimensions must divide evenly into the canvas for clean rendering.

### Adjust the gothic anchor hues

Edit the array:

```js
const GOTHIC_HUES = [0, 10, 330, 270, 220, 160, 200, 300, 30];
```

Each value is an HSB hue angle (0–360). Add or remove entries to widen or narrow the base palette.

### Change the radial threshold

Find the grid initialisation:

```js
const threshold = 0.4 + Math.sin(distance * Math.PI * 3) * 0.3;
```

- Change `0.4` to shift the baseline density (lower = more lit cells).
- Change `3` to alter the number of concentric rings.
- Change `0.3` to control how much the sine modulates the threshold (higher = more banding).

### Change surface tiling

Edit `surfaceDefs`:

```js
const surfaceDefs = [
  { id: 'surfacePrint',   wrapId: 'wrap-surfacePrint',   tile: { cols: 1, rows: 1 } },
  { id: 'surfaceScarf',   wrapId: 'wrap-surfaceScarf',   tile: { cols: 3, rows: 1 } },
  // ...
];
```

The `tile` object accepts any `{ cols, rows }` pair.

### Replace mockup images

Swap the files in `images/` or update the `src` attributes in the Commission section:

```html
<img src="images/your-mockup.png" alt="..." loading="lazy" />
```

---

## Design System

| Token | Value | Use |
|-------|-------|-----|
| `--bg` | `#0a0a0a` | Page background |
| `--paper` | `#f4f1ea` | Light surface (Surfaces section) |
| `--gold` | `#b8935a` | Primary accent |
| `--gold-soft` | `#d4b483` | Italic emphasis |
| `--serif` | Cormorant Garamond | Headings, titles |
| `--mono` | DM Mono | Labels, metadata, UI |

Type scale, spacing, and section rhythm match the Skyline, Portrait, and Girih templates. Responsive breakpoints at 900px, 720px, 560px, and 400px.

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `R` | Regenerate with a new seed |
| `S` | Download plate as PNG |
| `Enter` / `Space` | Activate focused archive item |
| `Esc` | Close legal modal |

---

## Browser Support

Modern evergreen browsers with:

- Canvas 2D (universal)
- `IntersectionObserver` (universal in modern browsers)
- `backdrop-filter` (Safari 9+, Chrome 76+, Firefox 103+)
- CSS custom properties (universal)

Fallback: if `IntersectionObserver` is unavailable, sketches render continuously.

---

## Accessibility

- Archive items are keyboard-focusable with `role="button"` and `aria-label`
- Legal modal traps focus, closes on `Esc` and overlay click
- `prefers-reduced-motion` disables transitions and smooth scroll
- All interactive elements have visible focus states (via browser default)

---

## Section Map

| Section | ID | Purpose |
|---------|-----|---------|
| Cover | — | Hero title + live grid |
| Statement | `#statement` | Artist statement |
| Plate | `#work` | Main work + metadata + controls |
| Surfaces | `#application` | Framed print + 4 tiled surfaces |
| Process | `#process` | Three process cards |
| Archive | `#archive` | 8 selectable seeds |
| Commission | `#commission` | Services + mockups + CTA |
| Colophon | — | Footer, links, legal |
| Legal modal | `#legalModal` | Licensing / Terms / Credits |

---

## Deployment (GitHub Pages)

1. Push the repo to GitHub.
2. Go to **Settings → Pages**.
3. Under **Source**, select `Deploy from a branch`.
4. Choose branch `main` and folder `/ (root)`.
5. Save — the site goes live at `https://reyrove.github.io/Gothic-Grid-Generative-Art/`.

---

## Notes on the Source Sketch

The generative engine is adapted almost verbatim from the original p5 sketch. The adaptations that turn it into a catalogue page:

1. **Seeded random replaces p5's `random()`** — a local `SeededRandom` class drives every choice. The original sketch calls `setSeeds()` for fxhash compatibility; that's replaced with the seeded generator directly.
2. **Sketch factory** — `makeSketch({ slotId, seedValue, ... })` returns a p5 sketch that can be instantiated into any DOM slot, enabling cover / plate / frame / surfaces / archive from one seed.
3. **DOM-driven canvas sizing** — `cs = min(windowHeight, windowWidth)` becomes slot-rect dimensions.
4. **Tiling support** — the `tile: { cols, rows }` parameter repeats the square composition across rectangular surfaces.
5. **Drawing logic takes `p` explicitly** — the original used p5's global mode; here `p` is passed so the factory closure works.
6. **`windowResized` replaced by a debounced `handleResize`** — all sketches are destroyed and recreated on resize (250ms debounce), since their canvases are locked to slot dimensions.
7. **`triggerPreview` removed** — the original calls it for fxhash thumbnail generation; not needed in the plain-browser catalogue.

One thing worth noting: the original sketch's `renderGrid` picks colours with `random()` per cell, which means the same seed in p5 would still produce a different result each run (p5's `random()` is not seeded by default). In this adaptation, `sr.int(0, colors.length - 1)` is called per cell in a deterministic sequence, so the composition is fully reproducible.

---

## Related Repositories

| Repo | Theme |
|------|-------|
| [`Skyline-Series`](https://github.com/reyrove/Skyline-Series) | Themed cityscapes |
| [`Portrait-Series-Generative-Art`](https://github.com/reyrove/Portrait-Series-Generative-Art) | Generative faces |
| [`Girih-2-Generative-Art`](https://github.com/reyrove/Girih-2-Generative-Art) | Islamic geometric ornament |
| **`Gothic-Grid-Generative-Art`** | Mosaic grid composition *(this repo)* |

All four share the same catalogue template, design tokens, and seed-based reproducibility model.

---

## Credits

- **Design & Generative System** — Reyhaneh Daneshdoost
- **Typefaces** — Cormorant Garamond, DM Mono
- **Engine** — p5.js 1.9.0
- **Edition** — Gothic Grid, Autumn 2026

---

## License

All artwork, code, and generative systems are the intellectual property of the artist. Compositions may not be reproduced, redistributed, resold, or adapted without a written license.

For licensing, commissions, or private systems: **reyhanehdaneshdoost@gmail.com**