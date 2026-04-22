---
name: wp-build
description: "Use when developing WordPress plugins with @wordpress/build: the next-generation esbuild-based build tool with convention-over-configuration approach, packages/ and routes/ directory structures, automatic PHP generation, and two page modes (fullscreen vs wp-admin)."
compatibility: "Targets WordPress 6.9+ (PHP 7.2.24+). Requires npm workspaces, Node.js 18+, and @wordpress/build package."
---

# WP Build (@wordpress/build)

## When to use

Use this skill for plugin development with `@wordpress/build`:

- setting up a new plugin with `@wordpress/build` (not `@wordpress/scripts`)
- organizing code in `packages/` directory with proper `package.json` configuration
- creating admin pages with `routes/` directory (fullscreen or wp-admin mode)
- configuring `wpPlugin` in root `package.json`
- registering page menus in PHP for either fullscreen or wp-admin mode
- troubleshooting page rendering issues (wrong mode, callback not found, etc.)

**Do NOT use this skill for:**
- Block development (use `wp-block-development` skill instead — `@wordpress/build` block support is still in progress)
- Projects using `@wordpress/scripts` (webpack-based)

## Inputs required

- Repo root and target plugin location.
- Desired page mode: fullscreen (Site Editor style) or wp-admin integrated.
- Plugin namespace and naming conventions (`wpPlugin.name`, `packageNamespace`, `handlePrefix`).
- Target WordPress + PHP versions.

## Procedure

### 0) Verify prerequisites

Before using `@wordpress/build`, ensure:

1. **npm workspaces** configured in root `package.json`
2. **@wordpress/\* packages** physically installed (not just peer deps)
3. **@babel/core** installed if using `@wordpress/components`

Run project triage if available:
```bash
node skills/wp-project-triage/scripts/detect_wp_project.mjs
```

### 1) Set up directory structure

Follow the convention strictly — directories are NOT configurable:

```
my-plugin/
├── packages/           # JavaScript packages (required)
│   └── {package-name}/
│       ├── package.json
│       └── src/
│           └── index.js
├── routes/             # Admin page routes (optional, experimental)
│   └── {route-name}/
│       ├── package.json    # Route config (required)
│       ├── stage.tsx       # Main content (required)
│       ├── inspector.tsx   # Sidebar (optional)
│       ├── canvas.tsx      # Custom canvas (optional)
│       └── route.tsx       # Lifecycle hooks (optional)
├── build/              # Auto-generated output
├── package.json        # Root config with wpPlugin
└── my-plugin.php       # Plugin main file
```

See:
- `references/directory-structure.md`

### 2) Configure root package.json

Add `wpPlugin` configuration block:

```json
{
  "name": "my-plugin",
  "wpPlugin": {
    "name": "my-plugin",
    "scriptGlobal": "myPlugin",
    "packageNamespace": "my-plugin",
    "handlePrefix": "my-plugin",
    "pages": ["my-admin-page"]
  },
  "scripts": {
    "build": "wp-build",
    "dev": "wp-build --watch"
  }
}
```

See:
- `references/configuration.md`

### 3) Configure packages

Each package in `packages/` needs its own `package.json`:

```json
{
  "name": "@my-plugin/editor",
  "main": "src/index.js",
  "wpScript": true,
  "wpScriptModuleExports": {
    ".": "./build-module/index.mjs"
  }
}
```

Key fields:
- `wpScript: true` — Bundle as IIFE, register as WordPress script
- `wpScriptModuleExports` — Expose as ES modules
- `wpWorkers` — Define Web Worker entries

See:
- `references/packages.md`

### 4) Configure routes (if using admin pages)

Each route in `routes/` needs a `package.json` with route config:

```json
{
  "route": {
    "path": "/",
    "page": "my-admin-page"
  }
}
```

The `page` field must match an entry in `wpPlugin.pages`.

**Route components:**
- `stage.tsx` — Main content (required): `export const stage = () => <div>...</div>`
- `inspector.tsx` — Sidebar (optional): `export const inspector = () => <div>...</div>`
- `canvas.tsx` — Custom canvas (optional)
- `route.tsx` — Lifecycle hooks: `beforeLoad`, `loader`, `canvas`

See:
- `references/routes.md`

### 5) Choose and register correct page mode

**CRITICAL: This is where most errors occur.**

`@wordpress/build` generates TWO page modes. You must choose the correct one:

| Mode | Generated File | Page Slug | Callback Function | Use Case |
|------|----------------|-----------|-------------------|----------|
| **Fullscreen** | `page.php` | `{page_id}` | `{plugin_name}_{page_id}_render_page` | Site Editor style apps |
| **WP-Admin** | `page-wp-admin.php` | `{page_id}-wp-admin` | `{plugin_name}_{page_id}_wp_admin_render_page` | Traditional settings pages |

**Fullscreen mode registration:**
```php
add_menu_page(
    __( 'My App', 'my-plugin' ),
    __( 'My App', 'my-plugin' ),
    'manage_options',
    'my-admin-page',                        // slug = page_id
    'my_plugin_my_admin_page_render_page',  // callback
    'dashicons-admin-generic',
    30
);
```

**WP-Admin mode registration:**
```php
add_submenu_page(
    'options-general.php',
    __( 'Settings', 'my-plugin' ),
    __( 'Settings', 'my-plugin' ),
    'manage_options',
    'my-admin-page-wp-admin',                        // slug = page_id + "-wp-admin"
    'my_plugin_my_admin_page_wp_admin_render_page'   // callback
);
```

See:
- `references/page-modes.md`

### 6) Integrate PHP

In the plugin main file, include the generated build file:

```php
<?php
/**
 * Plugin Name: My Plugin
 */

defined( 'ABSPATH' ) || exit;

require_once plugin_dir_path( __FILE__ ) . 'build/build.php';

// Register menu (see step 5 for correct mode)
add_action( 'admin_menu', 'my_plugin_admin_menu' );
```

See:
- `references/php-integration.md`

### 7) Build and verify

```bash
# Production build
npm run build

# Development with watch
npm run dev
```

The build tool will:
- Scan `packages/` and `routes/` directories
- Read each `package.json` configuration
- Compile and output to `build/`
- Generate PHP registration files automatically

## Verification

- [ ] `npm run build` completes without errors
- [ ] `build/build.php` exists and is included in plugin main file
- [ ] Admin menu appears in correct location
- [ ] Page renders without blank screen or console errors
- [ ] For fullscreen mode: WP admin sidebar is hidden
- [ ] For wp-admin mode: WP admin sidebar remains visible
- [ ] Route navigation works (if multiple routes)
- [ ] Scripts and styles are enqueued correctly

## Failure modes / debugging

### Page shows blank or 404

- **Wrong page slug**: Check if using `-wp-admin` suffix correctly
- **Wrong callback function**: Verify function name matches `{plugin_name}_{page_id}[_wp_admin]_render_page`
- **build/build.php not included**: Ensure `require_once` is present in plugin main file

### Fullscreen mode shows WP admin sidebar

- Using wp-admin mode callback instead of fullscreen callback
- Page slug has `-wp-admin` suffix when it shouldn't

### WP-Admin mode hides the sidebar (unwanted fullscreen)

- Using fullscreen callback instead of wp-admin callback
- Page slug missing `-wp-admin` suffix

### Route components not rendering

- Missing `export const stage = ...` (must be named export)
- Route `package.json` missing or malformed
- `page` field doesn't match `wpPlugin.pages` entry

### Scripts not loading

- Package `package.json` missing `wpScript: true`
- Package not in `packages/` directory
- Build not run after changes

### Build fails

- Missing npm workspaces configuration
- @wordpress/* packages not installed
- Syntax errors in source files

See:
- `references/debugging.md`

## Escalation

For canonical detail, consult:

- Official docs: https://developer.wordpress.org/block-editor/reference-guides/packages/packages-wp-build/
- GitHub source: https://github.com/WordPress/gutenberg/tree/trunk/packages/wp-build
- npm package: https://www.npmjs.com/package/@wordpress/build
- Vision issue: https://github.com/WordPress/gutenberg/issues/72032
