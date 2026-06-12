# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single, self-contained HTML pitch deck for **LexorAI** — a Brazilian legal-tech product that searches qualified precedents from the TST and STF courts. The entire deck lives in [lexor-pitch.html](lexor-pitch.html): inline CSS, inline JS, no build step, no package manager, no dependencies installed locally. All UI copy is in Brazilian Portuguese.

## Running / previewing

There is nothing to build, lint, or test. Open the file directly in a browser:

```powershell
start lexor-pitch.html
```

- Opening as a local `file://` works for everything except the screenshot images (see "Missing assets" below). Fonts load from the Fontshare CDN, so font rendering needs an internet connection.
- **Export to PDF / print:** `P` or the topbar button opens a themed pre-print modal (`#pdfModal`) that tells the user the two dialog settings that matter — **Margens: Nenhum** (else a white paper border frames each dark slide) and **Background graphics: on** — then "Abrir impressão" calls `window.print()`. The `@media print` rules set a **16:9 page** (`@page{size:1280px 720px}` = PowerPoint widescreen 13.333"×7.5") so each page matches the slide aspect with no empty band (an A4/Letter page leaves a tall dark gap below a 16:9 slide). The 1920×1080 slide is shrunk with `zoom:.66` to fill that page. The print page background is dark (`var(--bg)`), so the tiny residual letterbox blends instead of showing white. `break-inside:avoid` is intentionally NOT used — it turns any overflow into a hard clip instead of letting the engine scale-to-fit. Dark theme is forced via `print-color-adjust:exact` and every entrance-animation element (`.reveal*`) is forced to its final visible state. **If you add new content gated on `.slide.visible`, it must also be forced visible in the print block or it will vanish from the PDF.** `zoom` is honored by Chromium/Edge/Safari print, not Firefox — export from a Chromium browser.
- **Navigation:** arrows / space / PageUp-Down, `Home`/`End`, swipe, mouse wheel, and the dot controls. `F` toggles fullscreen.
- **Letterbox margins:** the fixed 16:9 stage leaves margins in any non-16:9 window (browser chrome makes a windowed view wider than 16:9); fullscreen (`F`) is exactly 16:9 and has no margins. To avoid a visible seam at the stage edge, the deck background gradient is painted **once on `.deck-viewport`**, and `.deck-stage` + `.slide` are `background:transparent` — so one continuous gradient spans margins and slide area with no edge line. Consequence: per-slide screen background lives on the viewport, not the slide. The slide's own gradient is re-applied **only inside `@media print`** (each PDF page needs its own backdrop).

## Architecture (one file, but several conventions)

**Fixed-stage scaling.** The deck is designed on a fixed **1920×1080** canvas (`.deck-stage`). The `SlidePresentation` class (bottom `<script>`) computes a uniform scale factor from the viewport and applies a single `transform: translate() scale()` to the stage. **Author everything in absolute 1920×1080 pixel coordinates** — most slide elements use `position:absolute` with hardcoded `px`. Do not use responsive/viewport units inside slides; the stage transform handles all responsiveness.

**Slides.** Each slide is one `<section class="slide">` under `<main class="deck-stage">`. Visibility is driven by two classes toggled in `show()`: `.active` (display) and `.visible` (triggers entrance animations). There are 13 slides; slides 1 and 13 also carry `.cover`.

**Per-slide scoped styles.** Global theme + shared components live in the `<head>` `<style>`. Slide-specific CSS is placed in its own `<style>` block **immediately after** that slide's `<section>` (e.g. the `.fric`, `.pillar`, `.bento`, `.shield` rules). When adding or restyling a slide, follow this pattern rather than growing the head block.

**Entrance animations.** Elements get a base class (`.reveal`, `.reveal-l`, `.reveal-s`) plus a stagger class `.d1`–`.d9` for transition-delay. They animate in only once the parent slide has `.visible`. Reuse these classes instead of writing new keyframes for entrances.

**Theme.** All colors/fonts/spacing come from CSS custom properties in `:root` (the "Esmeralda Editorial" emerald palette). Fonts: `--font-d` Boska (display/serif), `--font-b` Supreme (body), `--font-m` JetBrains Mono (technical labels). Change the look via these variables, not per-element overrides.

**Canvas animation.** `buildGraph(id)` renders the animated orbital "constellation" on a 620×620 `<canvas>` for the cover (`#coverGraph`) and closing (`#closeGraph`) slides. It respects `prefers-reduced-motion` (renders a single static frame).

**Hardcoded slide numbers.** Each slide hardcodes its number in two places: the `.snum` span (`NN / 13`) and the right-hand label in `.sfoot`. If you add, remove, or reorder slides, update both on every affected slide and the `/ 13` total.

## Product screenshots

Slides 7–9 embed four product screenshots from `assets/imgs/`, in deck order:
`img_1.png` (pesquisa, slide 7) · `img_2.png` (resultado ranqueado, slide 8) · `img_3.png` (chat, slide 9) · `img_4.png` (histórico, slide 9). They render inside the `.browser` mockup frame. If any image fails to load, a `shotFallback()` script swaps it for a clean labeled placeholder (`.shot-fallback`) instead of a broken-image icon — so a missing/renamed file degrades gracefully on screen and in the PDF.
