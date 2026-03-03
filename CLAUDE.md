# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dawn is Shopify's reference theme for Online Store 2.0. It uses a **web-native, HTML-first** approach: Liquid renders HTML server-side, CSS handles styling via custom properties, and vanilla JavaScript provides progressive enhancement through Web Components. There is no build step, no bundler, and no JS framework.

## Development Commands

```bash
# Local development server
shopify theme dev --store <store-name>

# Lint Liquid templates
shopify theme check

# Format code
npx prettier --write .
```

There is no test suite. Quality is enforced via Theme Check linting and Lighthouse CI audits (see `.github/workflows/ci.yml`).

## Architecture

### Template Hierarchy

`layout/` → `templates/` → `sections/` → `snippets/`

- **layout/** — Master layouts (`theme.liquid`, `password.liquid`). All pages render through these.
- **templates/** — JSON files that define which sections appear on each page type. These are not Liquid — they declare section composition as JSON.
- **sections/** — Self-contained Liquid components with their own settings schema (`{% schema %}`). Each section defines merchant-editable settings and block types.
- **snippets/** — Reusable Liquid partials included via `{% render %}`. No schema — they receive data through parameters.
- **config/** — `settings_schema.json` defines global theme settings. `settings_data.json` stores merchant choices. Theme version lives in `settings_schema.json` under `theme_info`.

### JavaScript Patterns

All JS lives in `assets/` as standalone files loaded via `<script>` tags (no ES modules, no imports). Key patterns:

- **Web Components** — UI behaviors are encapsulated as custom elements extending `HTMLElement` (e.g., `<product-info>`, `<cart-drawer>`, `<modal-dialog>`, `<slider-component>`). Defined in `global.js` and feature-specific files.
- **Pub/Sub** — Components communicate through `pubsub.js` (`subscribe()`/`publish()`). Events are defined in `constants.js` as `PUB_SUB_EVENTS`: `cartUpdate`, `quantityUpdate`, `optionValueSelectionChange`, `variantChange`, `cartError`.
- **HTMLUpdateUtility** — In `global.js`, handles DOM diffing/swapping with View Transition API support.
- **Debounce** — 300ms debounce timer (`ON_CHANGE_DEBOUNCE_TIMER` in `constants.js`) used across interactive components.

### CSS Patterns

- **`base.css`** is the main stylesheet. Component styles use `component-*.css` and section styles use `section-*.css`.
- **CSS custom properties** drive theming: colors, fonts, spacing, shadows. Color schemes are applied via `.color-{{ scheme }}` classes.
- **Responsive breakpoints**: 750px (tablet), 990px (desktop). Mobile-first.
- **Deferred loading**: Non-critical CSS uses `media="print" onload="this.media='all'"`.

## Code Style

- **Prettier**: 120 char width, single quotes in JS, double quotes in Liquid (see `.prettierrc.json`)
- **Theme Check**: Linting with `MatchingTranslations` and `TemplateLength` disabled (see `.theme-check.yml`)
- No TypeScript, no CSS preprocessors, no polyfills — targets evergreen browsers only

## Localization

53 locale files in `locales/`. English (`en.default.json`) is the source. Schema translations are in `locales/en.default.schema.json`. Translation management is configured in `translation.yml`.
