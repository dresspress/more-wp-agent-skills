---
name: wp-styling
description: "Use when writing, refactoring, or reviewing CSS and styling in WordPress themes, plugins, custom blocks, or admin interfaces. Focuses on specificity management, avoiding !important, CSS custom properties and design tokens, scoping and isolation, and maintaining compatibility with the WordPress Global Styles and Site Editor cascade."
compatibility: "WordPress 6.0+ (theme.json v2/v3, Gutenberg block styling, core CSS custom properties). Applicable to standard CSS, Sass/SCSS, and CSS Modules."
---

# WordPress Styling Guidelines (wp-styling)

Writing CSS in WordPress differs fundamentally from generic web applications. The WordPress styling ecosystem is a shared runtime where WordPress core styles, block library styles, `theme.json` presets, theme author rules, user customizations via the Site Editor, and multiple third-party plugins coexist on the same DOM.

High specificity selectors and `!important` declarations break this cascade, disrupt user customization, and cause cascading specificity wars across themes and plugins.

Use this skill to design, write, and review stylesheets that integrate harmoniously with WordPress styling architecture.

## Upstream docs

Before making significant styling architectural decisions, consult the authoritative upstream guidelines:

1. WordPress CSS Coding Standards:
   `https://developer.wordpress.org/coding-standards/wordpress-coding-standards/css/`
2. Block Editor Handbook — Global Settings & Styles (`theme.json`):
   `https://developer.wordpress.org/block-editor/how-to-guides/themes/theme-json/`
3. Theme JSON Reference:
   `https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/`
4. Gutenberg repository — CSS Specificity in Gutenberg:
   `https://raw.githubusercontent.com/WordPress/gutenberg/trunk/docs/how-to-guides/themes/block-theme-overview.md`

## How to use this skill

Use this skill for:
- Writing or reviewing CSS/SCSS for WordPress themes and plugins.
- Refactoring legacy stylesheets to remove `!important` and reduce specificity.
- Consuming WordPress presets and design tokens (`--wp--preset--*`) instead of hardcoded values.
- Scoping plugin admin styles to avoid leaking into WordPress core admin UI or third-party plugins.
- Handling style isolation between the frontend and the Site Editor iframe.
- Styling custom Gutenberg blocks without breaking block support attributes (margins, paddings, colors).

Do not use this skill for:
- `theme.json` schema configuration syntax (use `wp-block-themes` instead).
- Tailwind / utility-first CSS framework installation workflows (unless configuring them to respect WordPress CSS variables).
- Pure JavaScript or PHP logic unrelated to styling.

## Core Rules

1. **Never use `!important` for layout, typography, or color rules.** `!important` permanently breaks the WordPress Global Styles hierarchy, preventing site owners and theme users from overriding styles via the Site Editor or Global Styles.
2. **Zero out base specificity with `:where()`.** Emulate Gutenberg core: wrap base block and component selectors in `:where(...)` so child themes, user customizations, and block variations can override them effortlessly with single-class selectors.
3. **Prefer Core CSS Custom Properties over hardcoded units.** Rely on `var(--wp--preset--*)` for colors, spacing, and typography to ensure designs adapt to theme color palettes and fluid typography settings.
4. **Namespace and scope plugin stylesheets.** Every plugin class must be prefixed with the plugin namespace (e.g., `.myplugin-*`). In WP Admin, never target un-prefixed elements (e.g. `input`, `button`, `.wrap`) without a parent scope selector.
5. **Respect CSS Cascade Layers (`@layer`) when supported.** Group base resets, component rules, and overrides into deliberate cascade layers.

## Reference topics

Read the detailed guides in the `references/` directory:

- [Specificity & Cascade Management](./references/specificity-and-cascade.md) — The WordPress style hierarchy, why `!important` is forbidden, `:where()` patterns, and cascade layer usage.
- [Design Tokens & CSS Variables](./references/design-tokens-and-variables.md) — Consuming `--wp--preset--*`, providing fallbacks, and declaring component tokens.
- [Scoping & Isolation](./references/scoping-and-isolation.md) — Admin style safety, frontend containment, CSS Modules, and Site Editor iframe considerations.

## Verification checklist

When reviewing or writing CSS in WordPress:
- [ ] Are there zero instances of `!important` (except for rare accessibility utility overrides like `.screen-reader-text`)?
- [ ] Are base element or component styles wrapped with `:where()` to minimize specificity?
- [ ] Are theme-dependent values (colors, spacing, font sizes) mapped to `var(--wp--preset--*, fallback)`?
- [ ] In plugin admin CSS, are all selectors scoped to a unique wrapper or namespace?
- [ ] Does the UI adapt properly when customizer or Site Editor global colors/typography are updated?
