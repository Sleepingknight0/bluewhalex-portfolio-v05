# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page personal tech portfolio for **Kittiphum Prasomsap (BlueWhaleX)**. Self-contained static site, "SpaceX monochrome flight-deck" aesthetic (pure black/white/gray, mission-control terminology), fully bilingual EN↔Thai. **No build system, framework, or package manager** — one hand-authored HTML file (~3,000 lines) with inline CSS and JS.

## Build / run / test

Nothing to compile. Only external dependency is Google Fonts (Chakra Petch + IBM Plex Mono) via CDN. Preview by serving the project root (asset paths are relative):

```bash
python -m http.server 8137    # http://localhost:8137/index.html
```

Browser verification is done with `playwright-cli` (see the `/playwright` skill); screenshots go to `output/playwright/`. There is no test suite — verify changes in a real browser at 1440×900, 1024×768, 390×844, 360×800, in both languages, with the project dialog opened.

## File layout

- **`index.html`** — the site. The only file you normally edit.
- **`bluewhalex-portfolio.html`** — byte-identical distribution copy. **After editing `index.html`, re-sync it** (`cp index.html bluewhalex-portfolio.html`) and verify with `git diff --no-index index.html bluewhalex-portfolio.html` (no output = in sync).
- **`assets/images/`** — referenced images, including `projects/` (real app screenshots used by cards + dialog gallery) and `logo.png`.
- **`assets/videos/`** — referenced videos: `hero.mp4` (hero background), `grok-logo.mp4` (portrait on a **raw green screen**, keyed live in JS — see below), `ai-sector.mp4` (AI sector background).
- Root-level media (`grok-*`, `imagine-*`, `GettyImages-*`, loose `.png`s) and `index.mk4-backup.html` are raw source material / old snapshots — **not referenced by the live page**. Same for clips in `assets/videos/` beyond the three above.

## Page structure (DOM order)

`#hero` (bg video + chroma-keyed portrait overlay) → `#m1` "project registry" (all three tiers) → `#specs` → `#log` → `#uplink` (contact) → `#reserved` (AI sector). The registry contains, in order: a jump-nav bar (`.registry-jump`, anchors with `aria-current` tracking), **01 // MEGA PROJECTS** (vision program, intentional empty state), **02 // ADVANCED PROJECTS** (four clearly-labeled bilingual "CONCEPT SLOT" buttons — themes, not real projects; never present them as shipped work), **03 // MINI PROJECTS** (search toolbar + two real project cards).

`#log` ("FLIGHT LOG / TRAJECTORY") is a career ledger: a `.log-sheet` with `.log-band` groups (`is-now`/`is-past`) of `.log-entry` roles (NACC seal image: `assets/images/nacc.png`). This is the owner's real employment history — org names, titles, and dates are owner-asserted facts; never embellish or reorder them, and keep every string `data-i18n`-keyed.

## The two real projects (content integrity)

1. **NACC Overnight Building Parking Log** — Streamlit + gspread/Sheets app for security staff logging vehicles found overnight in NACC buildings. Source links to **`captwcan/naccparking`** (upstream); the relation copy states it is a *collaborative project forked to the owner's account* — keep that honest framing. Live: `naccparking.streamlit.app`.
2. **NACC Parking Operations Platform — Gen 2** — end-to-end parking-request platform, Next.js 15 / React 19 / Supabase / Turborepo monorepo (`Sleepingknight0/Parking_request`). Live user app verified: `nacc-parking-user.vercel.app`.

These are **two independent systems, never generations of one product**. Never fabricate metrics, users, deployment claims, or technologies; anything in dialog "facts" must be verifiable from the repos or explicitly owner-asserted. The fork `Sleepingknight0/naccparking` must never be presented as original solo work.

Project screenshots (cards + dialog galleries) are captured from public production pages. Never add screenshots containing credentials, personal records, vehicle registrations, or other operationally sensitive data.

## Runtime systems (one IIFE, `<script>` at end of body)

- **Boot sequence** → `body.ready`; skippable (click/Esc/Enter/Space); auto-finishes under reduced motion.
- **Chroma keyer** (`initGrokChroma`): draws `grok-logo.mp4` frames to `#grokCanvas`, keys out the green screen per pixel (distance-from-green + feather + **unconditional despill** — any kept pixel with `g > max(r,b)` gets green clamped; this is what prevents the dark-green hair rim, don't remove it). Uses `requestVideoFrameCallback` when available; skips pixel work while the canvas is hidden (≤520px); under reduced motion renders **one static keyed frame** instead of removing the portrait. Canvas has a CSS `mask-image` fade at the bottom to dissolve the waist-line cut.
- **Project dossier dialog**: native `<dialog id="projectDialog">` + `showModal()` (native focus trap/Esc), scroll lock via `body.modal-open`, focus restored to the triggering `.project-open` button on close. Content is rendered from the **`PROJECTS`** data object (all copy bilingual `{en:{...},th:{...}}` + `gallery` with real app screenshots). `applyLang()` re-renders an open dialog on language switch.
- **i18n**: static markup uses `data-i18n` keys → `TH` dictionary; English inline text is snapshotted to `data-en` at load. **Dynamic dialog content must come from `PROJECTS`, not `data-i18n`** (the snapshot only covers load-time markup). Thai `h2.split` headings render as plain text (Thai script must not be char-split); English ones get per-char animation.
- **Nav**: top bar collapses to a burger **on scroll even on desktop** (`nav#nav.desktop-compact`); the hidden links get `visibility:hidden` so they leave the tab order (keep this — a11y). Number keys 1–6 jump sections; disabled while the dialog is open or while typing.
- **Registry**: jump-bar docking, Mini-project search/filter toolbar, IntersectionObserver reveals (`.rv`/`.in`), starfield canvas, custom cursor + magnetic buttons (non-touch, non-reduced-motion only).

## Conventions

- Design tokens live in `:root` (`--void`, `--bone`, `--signal`, `--fm/--fd` fonts, `--e/--e-out` easings). Reuse them; keep strict monochrome; keep aerospace copy tone in both languages (Thai natural, technical terms may stay English).
- Every new visible string needs both the inline English and a `TH` entry (or an `{en,th}` pair in `PROJECTS`).
- Gate any new animation behind the `RM` (reduced motion) and `TOUCH` flags like the existing code; pair every `click` with a keyboard path.
- Adding a future Advanced/Mega project: add an entry to `PROJECTS` (copy the `parking` shape: title/status/summary/relation/facts/workflow/architecture/stack/highlights, all bilingual, plus `gallery`), add a card `<article>` in the appropriate tier with a `.project-open` button whose `data-project` matches the new key, and update that tier's head/state labels. Only verified or owner-asserted facts.
- Two-space indentation; kebab-case CSS classes, camelCase JS. Short imperative commits (`fix: preserve Thai heading animation`).
- Do not introduce a bundler, npm, or a framework — the point of this project is a single portable HTML file.
- `AGENTS.md` carries overlapping guidance for other agents — keep the two consistent if conventions change.
