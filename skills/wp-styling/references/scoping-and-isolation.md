# Scoping, Isolation & Environment Differences in WordPress

## The Danger of Unscoped Styles in WordPress

Unlike siloed Single Page Applications, WordPress plugins share the DOM with WordPress core, themes, and dozens of other plugins. Unscoped CSS leads to two common disasters:
1. **Admin Bleed**: Plugin CSS inadvertently changes core admin styles (e.g. styling all buttons, inputs, tables, or header elements in `wp-admin`).
2. **Frontend Collisions**: Plugin CSS overrides theme-level typography or container layouts.

---

## 1. WordPress Admin Style Scoping

### Strict Rule: Never Target Bare HTML Elements in Admin CSS
In `wp-admin`, never write un-scoped element selectors:

```css
/* DANGEROUS: Leaks into all WP Admin screens, breaks WP core forms & other plugins */
input[type="text"], select, textarea {
    border-radius: 8px;
    border: 1px solid #cbd5e1;
    padding: 10px;
}

button.button {
    background: #4f46e5;
}
```

### Required Pattern: Root Wrapper Scoping
Wrap your plugin's admin UI in a unique top-level container (e.g., `<div class="dresspress-settings-wrap wrap">`), and scope every rule under it:

```css
/* Safe: Restricted exclusively to the plugin's settings page */
.dresspress-settings-wrap input[type="text"],
.dresspress-settings-wrap select,
.dresspress-settings-wrap textarea {
    border-radius: 8px;
    border: 1px solid #cbd5e1;
}

.dresspress-settings-wrap .button.dresspress-button--primary {
    background: #4f46e5;
    color: #ffffff;
}
```

### Admin Enqueue Targeting
Only enqueue admin stylesheets on the specific admin hook/page where they are needed:

```php
function myplugin_enqueue_admin_styles( string $hook_suffix ): void {
    // Only load on the specific plugin settings page
    if ( 'toplevel_page_myplugin-settings' !== $hook_suffix ) {
        return;
    }

    wp_enqueue_style(
        'myplugin-admin-css',
        plugins_url( 'build/admin.css', __FILE__ ),
        [],
        MYPLUGIN_VERSION
    );
}
add_action( 'admin_enqueue_scripts', 'myplugin_enqueue_admin_styles' );
```

---

## 2. Block Editor (Iframe) vs Frontend Isolation

In modern WordPress (WP 6.3+), the Block Editor renders block content inside an isolated `<iframe>` (`iframe[name="editor-canvas"]`).

### Key Impacts:
- **Global `wp-admin` styles do not enter the editor iframe**: Do not rely on WordPress admin stylesheets to style blocks inside the editor canvas.
- **Editor Styles Enqueueing**: Register editor styles using `add_editor_style()` or via `block.json`:
  ```json
  {
      "name": "myplugin/card",
      "style": "file:./style-index.css",
      "editorStyle": "file:./index.css"
  }
  ```
- **The `.editor-styles-wrapper` class**: For non-iframed editor contexts (legacy or custom meta boxes), WordPress prefixes editor stylesheets with `.editor-styles-wrapper`. Avoid hardcoding `.editor-styles-wrapper` in your SCSS/CSS directly; let build tools (`@wordpress/scripts`) handle prefixing automatically.

---

## 3. CSS Modules for Component Isolation

When building React-based admin interfaces or complex custom blocks using `@wordpress/scripts` or `@wordpress/build`, prefer **CSS Modules** (`*.module.css` / `*.module.scss`).

### Benefits:
- Generates locally scoped, unique class names automatically (e.g., `card_container__x7y2z`).
- Zero chance of collision with WordPress core or other plugins.
- Eliminates the need for manual BEM prefix chains in React components.

### Example:
```jsx
// src/components/SettingsCard.jsx
import styles from './SettingsCard.module.css';

export function SettingsCard({ title, children }) {
    return (
        <div className={styles.card}>
            <h2 className={styles.title}>{title}</h2>
            <div className={styles.body}>{children}</div>
        </div>
    );
}
```

```css
/* src/components/SettingsCard.module.css */
.card {
    background: #ffffff;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    padding: 1.5rem;
}

.title {
    font-size: 1.25rem;
    margin-bottom: 1rem;
}
```
