# Specificity & Cascade Management in WordPress

## The Fundamental Problem with `!important` in WordPress

In standard standalone web development, using `!important` is an antipattern because it disrupts the normal cascade. In WordPress, **it is destructive to the entire platform's extensibility model**.

WordPress 5.8+ introduced Full Site Editing (FSE) and `theme.json`, establishing a strict hierarchy for resolving visual styles:

```
[Level 1: Core Defaults] (Gutenberg block library styles, reset CSS)
         ↓
[Level 2: theme.json] (Theme presets, element styles, block styles)
         ↓
[Level 3: Theme Stylesheet] (style.css, custom theme CSS)
         ↓
[Level 4: Child Theme Styles] (child style.css)
         ↓
[Level 5: Global Styles / Site Editor] (User customizations stored in database / wp_global_styles)
         ↓
[Level 6: Block Attribute Overrides] (Inline styles applied by user directly in post editor)
```

### Why `!important` breaks this:
When a theme or plugin author uses `!important`:
1. **User customizations are ignored**: A site owner attempting to change background color or font size in Site Editor cannot override an `!important` rule defined in a plugin or theme without custom CSS hacks.
2. **Third-party integrations break**: Block variations, extensions, and child themes cannot adapt or restyle components cleanly.
3. **Escalation wars**: Other plugins and developers are forced to write `!important` with higher specificity, creating an unmaintainable codebase.

---

## The Rule: Avoid `!important`

**Default policy**: Do not use `!important` for styling layout, spacing, typography, borders, or colors.

### The Only Valid Exceptions:
1. **Accessibility utility classes**: Utilities like `.screen-reader-text` or `.is-hidden` where visibility state must be absolute and never overridden by adjacent component styling:
   ```css
   .screen-reader-text {
       border: 0 !important;
       clip: rect(1px, 1px, 1px, 1px) !important;
       clip-path: inset(50%) !important;
       height: 1px !important;
       margin: -1px !important;
       overflow: hidden !important;
       padding: 0 !important;
       position: absolute !important;
       width: 1px !important;
       word-wrap: normal !important;
   }
   ```
2. **Overriding third-party un-scoped inline styles**: When interacting with legacy third-party plugins that inject hardcoded `style="..."` attributes on the DOM and offer no filter hooks.

---

## Strategy 1: Lower Specificity with `:where()`

WordPress Core block styles (since WP 5.9) wrap selectors in `:where()` to drop specificity to **(0, 0, 0)**. Follow this exact pattern in your themes and plugins.

### Anti-Pattern (High Specificity):
```css
/* Specificity: (0, 2, 1) — Hard to override without matching or exceeding */
.entry-content .my-plugin-card .my-plugin-card__title {
    font-size: 1.5rem;
    color: #111;
}
```

### Preferred Pattern (Zero Specificity):
```css
/* Specificity: (0, 0, 0) — Any single class can override this without friction */
:where(.my-plugin-card) :where(.my-plugin-card__title) {
    font-size: 1.5rem;
    color: #111;
}
```

By wrapping selectors in `:where()`, you provide sensible default styles that any user rule, `theme.json` setting, or block support attribute can easily override without specificity escalation.

---

## Strategy 2: Single-Class BEM Selectors

When `:where()` is not needed for a baseline reset, keep selector specificity strictly flat at **(0, 1, 0)** using flat BEM naming:

```css
/* Good: Single class (0, 1, 0) */
.dresspress-card { ... }
.dresspress-card__header { ... }
.dresspress-card__title { ... }
.dresspress-card--featured { ... }

/* Bad: Nested descendant selectors (0, 3, 0) */
.dresspress-container .dresspress-card .dresspress-card__title { ... }
```

Avoid qualifying class names with element tags:
- **Avoid**: `h2.dresspress-card__title` (0, 1, 1)
- **Use**: `.dresspress-card__title` (0, 1, 0)

---

## Strategy 3: CSS Cascade Layers (`@layer`)

Modern browsers (and modern WordPress themes/builds) support CSS Cascade Layers. Layers allow you to explicitly define the order of precedence regardless of selector specificity.

```css
/* Declare layer priority in your entry stylesheet */
@layer reset, base, components, utilities;

@layer base {
    /* Rules here will always yield to @layer components, even with higher specificity */
    h1, h2, h3 {
        margin-block: 1rem;
        line-height: 1.2;
    }
}

@layer components {
    .my-card {
        padding: 1.5rem;
    }
}
```

Styles outside any layer have the highest precedence, making it easy for users or child themes to override layered rules without `!important`.

---

## How to Fix Legacy Code Using `!important`

When refactoring legacy stylesheets containing `!important`:

1. **Identify the collision**: Why was `!important` added? Usually to beat a theme reset, an admin table style, or a core block style.
2. **Inspect the competing selector in DevTools**: Check the exact specificity of the conflicting rule.
3. **Use source order or equal specificity**:
   - If the conflict has specificity `(0, 1, 0)`, ensure your stylesheet is enqueued after the conflicting one (`wp_enqueue_style` dependencies array).
4. **Use duplicate class trick if temporary specificity boost is unavoidable**:
   If you must overcome a stubborn `(0, 2, 0)` rule without modifying HTML and cannot use `!important`, use:
   ```css
   /* Specificity: (0, 2, 0) instead of using !important */
   .my-component.my-component {
       color: blue;
   }
   ```
   *Note: This is still a compromise, but far safer than `!important` because it does not escape the normal specificity cascade.*
