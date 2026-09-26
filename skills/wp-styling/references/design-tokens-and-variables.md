# Design Tokens & CSS Custom Properties in WordPress

## WordPress Preset Tokens Architecture

WordPress generates CSS Custom Properties at `:root` (and within block containers) based on `theme.json` configuration and Core presets. Using these tokens ensures theme consistency and enables user customization without hardcoded values.

### Core Token Namespaces

| Token Namespace | Purpose | Example |
| :--- | :--- | :--- |
| `--wp--preset--color--<slug>` | Palette colors | `var(--wp--preset--color--primary)` |
| `--wp--preset--spacing--<slug>` | Spacing steps (margins, paddings, gaps) | `var(--wp--preset--spacing--30)` |
| `--wp--preset--font-size--<slug>` | Typography scale (often fluid) | `var(--wp--preset--font-size--medium)` |
| `--wp--preset--font-family--<slug>` | Registered font families | `var(--wp--preset--font-family--body)` |
| `--wp--preset--shadow--<slug>` | Box shadows | `var(--wp--preset--shadow--natural)` |

---

## Best Practices for Consuming Tokens

### 1. Always Provide Meaningful Fallbacks
Never assume a specific theme preset slug exists. While core presets like `small`, `medium`, and `large` exist on standard block themes, custom themes may rename or omit them:

```css
/* Bad: If theme does not define 'accent', property resolves to invalid/transparent */
.my-plugin-badge {
    background-color: var(--wp--preset--color--accent);
}

/* Good: Graceful degradation when preset is absent */
.my-plugin-badge {
    background-color: var(--wp--preset--color--accent, #2563eb);
}
```

### 2. Map Core Tokens to Component-Scoped Variables
Declare component-level CSS variables on your root component class, then use those scoped variables internally. This allows developers and themes to customize your entire component from a single root selector:

```css
:where(.dresspress-pricing-table) {
    /* Component API tokens */
    --dp-pt-bg: var(--wp--preset--color--base, #ffffff);
    --dp-pt-text: var(--wp--preset--color--contrast, #1e293b);
    --dp-pt-accent: var(--wp--preset--color--primary, #3b82f6);
    --dp-pt-padding: var(--wp--preset--spacing--40, 1.5rem);
    --dp-pt-radius: 8px;

    /* Applied internal styles */
    background-color: var(--dp-pt-bg);
    color: var(--dp-pt-text);
    padding: var(--dp-pt-padding);
    border-radius: var(--dp-pt-radius);
}

.dresspress-pricing-table__cta {
    background-color: var(--dp-pt-accent);
}
```

If another developer wants to restyle this pricing table in their theme, they only need to override the variables:
```css
.my-custom-theme .dresspress-pricing-table {
    --dp-pt-accent: #10b981;
    --dp-pt-radius: 16px;
}
```

---

## Fluid Typography and Spacing

Since WordPress 6.1, block themes frequently enable **Fluid Typography** and **Fluid Spacing**. This means preset variables do not output static pixels or rems, but dynamic `clamp()` expressions:

```css
/* Example of what WordPress outputs under the hood: */
--wp--preset--font-size--large: clamp(1.5rem, 1.5rem + ((1vw - 0.48rem) * 1.5), 2.5rem);
```

### Rules:
- **Do not wrap preset font sizes in arithmetic operations** (e.g. `calc(var(--wp--preset--font-size--large) * 1.2)`) unless you verify they are not complex clamp expressions, as nested calculations may cause unexpected clamp bounds.
- Use spacing presets (`--wp--preset--spacing--*`) directly for `gap`, `padding`, and `margin`.
