# Geist Playground

A two-file, no-build-step website (`index.html` + `docs.html`) built to learn **Geist**, Vercel's open-source design system, by actually building with it rather than just reading about it. Along the way it grew a bunch of self-contained visual effects (ASCII art, dithering, generative graphics, two playable games, WebGL, etc.) mostly for fun / to see how far vanilla JS could go.

- No framework (no React/Vue/etc.), no bundler, no `npm install`.
- Fonts are the **real, official Vercel Geist fonts**, downloaded and self-hosted in `/fonts` (not loaded from Google Fonts or a CDN).
- Everything — HTML, all CSS, all JS — lives inline in two `.html` files. `index.html` is ~3,440 lines; `docs.html` is ~955 lines.
- Live: [subramaniyajothi6.github.io/asci-web](https://subramaniyajothi6.github.io/asci-web/) · Learn/docs page: [subramaniyajothi6.github.io/asci-web/docs.html](https://subramaniyajothi6.github.io/asci-web/docs.html)

---

## Verified facts (checked against the real Vercel product, July 2026)

The site makes specific factual claims about Geist Pixel. These are checked against Vercel's own announcement and GitHub repos — **they're accurate**:

- Geist Pixel really does ship **5 shape variants**: Square, Circle, Grid, Triangle, Line — each renders every glyph as a grid of that shape.
- It really does have **480 glyphs, 7 stylistic sets, and support for 32 languages**.
- It's really licensed under the **SIL Open Font License** (free/open-source), matching the `fonts/LICENSE.txt` file in this repo.
- Geist Sans and Geist Mono are the other two members of the real Geist type family, distributed via `npm install geist`.

Sources: [Introducing Geist Pixel — Vercel](https://vercel.com/blog/introducing-geist-pixel), [vercel/geist-pixel-font on GitHub](https://github.com/vercel/geist-pixel-font), [vercel/geist-font LICENSE](https://github.com/vercel/geist-font/blob/main/LICENSE.txt)

**Caveat:** `docs.html` also has a whole "Learn Vercel" section describing platform features (Fluid Compute, AI Gateway, Rolling Releases, BotID, etc.). Those read as accurate descriptions of Vercel's product lineup as of when this was written, but platform products change fast — treat that section as "how the author understood Vercel's platform at the time," not a guaranteed-current spec sheet. Geist Pixel's stats above are the ones actually confirmed independently.

---

## `index.html` — the main site

### Structure
A sticky nav, a hero, a horizontally-scrolling marquee, then ~19 `<section>` blocks, then a footer. Every section follows the same visual pattern: an `.eyebrow` label, an `h2.title`, a `.lead` paragraph, then the actual demo.

### Design tokens (top of `<style>`)
CSS custom properties on `:root` define the whole palette (`--blue`, `--purple`, `--pink`, `--red`, `--amber`, `--green`, `--teal`, `--cyan`), spacing/radius, and font stacks. Dark/light theme is two more `:root[data-theme="dark"|"light"]` blocks overriding background/foreground/border variables — switching themes is just flipping one `data-theme` attribute on `<html>`.

### Section-by-section

1. **Hero** — animated canvas background (`heroField`) drawing a faint, slowly shimmering field of random characters (`1 0 + - : * · / × . ; ■ ● ▲ ◆ □ ○`) behind the headline. The headline itself cycles through Geist Pixel's 5 shape variants every 1.5s (`heroShapeCycle`). Below it: a hero hint that opens the command palette.

2. **Stats bar** — four numbers that count up from 0 when scrolled into view (`IntersectionObserver` + `runCount`, driven by `data-count` attributes).

3. **Marquee** — a horizontally auto-scrolling strip of terms, built by duplicating the track's `innerHTML` once so the CSS `translateX(-50%)` loop is seamless.

4. **ASCII FX** — seven `<pre>` blocks, each updated on a `setInterval`, each a different classic text-based effect:
   - **donut.js** — the famous "spinning donut" (`a1k0n`'s algorithm): parametrize points on a torus, rotate them in 3D, project to 2D, keep the nearest point per cell with a z-buffer, and pick an ASCII character by how much simulated light hits that point (`.,-~:;=!*#$@` dark→bright).
   - **matrix.js** — Matrix-style digital rain: each column has a falling "head" that writes a random glyph and clears a trailing row behind it.
   - **plasma.js** — sums several sine waves of `x`, `y`, and distance-from-center to get a value, maps it to a shading ramp (` .:-=+*#%@`).
   - **fire.js** — a classic "doom fire" cellular simulation: seed random heat at the bottom row, each frame average a cell with the 3 cells below it and decay slightly, map heat to a character ramp.
   - **cube.js / tunnel.js / ripple.js** — same "compute a value per (x,y) cell → look up a character" pattern with different math (rotating cube wireframe, polar-coordinate tunnel, radial wave).

5. **Bitmap aesthetic** — real-time **1-bit ordered (Bayer) dithering**. A 4×4 Bayer matrix (`BAYER4`) gives each pixel a different brightness threshold; if the pixel's computed brightness beats its threshold it's drawn "on," otherwise "off" — no actual grayscale, just a clever black/white pattern that reads as gray from a distance. Used for: metaball blobs that react to your cursor, a rotating dot-size halftone screen, and a dithered gradient ramp. Rendered small and scaled up with `image-rendering: pixelated` for crisp chunky pixels.

6. **Visual graphics** — five independent `requestAnimationFrame` particle systems on `<canvas>`:
   - **constellation.js** — particles drift; draw a faint line between any two within a distance threshold, and to the cursor.
   - **flowfield.js** — particles follow an angle read from a math "field," leaving trails (canvas isn't cleared each frame, just painted over with a translucent rectangle).
   - **fireworks.js** — click to launch a rocket that arcs up and explodes into dozens of colored particles under simulated gravity.
   - **orbits.js** — particles are gravitationally pulled toward the cursor.
   - **starfield.js** — points rush outward from center for a hyperspace-warp look, speeding up on hover.

7. **Image → ASCII / bitmap tool** — drop or load a sample image; it's drawn onto a small offscreen `<canvas>`, each cell's RGB is converted to a single brightness value (`0.299R + 0.587G + 0.114B`), and that value indexes into a character ramp (` .`^:;~=+*rsoxIYUJCLQ0OZmwqpdbkhao#MW&8%B@`, dark→bright) for ASCII mode, or into the Bayer threshold for dither mode. Braille and halftone modes work the same way with different output encodings.

8. **Playable Snake + Breakout** — both are the same basic game-loop shape: keep state in plain variables (snake body array / ball position+velocity), read keyboard/mouse input, update state on a fixed-interval `setInterval`, redraw the whole canvas each tick. Snake's high score persists via `localStorage`.

9. **The Lab** — Conway's **Game of Life** (click/drag to seed cells; each cell's next state depends on its 8 neighbors via the classic under/overpopulation rules), a zoomable **Mandelbrot** fractal (click zooms in centered on the click point by re-scanning a smaller `x0,y0` range and raising the iteration cap; shift-click zooms out), and a real **WebGL fragment shader** (a tiny GLSL program that runs once per pixel on the GPU) with three modes — plasma, kaleidoscope, tunnel — falling back to a plain 2D "WebGL not available" message if the browser/GPU doesn't support it.

10. **Pixel sprites** — hand-drawn ASCII-art grids (e.g. `"..X.....X..\n...X...X..."`) get converted into absolutely-positioned colored `<div>` squares (one per non-empty character) by a small `pixelSprite()` helper. Multiple "frames" are stacked and cross-faded on a timer for choppy, deliberately non-smooth 8-bit animation — reinforced by CSS `steps()` easing instead of the default smooth interpolation.

11. **Typography (Geist Pixel)** — static showcase of the 5 shape variants side by side, real `@font-face` declarations pointing at the self-hosted `.woff2` files.

12. **Type Lab** — four sub-tools:
    - **Pixel renderer** — big live text sample you can type into, switch between the 5 shape variants.
    - **Font playground** — sliders for size/letter-spacing, live CSS output.
    - **Glyph inspector** — draws a single character to a `<canvas>`, scans the actual rendered pixels to find the real ink boundaries, and draws baseline/x-height/cap-height guide lines aligned to what's actually on screen (not just theoretical font metrics).
    - **Stylistic set explorer** — toggles Geist Pixel's real OpenType stylistic sets (`ss01`–`ss09`) live via the CSS `font-feature-settings` property, showing before/after glyphs for each.

13. **Color** — the 8 accent colors + a 10-step gray scale; click any swatch to copy its hex to clipboard (`navigator.clipboard.writeText`).

14. **Theme Engine** — a hue + saturation slider pair (plus 5 presets) that **rewrite the site's own live CSS custom properties** (`--blue`, `--purple`, `--pink`, `--teal`, `--cyan`) via `root.style.setProperty()`. Because buttons, links, the hero gradient, and the scroll-progress bar all reference those same `var(--blue)` etc. tokens, the whole page repaints instantly. Deliberately excludes the semantic status colors (green/red/amber). Persists the chosen hue/saturation to `localStorage`; "Reset" calls `removeProperty()` to restore the exact original hard-coded values rather than recomputing an approximation.

15. **Buttons / Forms / Components** — static reference galleries: button variants/sizes/states, inputs/toggles/checkboxes, a sliding-indicator tab control, badges, tooltips, progress bars, skeleton loaders, an avatar stack, and an animated conic-gradient "border beam" effect.

16. **FAQ** — a single-open accordion (only one answer visible at a time) with 7 honest Q&As about the project itself (is it official, does it track you, can you reuse the code, etc.). Uses the CSS `grid-template-rows: 0fr → 1fr` trick for a smooth height animation without JS measuring actual pixel heights.

17. **Build Log** — a vertical timeline built from this repo's **real `git log`** (not fabricated copy) — one entry per actual commit, with the real short hash. Click an entry to expand it and reveal a "click to copy" hash chip.

18. **Footer** — static credit line + tech-stack tags.

### Cross-cutting JS systems (not tied to one section)
- **Theme toggle** — flips `data-theme` on `<html>`, swaps the nav icon between sun/moon SVGs.
- **Cursor spotlight** — a fixed radial-gradient `<div>` that follows the pointer via CSS custom properties updated on `pointermove`.
- **Scroll progress bar** — a 2px bar at the very top whose width tracks `scrollY / (scrollHeight - innerHeight)`.
- **Reveal-on-scroll** — a single shared `IntersectionObserver` adds a `.show` class (which CSS transitions from `opacity:0; translateY(28px)` to visible) to any element with class `.reveal` the first time it enters the viewport; it also triggers the stat counters and progress-bar fills at the same moment.
- **8-bit sound** — a tiny `Sound` object wrapping the Web Audio API (`AudioContext` + `OscillatorNode`) to generate square-wave beeps for clicks/coins/hits, toggleable via a mute button.
- **CRT boot screen** — types out a fake BIOS log character-by-character on load, then fades away (click to skip).
- **Konami code** — the classic ↑↑↓↓←→←→BA sequence toggles a "party mode" (`hue-rotate` animation on the whole page) and rains emoji.
- **Command palette** — `Ctrl/Cmd+K` (or the nav search button) opens a searchable list built from two sources: every `<section id="...">` on the page (auto-collected, using its eyebrow/title/lead text as the search index) plus a hand-written list of quick actions (toggle theme/sound, trigger a toast, copy a random accent color, reset the Theme Engine, toggle party mode, open external links). Arrow keys navigate, Enter runs the highlighted item.

---

## `docs.html` — the companion "learn" page

A classic docs-site layout: sticky top bar with a search-filterable sidebar (grouped into "Start here," "How the effects work," "Learn Vercel," "Geist design system"), and a long scrolling `<main>` with a scrollspy (`IntersectionObserver`) that highlights the current section in the sidebar as you scroll.

It's explicitly the *teaching* version of the main page — each concept from `index.html` gets a plain-language explanation, a small live re-implementation of the same effect (smaller/simpler canvases: `dmDonut`, `dmDither`, `dmNet`, `dmLife`, `dmMandel`, `dmShader`, etc.), and a copyable minimal code snippet showing the core technique in ~5–10 lines. Copy buttons use `navigator.clipboard.writeText`.

Three "courses," in order:
1. **How the effects work** — ASCII animation, dithering, sprites, WebGL shaders, `steps()`-based choppy motion, generative particle systems, image→ASCII conversion, Game of Life, Mandelbrot, generic game loops, and a from-scratch imitation of how Geist Pixel might render (draw text to a hidden canvas, read which pixels are "lit," stamp a shape per lit pixel).
2. **Learn Vercel** — what Vercel is, deployments/preview URLs, Functions & Fluid Compute, rendering strategies (Static/SSR/ISR/PPR), automation (Cron/Queues/Workflow/Sandbox), AI Gateway & AI SDK, storage, scaling/observability, collaboration tools, security, and a full product-map grid.
3. **Geist design system** — the 3 real font families, color tokens, spacing/radius/type-scale tokens, an icon showcase, and a static component catalog (buttons, inputs, badges, switches, avatars, progress, skeletons, tooltips, chips).

---

## Fonts (`/fonts`)

Real, self-hosted Vercel typefaces as variable `.woff2` files:
- `Geist-Variable.woff2` (weight 100–900)
- `GeistMono-Variable.woff2` (weight 100–900)
- `pixel/GeistPixel-{Square,Circle,Grid,Triangle,Line}.woff2` (one file per shape variant)
- `LICENSE.txt` — the actual SIL Open Font License text

---

## Custom domain

Currently served from GitHub Pages at the default `github.io` URL. No custom domain configured yet — GitHub Pages supports one via a `CNAME` file in the repo root plus a DNS record at any registrar, if that changes later.

---

## One-sentence summary

A single-file (well, two-file) vanilla HTML/CSS/JS site built as a hands-on way to learn Vercel's Geist design system — using real, self-hosted Geist/Geist Pixel fonts and real CSS custom-property tokens — extended with self-contained JS experiments (ASCII renderers, 1-bit dithering, generative particle graphics, two playable games, Conway's Game of Life, a zoomable Mandelbrot, a WebGL shader, and a font-metrics glyph inspector) purely because they were fun to build.
