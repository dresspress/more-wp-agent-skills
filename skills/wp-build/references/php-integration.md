# PHP Integration

Integrating `@wordpress/build` into your WordPress plugin involves including the generated build file and registering your admin menus.

## 1. Include the build file

In your main plugin file (e.g., `my-plugin.php`), include the generated `build/build.php`. This file contains all the registration logic for scripts, modules, styles, and pages.

```php
<?php
/**
 * Plugin Name: My Awesome Plugin
 */

defined( 'ABSPATH' ) || exit;

// Path to the generated build entry point
$my_plugin_build_path = plugin_dir_path( __FILE__ ) . 'build/build.php';

if ( file_exists( $my_plugin_build_path ) ) {
    require_once $my_plugin_build_path;
}
```

## 2. Register Admin Menus

Use the `admin_menu` hook to register your pages. `@wordpress/build` generates the callback functions for you.

The function names are derived from:
`{wpPlugin.name}_{page_id}[_wp_admin]_render_page`

```php
add_action( 'admin_menu', function() {
    // Fullscreen Page
    add_menu_page(
        __( 'My App', 'my-plugin' ),
        __( 'My App', 'my-plugin' ),
        'manage_options',
        'my-app-page',
        'my_plugin_my_app_page_render_page'
    );

    // Integrated Settings Page
    add_submenu_page(
        'options-general.php',
        __( 'My Settings', 'my-plugin' ),
        __( 'My Settings', 'my-plugin' ),
        'manage_options',
        'my-app-page-wp-admin',
        'my_plugin_my_app_page_wp_admin_render_page'
    );
} );
```

## 3. Registering Routes and Menu Items (PHP Side)

For each page defined in `wpPlugin.pages`, the build tool generates helper functions to register routes and menu items from PHP. These are useful for dynamic routes or when extensions want to add their own routes.

### Route Registration
`{prefix}_register_{page_id}_route( $path, $content_module, $route_module )`

```php
// Register a route for 'my-app-page'
my_plugin_register_my_app_page_route(
    '/custom-path',
    '@my-plugin/custom-content-module'
);
```

### Menu Item Registration
`{prefix}_register_{page_id}_menu_item( $id, $label, $to, $parent_id, $parent_type )`

```php
// Add an item to the app sidebar
my_plugin_register_my_app_page_menu_item(
    'custom-item',
    __( 'Custom Action', 'my-plugin' ),
    '/custom-path'
);
```

## 4. Hooking into Page Initialization

Each page fires an action hook when it initializes:
`do_action( "{$page_id}_init" )` or `do_action( "{$page_id}-wp-admin_init" )`.

This is the perfect place to register dynamic routes or menu items.

```php
add_action( 'my-app-page_init', function() {
    my_plugin_register_my_app_page_route( '/extra', '@my-plugin/extra' );
} );
```

## 5. Summary of generated PHP files in `build/`

- `build.php`: Main entry point (include this!).
- `constants.php`: Build-time constants (URLs, paths).
- `modules.php`: Script module registration.
- `scripts.php`: Classic script registration.
- `styles.php`: Style registration.
- `pages.php`: Loader for all generated page files.
- `pages/{id}/page.php`: Fullscreen mode logic for a specific page.
- `pages/{id}/page-wp-admin.php`: WP-Admin mode logic for a specific page.
