# Repository Guidelines

## Project Structure & Module Organization

This repository is a self-contained bilingual portfolio site with no framework or build step. Make normal source changes in `index.html`, which contains the HTML, CSS, and JavaScript. Keep `bluewhalex-portfolio.html` synchronized as the distribution copy after every approved change. Do not edit `index.mk4-backup.html`; it is a historical snapshot.

Runtime media belongs in `assets/images/` and `assets/videos/`. Root-level generated images and videos are source material, not automatically part of the live site. Browser-check screenshots may be stored under `output/playwright/`.

## Build, Test, and Development Commands

Serve the repository root so relative asset paths behave correctly:

```bash
python -m http.server 8000
```

Then open `http://localhost:8000/index.html`. There is nothing to compile or install. After editing, verify that the distribution copy matches:

```bash
git diff --no-index index.html bluewhalex-portfolio.html
```

No output means the files are identical.

## Coding Style & Naming Conventions

Use two-space indentation in HTML, CSS, and JavaScript, and preserve the existing single-file organization. Reuse CSS custom properties from `:root` instead of hardcoding colors, fonts, spacing, or easing. Keep the monochrome flight-deck visual language and mission-oriented copy.

Use lowercase kebab-case for CSS classes and descriptive camelCase for JavaScript variables. New translated elements require a stable `data-i18n` key and a matching entry in the `TH` dictionary. Gate motion and pointer effects with the existing `RM` and `TOUCH` capability flags.

## Testing Guidelines

There is no automated test suite. Manually test desktop and mobile widths, English/Thai switching, keyboard navigation, reduced-motion behavior, media loading, drawer focus trapping, and lightbox controls. Check the browser console for errors and capture screenshots for visual changes when useful.

## Commit & Pull Request Guidelines

Git history is not included in this checkout. Use short imperative commits such as `fix: preserve Thai heading animation`. Keep commits focused. Pull requests should summarize user-visible changes, list manual checks, identify changed assets, and include before/after screenshots for layout or animation work.

## Security & Dependencies

Do not add secrets, analytics IDs, or credentials to the HTML. Avoid new CDN dependencies unless justified; Google Fonts is currently the only external runtime dependency.
