---
name: wp-styling
description: "Use when writing, refactoring, or reviewing CSS stylesheets in WordPress themes or plugins. Supplements official skills (wp-block-themes, wpds) by focusing specifically on specificity pitfalls, avoiding !important, :where() zero-specificity patterns, and admin style containment."
compatibility: "WordPress 5.8+ (Block themes, theme.json, and classic themes). Applicable to plain CSS, Sass/SCSS, and custom stylesheets."
---

# WordPress Styling Guardrails (wp-styling)

This skill is a **supplement** to the official WordPress agent skills:
- For `theme.json` configuration, presets, and Site Editor templates, use **`wp-block-themes`**.
- For WordPress Design System components and official design tokens, use **`wpds`**.

Use this skill strictly for **CSS authoring pitfalls, specificity rules, and style leakage guardrails** when writing stylesheets (`style.css`, plugin CSS, block stylesheets).

## When to use

Use this skill when:
- Writing or reviewing CSS for WordPress plugins, themes, or custom blocks.
- Refactoring stylesheets to eliminate `!important` and reduce specificity.
- Preventing plugin admin stylesheets from contaminating WordPress Core admin screens.
- Resolving conflicts between theme stylesheets and Site Editor Global Styles.

## Practical Guardrails

1. **Avoid `!important`**: Never use `!important` for colors, typography, spacing, or layout. It prevents users from customizing styles via `theme.json` and the Site Editor.
2. **Zero-out specificity with `:where()`**: Wrap base component selectors in `:where(...)` so child themes and user customizations can easily override them.
3. **Scope all admin stylesheets**: Never target un-scoped HTML tags (`input`, `select`, `button`) in `wp-admin`. Always scope under a unique root wrapper class.

## Reference topics

- [Specificity & Cascade Guardrails](./references/specificity-and-cascade.md) — Detailed rules on `!important`, `:where()` patterns, admin leakage prevention, and exceptions.
