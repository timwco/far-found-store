# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Shopify Liquid theme ("Outstock_cosmetic01" by Velatheme v5.1) for the Far & Found e-commerce store. There is no build system, no package.json, and no compilation step — all files are deployed directly to Shopify.

## Development Workflow

The repo is connected to Shopify via GitHub integration — merging to `main` automatically deploys to the live store. Work on feature branches and open PRs against `main`.

There is no build step and no tests. Validation is done visually in the Shopify theme editor and storefront preview.

To preview changes before merging, you can use the Shopify CLI to push to a dev theme:

```bash
shopify theme dev --store=<store-url>   # live-sync local edits to a dev theme
shopify theme push --theme=<theme-id>   # push to a specific unpublished theme
```

## Architecture

### Template Hierarchy

```
layout/theme.liquid          ← Master layout; wraps all pages
  ↳ sections/header.liquid   ← Always-present sections
  ↳ {{ content_for_layout }} ← Page-specific template content
  ↳ sections/footer.liquid
```

- **`templates/`** — JSON files that declare which sections appear on each page type (product, collection, page, blog, etc.)
- **`sections/`** — Customizable, editor-configurable page building blocks with their own schema (settings + blocks)
- **`snippets/`** — Reusable partials, included via `{% render 'snippet-name' %}` or `{% include 'snippet-name' %}`
- **`layout/`** — Only `theme.liquid` (storefront) and `password.liquid` (coming soon page)

### CSS / Theming

All theme colors and typography are driven by `config/settings_schema.json` (schema) and `config/settings_data.json` (current values). They are rendered as CSS custom properties in `snippets/css-variables.liquid`, which is included in `<head>` via `layout/theme.liquid`.

Key CSS variables (set from theme settings):
- `--primary`, `--secondary` — Brand accent colors (currently `#b97a4f`, `#f57e7e`)
- `--font-body-family`, `--font-heading-family` — Set from Shopify font pickers
- `--header-bg`, `--footer-bg`, `--menu-bg` — Section-specific backgrounds
- `--btn-default-*` — Button states (normal/hover color, border, bg)

Bootstrap CSS variables are overridden at the bottom of `snippets/css-variables.liquid` (e.g., `--bs-primary`).

To change colors or fonts, update values in `config/settings_data.json` under the `"current"` key, or use the Shopify theme editor UI.

### JavaScript

- **`assets/vela.js`** — Core section lifecycle management; listens for Shopify section editor events (`shopify:section:load`, etc.)
- **`assets/vela-product.js`** — Product page functionality
- **`assets/cart.js`** — Cart drawer/modal and AJAX cart operations (POST to `/cart/add`, `/cart/change`)
- **`assets/predictive-search.js`** — Search suggestions via `/search/suggest`
- **`assets/global.js`** — Focus trap, accessibility utilities

All custom JS uses jQuery and a `window.vela` namespace. Vendor libraries (Bootstrap, Magnific Popup, Ion Range Slider, lazysizes) are in `assets/vendor.js`.

### Section Schema Pattern

Each section file in `sections/` ends with a `{% schema %}` block that defines:
- `settings` — Section-level customization fields shown in the theme editor
- `blocks` — Repeatable sub-components within the section
- `presets` — Default configuration when adding the section to a page

### Localization

Translation strings live in `locales/` as JSON files (22 languages). Use the Liquid `t` filter: `{{ 'key.path' | t }}`. The `locales/en.default.json` file is the source of truth.
