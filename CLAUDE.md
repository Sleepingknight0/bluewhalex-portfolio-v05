# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page personal tech portfolio for **Kittiphum Prasomsap (BlueWhaleX)**. It is a self-contained static site with a "SpaceX monochrome flight-deck" aesthetic (pure black/white/gray, aerospace/mission terminology) and full English↔Thai bilingual support. There is **no build system, framework, package manager, or test suite** — the deliverable is one hand-authored HTML file with inline CSS and JS.

## Build / run / test

There is nothing to build or compile. Everything is inline in `index.html`; the only external dependency is Google Fonts (Chakra Petch + IBM Plex Mono) loaded via CDN, so a network connection is needed for correct typography.

Preview locally by serving from the project root (asset paths are relative, so serve the root, don't open a nested copy):

```bash
python -m http.server 8000    # then open http://localhost:8000/index.html
```

Opening `index.html` directly via `file://` also works. Screenshots from past preview sessions live in `output/playwright/` (the `/playwright` skill was used); `.playwright-cli/` holds its transient page snapshots.

## File layout

- **`index.html`** — the site. This is the only file you normally edit. ~1645 lines: inline `<style>` (~L18–825), HTML body sections (~L920–1167), inline `<script>` IIFE (~L1169–1643).
- **`bluewhalex-portfolio.html`** — a **byte-identical copy** of `index.html` (distribution/download alias). If you edit `index.html`, re-sync this file or it goes stale.
- **`index.mk4-backup.html`** — an older ("mk4") snapshot kept as a backup. Do not treat as current.
- **`assets/images/`** and **`assets/videos/`** — the curated, renamed media the page actually references (9 images + `hero.mp4`).
- **Root-level `grok-*.jpg/mp4`, `imagine-*.mp4`, `Layer 0.png`, `bluewhalex-logo.png`, and extra clips in `assets/videos/`** — raw/original source material, **not referenced by the live page**. The live asset set is only what's listed under "referenced assets" below.

Referenced assets: `assets/images/{logo,portrait-suit,ops-network,office-skyline,studio-screen,lab-robotics,cleanroom-wafer,race-garage,whiteboard-flow}.png/.jpg` and `assets/videos/hero.mp4`. Nothing else in the tree is loaded at runtime.

## Architecture of `index.html`

All runtime behavior is one IIFE (`(function(){ "use strict"; … })()`). Two capability flags gate almost everything and must be respected in any new code:

- `RM` = `prefers-reduced-motion: reduce` → skip/short-circuit animations (boot, starfield, parallax, count-ups, char reveals all check this).
- `TOUCH` = coarse pointer → disables the custom cursor and magnetic/tilt effects; adds `body.touch`.

Runtime systems, each a self-contained block in the IIFE:

- **Boot sequence** (`#boot`) — fake telemetry log + progress bar, then `body.ready`. Skippable via click/Esc/Enter/Space; auto-finishes under `RM`.
- **Starfield** — `<canvas id="stars">` with twinkling stars + occasional meteors; pauses on tab hide.
- **Custom cursor + magnetic buttons** — dot/ring follower; `.mag` elements attract toward the pointer; sections with `.content` tilt slightly. All non-touch, non-RM only.
- **Scroll systems** — top progress bar, nav `scrolled` state, and parallax on `.bg[data-speed]` backgrounds via a `--py` CSS var.
- **Reveal + count-up** — an `IntersectionObserver` adds `.in` to sections on entry, animates `[data-count]` numbers (`data-prefix`/`data-suffix`/`data-dec` modifiers), and reveals `h2.split` heading characters. A second observer drives active-nav highlighting via the `navKey` map.
- **Backgrounds** — `.bg[data-src]` elements get their `background-image` set from JS after preloading, so sections never flash blank.
- **Hero video** — `assets/videos/hero.mp4`, muted/looping, with layered fallbacks (autoplay retry on first gesture, poster image underneath) so it degrades gracefully.
- **Drawer menu** — mobile nav with focus trap + Esc close; number keys `1–6` jump to sections.
- **Lightbox** — click/Enter on `#mediaGrid .media-card` opens `data-full` full-size with `data-cap` caption.
- **Clocks** — live UTC clock (`#clock`) and a mission-elapsed timer (`#met`, `MET T+…`).

## Conventions when editing

**Design system:** colors, fonts, easings, and layout metrics are CSS custom properties in `:root` (`--void`, `--bone`, `--signal`, `--fd`/`--fm` fonts, `--e`/`--e-out` easings, `--nav-h`, etc.). Reuse these rather than hardcoding. Copy tone is aerospace/mission ("MISSIONS", "FLIGHT LOG", "ESTABLISH UPLINK", "ALL SYSTEMS NOMINAL"). Keep the strict monochrome palette.

**Adding a translatable string (bilingual is required):** English is authored inline in the markup and is the source of truth. Add `data-i18n="some_key"` to the element, then add a matching `some_key: "…"` entry to the `TH` dictionary in the script (~L1560). On load, JS snapshots each element's English into `data-en`; the toggle swaps between `data-en` and `TH[key]` and persists the choice in `localStorage['bwx-lang']`. Headings that also animate use `h2.split` and are handled separately by `splitHeading()` — a translated `h2.split` needs both `class="split"` and `data-i18n`.

**Adding a content section:** give it a unique `id`, register that id in the `navKey` map (for nav highlighting), in the active-nav observer's id list, and in the number-key `keys` map if it should be keyboard-jumpable. Use existing section scaffolding (`.content` > `.frame`, `.eyebrow`, `.rv` reveal class, `.panel` variant for text-only sections).

**Adding a media card:** append a `<figure class="media-card" tabindex="0" role="button" data-full="…" data-cap="…">` to `#mediaGrid` following the existing pattern; the lightbox wires up automatically.

**Animation additions must be gated** by `RM`/`TOUCH` the same way existing blocks are, and any interactive element should have a keyboard path (the code consistently pairs `click` with `keydown` Enter/Space).

Do not introduce a bundler, npm, or a framework unless explicitly asked — the whole point of this project is a single portable HTML file.
