# Page Modes

`@wordpress/build` generates two distinct modes for each page defined in `wpPlugin.pages`. Choosing the correct mode depends on whether you want a full-screen experience (like the Site Editor) or an integrated WP-Admin experience (like traditional settings pages).

## Mode Comparison

| Feature | Fullscreen Mode | WP-Admin Mode |
|---------|-----------------|---------------|
| **Page Slug** | `{page_id}` | `{page_id}-wp-admin` |
| **Callback** | `{plugin}_{page_id}_render_page` | `{plugin}_{page_id}_wp_admin_render_page` |
| **PHP File** | `build/pages/{page_id}/page.php` | `build/pages/{page_id}/page-wp-admin.php` |
| **WP Sidebar** | Hidden (Takes over screen) | Visible (Standard sidebar) |
| **Header** | Custom (from JS) | Standard WP Header |
| **JS Init** | `@wordpress/boot.init()` | `@wordpress/boot.initSinglePage()` |
| **Best For** | Immersive apps, Editors | Settings, Dashboards, CRUD lists |

## 1. Fullscreen Mode

This mode is designed for "app-like" experiences. It intercepts `admin_init` and renders a clean HTML skeleton, bypassing the standard WordPress admin header and footer.

### PHP Registration

```php
add_menu_page(
    __( 'My App', 'my-plugin' ),
    __( 'My App', 'my-plugin' ),
    'manage_options',
    'my-admin-page',                        // Slug matches page_id
    'my_plugin_my_admin_page_render_page',  // Callback
    'dashicons-admin-generic',
    30
);
```

### Characteristics
- Hides the WordPress admin bar (`#wpadminbar { display: none; }`).
- Sets `height: 100vh` on the app container.
- Uses the full-page boot flow from `@wordpress/boot`.

## 2. WP-Admin Mode

This mode integrates within the existing WordPress admin structure. It keeps the WordPress sidebar and admin bar, rendering the app inside the `#wpcontent` area.

### PHP Registration

```php
add_submenu_page(
    'options-general.php',
    __( 'Settings', 'my-plugin' ),
    __( 'Settings', 'my-plugin' ),
    'manage_options',
    'my-admin-page-wp-admin',                        // Slug matches page_id + "-wp-admin"
    'my_plugin_my_admin_page_wp_admin_render_page'   // Callback
);
```

### Characteristics
- Resets standard `#wpcontent` padding to allow the app to fill the space.
- Hides legacy admin elements (like notices) that might interfere with the UI.
- Uses the single-page boot flow from `@wordpress/boot`.
- Does not wire fullscreen-style `menuItems` or `initModules`; route rendering works, but sidebar/menu icon customization is not the same capability surface as full-page mode.

The exact `@wordpress/boot` internals are evolving faster than its published package reference. Keep examples focused on observable page behavior, not internal method guarantees, unless you have confirmed them against the current source.

## Critical Implementation Rule

**Do not mix the slug and the callback.**

If you use the Fullscreen callback (`_render_page`) with the WP-Admin slug (`-wp-admin`), the page will fail to render correctly because the interceptor won't trigger or will trigger at the wrong time.

| Desired UI | Menu Slug | Callback Name |
|------------|-----------|---------------|
| **Take over screen** | `my-page` | `my_plugin_my_page_render_page` |
| **Stay inside WP** | `my-page-wp-admin` | `my_plugin_my_page_wp_admin_render_page` |

## Navigation behavior

Both modes use the `p` query parameter for internal routing:
- Fullscreen: `admin.php?page=my-page&p=/settings`
- WP-Admin: `admin.php?page=my-page-wp-admin&p=/settings`

The JavaScript boot system automatically detects this parameter and navigates the internal router.
