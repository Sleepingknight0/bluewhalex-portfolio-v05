# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page personal tech portfolio for **Kittiphum Prasomsap (BlueWhaleX)**. Self-contained static site, "SpaceX monochrome flight-deck" aesthetic (pure black/white/gray, mission-control terminology), fully bilingual EN↔Thai. **No build system, framework, or package manager** — one hand-authored HTML file (~3,750 lines) with inline CSS and JS, in three blocks: a `<style>` head, the markup, then a single IIFE `<script>` at the end of body. The file is edited often — locate things by searching for ids/class names, never by remembered line numbers.

## Build / run / test

Nothing to compile. Only external dependency is Google Fonts (Chakra Petch + IBM Plex Mono) via CDN. Preview by serving the project root (asset paths are relative):

```bash
python -m http.server 8137    # http://localhost:8137/index.html
```

Browser verification is done with `playwright-cli` (see the `/playwright` skill); screenshots go to `output/playwright/`. There is no test suite — verify changes in a real browser at 1440×900, 1024×768, 390×844, 360×800, in both languages, with the project dialog opened.

Two traps when driving this page from a headless browser:

- **`screenshot --full-page` is useless here.** 33 `.rv` elements only get `.in` when the IntersectionObserver fires, so a full-page capture of an unscrolled page is almost entirely black. Scroll section by section (the page's own number keys 1–6, or the `.registry-jump` anchors) and take viewport screenshots.
- The boot overlay must be cleared first — `press Escape`, then assert `document.body.classList.contains('ready')` before interacting.

Sanity checks worth running after any edit:

```bash
git diff --no-index index.html bluewhalex-portfolio.html   # no output = distribution copy in sync
grep -o "assets/[A-Za-z0-9_./-]*" index.html | sort -u      # every referenced asset must exist on disk
```

## File layout

- **`index.html`** — the site. The only file you normally edit.
- **`bluewhalex-portfolio.html`** — byte-identical distribution copy. **After editing `index.html`, re-sync it** (`cp index.html bluewhalex-portfolio.html`) and verify with `git diff --no-index index.html bluewhalex-portfolio.html` (no output = in sync).
- **`assets/images/`** — referenced images, including `projects/` (real app screenshots used by cards + dialog gallery) and `logo.png`.
- **`assets/videos/`** — referenced videos: `hero.mp4` (hero background), `grok-logo.mp4` (portrait on a **raw green screen**, keyed live in JS — see below), `ai-sector.mp4` (AI sector background).
- Root-level media (`grok-*`, `imagine-*`, `GettyImages-*`, loose `.png`s) and `index.mk4-backup.html` are raw source material / old snapshots — **not referenced by the live page**. Same for clips in `assets/videos/` beyond the three above.
- `.gitignore` keeps the deployable video set small with `assets/videos/*` plus negations for `hero.mp4` and `ai-sector.mp4` only. **`grok-logo.mp4` is therefore referenced by the live page but not tracked in git** — a fresh clone renders the hero with no portrait and a silent chroma-keyer no-op. Don't "fix" the missing file by deleting the reference; the file must be copied in out-of-band (or the ignore rule negated deliberately). `output/`, `.playwright-cli/`, and `.claude/` are ignored too.
- Tracked but unreferenced: `assets/images/race-garage.jpg`, `assets/images/ops-network.jpg`. Safe to reuse, not safe to assume live.

## Page structure (DOM order)

`nav.rail#rail` (right-edge dot navigator, hidden ≤900px) → `nav#nav` → `.drawer` → `main#main`: `#hero` (bg video + chroma-keyed portrait overlay) → `#m1` "project registry" (all three tiers) → `#specs` → `#log` → `#uplink` (contact) → `#reserved` (AI sector) → `footer` (dot-lattice canvas) → `dialog#projectDialog`.

The registry contains, in order: a jump-nav bar (`.registry-jump`, anchors with `aria-current` tracking), then three tiers — each tier is a *different* UI mechanism, so don't assume the card pattern generalizes:

- **01 // MEGA PROJECTS** (`#mega-projects`) — a three-slide `.mega-showcase` carousel of *vision chapters* (Earth → Moon → Event Horizon), not projects. Every slide carries the `mega_empty` badge ("NO MEGA PROJECT ASSIGNED — VISION ACTIVE"). The empty state is intentional; the carousel is the content.
- **02 // ADVANCED PROJECTS** (`#advanced-projects`) — a `.advanced-workspace` split into a live display panel (`#advancedDisplay`) and a four-button selector (`[data-advanced]`). The display copy is rendered from the JS object `ADVANCED_CONCEPTS`, not from markup. These are reserved *themes*; each render stamps a `FUTURE CONCEPT — NOT A CLAIMED DEPLOYMENT` line (`#advancedHonesty`). Never present them as shipped work, and never remove that honesty line.
- **03 // MINI PROJECTS** (`#mini-projects`) — `.catalog-toolbar` (free-text `#projectSearch` + three `role="radiogroup"` chip rows: STATUS / STACK / CLASS, plus `#projectClear`) over `#projectGrid` of `[data-project-card]` articles (currently two real projects), paginated 12 at a time via `#projectMore`, with `#projectCount` and a `#projectEmpty` no-match state. Filtering is driven by **`data-*` on the card**, not by its text: `data-status`, `data-live`, `data-class`, `data-stack` (space-separated slugs). A new card must carry all four or it will silently drop out of every filter.

`#log` ("FLIGHT LOG / TRAJECTORY") is a career ledger: a `.log-sheet` with `.log-band` groups (`is-now`/`is-past`) of `.log-entry` roles (NACC seal image: `assets/images/nacc.png`). This is the owner's real employment history — org names, titles, and dates are owner-asserted facts; never embellish or reorder them, and keep every string `data-i18n`-keyed.

## The two real projects (content integrity)

1. **NACC Overnight Building Parking Log** — Streamlit + gspread/Sheets app for security staff logging vehicles found overnight in NACC buildings. Source links to **`captwcan/naccparking`** (upstream); the relation copy states it is a *collaborative project forked to the owner's account* — keep that honest framing. Live: `naccparking.streamlit.app`.
2. **NACC Parking Operations Platform — Gen 2** — end-to-end parking-request platform, Next.js 15 / React 19 / Supabase / Turborepo monorepo (`Sleepingknight0/Parking_request`). Live user app verified: `nacc-parking-user.vercel.app`.

These are **two independent systems, never generations of one product**. Never fabricate metrics, users, deployment claims, or technologies; anything in dialog "facts" must be verifiable from the repos or explicitly owner-asserted. The fork `Sleepingknight0/naccparking` must never be presented as original solo work.

Project screenshots (cards + dialog galleries) are captured from public production pages. Never add screenshots containing credentials, personal records, vehicle registrations, or other operationally sensitive data.

## Runtime systems (one IIFE, `<script>` at end of body)

- **Boot sequence** → `body.ready`; skippable (click/Esc/Enter/Space); auto-finishes under reduced motion.
- **Chroma keyer** (`initGrokChroma`): draws `grok-logo.mp4` frames to `#grokCanvas`, keys out the green screen per pixel (distance-from-green + feather + **unconditional despill** — any kept pixel with `g > max(r,b)` gets green clamped; this is what prevents the dark-green hair rim, don't remove it). Uses `requestVideoFrameCallback` when available; skips pixel work whenever `grokCanvas.clientWidth` is 0 (a defensive "no layout box yet" guard — the portrait is **never** hidden by breakpoint: at ≤900px it tucks right, at ≤600px it goes full-width above the mission plate); under reduced motion renders **one static keyed frame** instead of removing the portrait. Canvas has a CSS `mask-image` fade at the bottom to dissolve the waist-line cut.
- **Project dossier dialog**: native `<dialog id="projectDialog">` + `showModal()` (native Esc; an extra manual Tab wrap is layered on), scroll lock via `body.modal-open`, focus restored to the triggering `.project-open` button on close. Content is rendered from the **`PROJECTS`** data object (all copy bilingual `{en:{...},th:{...}}` + `gallery` with real app screenshots). The gallery is a scroll-snap carousel with prev/next and a live count.
- **Mega carousel** (`paintMega`/`goMega`/`armMegaAuto`): scroll-snap track with a **cloned first slide appended** for seamless wrap — index math is over `megaOriginals`, so anything iterating `.mega-slide` must skip `[data-mega-clone]`. Auto-advances every `MEGA_CYCLE` (6500 ms) only when `canMegaAuto()` holds: not reduced-motion, not user-paused (`#megaToggle`), in view, tab visible. Manual interaction calls `holdMegaAuto()` to suspend and resume.
- **Advanced concept selector** (`renderAdvanced`): swaps image/kicker/title/summary/criteria from `ADVANCED_CONCEPTS[id]` for the current `lang`, syncs `aria-pressed` across options, arrow keys move between slots, and on ≤820px a click scrolls the display into view.
- **Footer FX** (`initFooterFx`, `#footerFx`): canvas dot lattice, DPR-capped at 2, gap derived from `min(w,h)/32`, twinkle via per-dot phase. Rebuilds on debounced resize and on first IntersectionObserver hit (size settles late); pauses on `visibilitychange`; under reduced motion paints **one static frame**.
- **Nav**: top bar collapses to a burger **on scroll even on desktop** (`nav#nav.desktop-compact`); the hidden links get `visibility:hidden` so they leave the tab order (keep this — a11y). The `.drawer` is a `role="dialog"` panel with a **hand-rolled** Tab trap + Esc (it is not a `<dialog>` — unlike `#projectDialog`). Number keys 1–6 jump `hero/m1/specs/log/uplink/reserved`; disabled while the dialog is open or while typing.
- **Misc**: IntersectionObserver reveals (`.rv`/`.in`) and `#rail`/nav active-state sync, jump-bar docking, spec counters (`countUp`), starfield canvas, `#coords` click-to-copy → `toast()`, custom cursor + magnetic buttons (non-touch, non-reduced-motion only).

### Two layout constraints that are easy to break

- **`.tier-head[id]` `scroll-margin-top` must budget the UNDOCKED jump bar** (`--registry-h`, not `--registry-docked-h`). `.registry-jump` is sticky and holds flow space at its natural height; the anchor scroll is computed while it is still undocked, then docking shrinks it by `--registry-h - --registry-docked-h` (~30px) and reflows the heading *up* by that much. Budgeting for the docked height lands headings ~8px under the bar — invisible in English, but it clips Thai tone marks and upper vowels. Mobile is exempt because the `.is-docked` rules live inside `@media(min-width:769px)`.
- **`.rail a::after` floats over the content column.** The label extends left from a fixed right-edge rail, so it lands on whatever is beneath. Nearly everything is near-black, but the active `.advanced-option` is `#F0F0F0`; the label carries its own dark chip so it stays legible there. Keep the chip if you restyle the rail.

## i18n (five separate channels — pick the right one)

English is authored inline in the markup and snapshotted to `data-en` at load; Thai comes from the `TH` dictionary or `{en,th}` pairs. A new string needs *one* of:

1. `data-i18n="key"` + a `TH[key]` entry — visible text in static markup.
2. `data-aria-th` / `data-alt-th` / `data-placeholder-th` — the Thai counterpart of an `aria-label`, `alt`, or `placeholder`. `applyLang()` snapshots the English into `data-*-en` on first pass, so **only the Thai side is authored**.
3. `{en,th}` inside `PROJECTS` — anything rendered into the dossier dialog.
4. `{en,th}` inside `ADVANCED_CONCEPTS` — anything rendered into the Advanced display panel.
5. An inline `lang==='th'? … : …` ternary in JS — reserved for generated strings (catalog counts, carousel aria-labels, close button, honesty line). Avoid adding more of these; prefer 1–4.

The `data-i18n` snapshot only covers **load-time** markup, so JS-injected content must never rely on it. `applyLang()` is the single fan-out point: it re-renders the open dialog, `renderAdvanced`, `updateMegaToggle`, and `updateProjectCatalog` — **any new dynamic region must be re-rendered there too**, or it will stay in the previous language after a switch. Thai `h2.split` headings render as plain text (Thai script must not be char-split); English ones get per-char animation. Language choice persists to `localStorage` under `bwx-lang`.

## Conventions

- Design tokens live in `:root` (`--void`, `--bone`, `--signal`, `--fm/--fd` fonts, `--e/--e-out` easings). Reuse them; keep strict monochrome; keep aerospace copy tone in both languages (Thai natural, technical terms may stay English).
- Every new visible string needs an English side and a Thai side through one of the five i18n channels above.
- Gate any new animation behind the `RM` (reduced motion) and `TOUCH` flags like the existing code; pair every `click` with a keyboard path. Long-running canvas loops should also stop on `visibilitychange` — every existing one does.
- Adding a real project: add an entry to `PROJECTS` (copy the `parking` shape — `source`/`live`/`gallery` plus `en` and `th` blocks of title/status/summary/relation/facts/workflow/architecture/stack/highlights), add a card `<article data-project-card>` in the appropriate tier with a `.project-open` button whose `data-project` matches the new key, and update that tier's head/state labels **and** the matching `.registry-jump` state label (`jump_*_state`) — they are separate strings and drift apart easily. Only verified or owner-asserted facts.
- Two-space indentation; kebab-case CSS classes, camelCase JS. Short imperative commits (`fix: preserve Thai heading animation`).
- Do not introduce a bundler, npm, or a framework — the point of this project is a single portable HTML file.
- `AGENTS.md` carries overlapping guidance for other agents — keep the two consistent if conventions change. `README.md` still advertises a `PORTFOLIO_CONTENT.md` that does not exist, and the three docs quote three different preview ports; treat this file as authoritative.
