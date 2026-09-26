# CSS Specificity & Cascade Guardrails in WordPress

This reference supplements official WordPress skills (`wp-block-themes`, `wpds`). It covers practical CSS specificity pitfalls and gotchas that frequently break WordPress themes and plugins.

---

## 1. Never Use `!important` in Custom Styles

### The Problem
In modern WordPress (Block Themes and classic themes with `theme.json`), styles follow a strict cascade:
`Core Defaults` → `theme.json` → `Theme CSS` → `Site Editor (User Customizations)` → `Block Inline Overrides`.

When a plugin or theme uses `!important`:
- **Site Editor styles are ignored**: User color/font/spacing customizations cannot override `!important` without database hacks or custom CSS plugins.
- **Child themes and block variations cannot adapt**: Elements become locked against normal CSS cascading.

### The Guardrail
Do not use `!important` for typography, colors, layout, or spacing.

### The Only Valid Exceptions
- **Accessibility utilities**: Absolute visibility classes like `.screen-reader-text` where hiding is mandatory.
- **Un-hookable 3rd-party inline styles**: Overriding hardcoded `style="..."` injected by legacy third-party plugins where no PHP filter exists.

---

## 2. Zero-out Default Specificity with `:where()`

Emulate WordPress Core: Core block styles wrap selectors in `:where()` to drop specificity to `(0, 0, 0)`.

```css
/* Anti-pattern: High specificity (0, 2, 0) - hard for themes/users to override */
.my-plugin-box .my-plugin-title {
    color: #111;
}

/* Recommended: Zero specificity (0, 0, 0) - allows effortless overrides */
:where(.my-plugin-box) :where(.my-plugin-title) {
    color: #111;
}
```

---

## 3. Prevent Admin CSS Leakage (wp-admin)

In WordPress admin, all plugins share the global DOM. A common anti-pattern is styling bare tags:

```css
/* Dangerous Anti-pattern: Corrupts WP core admin UI & other plugins */
input[type="text"], select, .button {
    padding: 10px;
    border-radius: 8px;
}

/* Recommended: Strictly scope all admin CSS to the page or wrapper container */
.myplugin-admin-wrap input[type="text"],
.myplugin-admin-wrap select {
    padding: 10px;
}
```
