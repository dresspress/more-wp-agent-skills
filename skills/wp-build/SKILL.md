---
name: wp-build
description: "Use when developing WordPress plugins with @wordpress/build: the next-generation esbuild-based build tool with convention-over-configuration approach, `packages/` plus experimental `routes/`, automatic PHP generation, worker bundling, and two page modes (fullscreen vs wp-admin)."
compatibility: "Requires modern Node/npm (`@wordpress/build` 0.13.0 declares Node >=20.10 and npm >=10.2). Script Modules/Import Maps need WordPress 6.5+, but page/routing workflows now assume `@wordpress/boot`, `@wordpress/route`, and `@wordpress/theme` are provided by WordPress Core 7.0+ or by the Gutenberg plugin because `@wordpress/build` no longer bundles them. `routes/` and `wpPlugin.pages` remain experimental."
---

# WP Build (@wordpress/build)

## When to use

Use this skill for plugin development with `@wordpress/build`:

- setting up a new plugin with `@wordpress/build` (not `@wordpress/scripts`)
- organizing code in `packages/` directory with proper `package.json` configuration
- creating admin pages with experimental `routes/` directory (fullscreen or wp-admin mode)
- creating experimental `widgets/` entries when a plugin needs self-contained script-module widgets
- designing fullscreen SPA sidebar navigation/menu hierarchies with PHP-first registration
- configuring `wpPlugin` in root `package.json`
- registering page menus in PHP for either fullscreen or wp-admin mode
- implementing route lifecycle hooks (loader, beforeLoad, canvas, inspector)
- creating "Init Modules" for page-level initialization (icons, etc.)
- using `@wordpress/admin-ui` components for standard layout

**Do NOT use this skill for:**
- Block development (use `wp-block-development` skill instead)
- Projects using `@wordpress/scripts` (webpack-based)
- Old WordPress versions (< 6.5) that lack script module support
- Stable block-plugin workflows; `@wordpress/build` still has gaps there and may require manual workarounds

## Inputs required

- Repo root and target plugin location.
- Desired page mode: fullscreen (Site Editor style) or wp-admin integrated.
- Plugin namespace and naming conventions (`wpPlugin.name`, `packageNamespace`, `handlePrefix`).
- Whether the plugin can accept experimental routing/page APIs.
- Target runtime: WordPress Core 7.0+ vs Gutenberg plugin, because page/routing features depend on host-provided boot/route/theme packages.
- Desired sidebar information architecture for fullscreen pages: top-level routes, parent groups, and whether grouped children should use `dropdown` or `drilldown`.
- Whether the plugin needs experimental widget discovery under `widgets/`.

## Procedure

### 1) Set up directory structure

Follow the convention strictly — directories are NOT configurable:

```
my-plugin/
├── packages/           # JavaScript packages (required)
│   └── {package-name}/
│       ├── package.json
│       └── src/
│           └── index.js
├── routes/             # Admin page routes (experimental, optional)
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

### 2) Configure root package.json

Add `wpPlugin` configuration block:

```json
{
  "name": "my-plugin",
  "wpPlugin": {
    "name": "my_plugin",
    "scriptGlobal": "myPlugin",
    "packageNamespace": "my-plugin",
    "handlePrefix": "my-plugin",
    "externalNamespaces": {
      "woo": {
        "global": "woo",
        "handlePrefix": "woocommerce"
      }
    },
    "pages": [
      "my-admin-page",
      {
        "id": "settings-page",
        "init": ["@my-plugin/settings-init"]
      }
    ]
  },
  "workspaces": ["packages/*"],
  "scripts": {
    "build": "wp-build",
    "dev": "wp-build --watch"
  }
}
```

Notes:
- `wpPlugin.name` is **not** the npm package name. It must be a valid PHP function-name prefix, so do not use hyphens there.
- `wpPlugin.pages` object entries may also carry `title` and `experimental` fields in current upstream source.
- `@wordpress/build` 0.13.0 itself is still pre-1.0; avoid assuming older examples from early blog posts still match generated file names or runtime behavior.

### 3) Implement Routes with standard UI

In `stage.tsx`, prefer using `@wordpress/admin-ui` components:

```tsx
import { Page, Breadcrumbs } from '@wordpress/admin-ui';
import { __ } from '@wordpress/i18n';

export const stage = () => (
  <Page 
    title={ __('My Page', 'my-plugin') }
    breadcrumbs={ 
      <Breadcrumbs items={[
        { label: __('Home', 'my-plugin'), to: '/' },
        { label: __('Settings', 'my-plugin') }
      ]} /> 
    }
  >
    <div>Main Content</div>
  </Page>
);
```

### 4) Use Init Modules for Icons in fullscreen pages

Icons and command palette entries should be registered in an init module:

```typescript
// packages/settings-init/src/index.ts
import { settings } from '@wordpress/icons';
import { dispatch } from '@wordpress/data';
import { store as bootStore } from '@wordpress/boot';

export async function init() {
    // Add icon to the sidebar menu item
    dispatch( bootStore ).updateMenuItem( 'settings-page', { icon: settings } );
}
```

The named `init()` export is mandatory. Init modules are loaded as static dependencies and executed sequentially before the boot system registers menu items and routes.

This pattern is for fullscreen pages. Current `page-wp-admin.php` uses `@wordpress/boot.initSinglePage()` and does not pass `menuItems` or `initModules`, so sidebar icon/menu customization should not be described as a WP-Admin-mode capability.

### 5) Treat fullscreen sidebar navigation as PHP-first architecture

For fullscreen pages, define menu structure, nesting, and order in PHP first, then use JS only for icon mapping or light metadata updates.

Use the generated helper function from `build/build.php`:

```php
{prefix}_register_{page_slug}_menu_item( $id, $label, $to, $parent_id = '', $parent_type = '' );
```

Rules:
- Register top-level items in PHP call order; that order becomes the rendered sidebar order.
- Use real route paths for navigable items such as `/`, `/analytics`, or `/settings/general`.
- For parent groups, set `$to` to an empty string and choose `parent_type` deliberately:
  - `dropdown` is the default and recommended mode for settings/data-management groups.
  - `drilldown` is for immersive sub-sections that should open in a dedicated sliding panel.
- Pass the parent item ID as `$parent_id` when registering children.
- Do not try to build or reorder the menu tree in JS with `dispatch( bootStore ).updateMenuItem()`. Use JS to attach icons, not to define the authoritative hierarchy.

See `references/navigation.md` for examples and detailed guidance.

### 6) Register correct page mode in PHP

**CRITICAL: Use the correct slug and callback pair.**

| Mode | Generated File | Page Slug | Callback Function |
|------|----------------|-----------|-------------------|
| **Fullscreen** | `page.php` | `{page_id}` | `{plugin}_{page_id}_render_page` |
| **WP-Admin** | `page-wp-admin.php` | `{page_id}-wp-admin` | `{plugin}_{page_id}_wp_admin_render_page` |

See `references/page-modes.md` for detailed comparison.

`wpPlugin.pages` and `routes/` are still experimental. Prefer them when the plugin owns the page experience and can tolerate upstream API churn.

### 7) Integrate PHP

```php
require_once plugin_dir_path( __FILE__ ) . 'build/build.php';

add_action( 'admin_menu', function() {
    add_menu_page( 'Title', 'Menu', 'capability', 'my-page', 'my_prefix_my_page_render_page' );
} );
```

### 8) Account for build outputs and experimental widgets

`@wordpress/build` emits both CommonJS-oriented `build/` output and ESM-oriented `build-module/` output. Do not describe the tool as `build/`-only.

If the plugin uses experimental widgets:
- Create a root-level `widgets/` directory with one subdirectory per widget.
- Treat widget discovery as experimental and verify generated registration files against the exact host/runtime.
- **Widget-to-page wiring is no longer handled by wp-build.** Surface packages use the `{page_id}_script_module_dependencies` PHP filter to inject widget modules into a page. Do not assume widgets auto-appear on any page.
- Widget types are registered server-side via a PHP registry and exposed through a REST endpoint; check whether the target host version includes this.
- See `references/widgets.md` before defining widget metadata or render entries.

## Verification

- [ ] `npm run build` completes without errors
- [ ] `build/build.php` exists and is included
- [ ] Generated outputs match the expected `build/` and `build-module/` split
- [ ] Page renders without blank screen
- [ ] Navigation via `?p=/path` works
- [ ] Fullscreen menu order and parent/child hierarchy come entirely from PHP registration
- [ ] Sidebar icons appear (fullscreen mode only, if using init modules)
- [ ] UI follows WordPress standards (uses `admin-ui` package when appropriate)
- [ ] Experimental assumptions are still valid against the current upstream docs

## Failure modes / debugging

- **Blank screen**: Check console for module loading errors; check PHP callback names.
- **Generated PHP callbacks have unexpected names**: Check `wpPlugin.name`; it must be a valid PHP function prefix, not a kebab-case slug.
- **Wrong layout**: Mixing Fullscreen callback with WP-Admin slug or vice versa.
- **Route 404**: Check `route.page` in route `package.json`.
- **Menu hierarchy looks wrong**: Check PHP registration order, `parent_id`, and `parent_type` before debugging JS.
- **Icons missing**: Init module not exported correctly or not registered in `wpPlugin.pages`, or you are testing in WP-Admin mode where `initModules`/boot menu items are not wired in.
- **Init modules do not run**: Confirm each declared module exports a named async or sync `init()` function.
- **Menu order flashes or mutates in JS**: Remove client-side tree construction; PHP should emit the final ordered hierarchy.
- **Page features fail on older hosts**: Since `@wordpress/build` 0.10.0, page/routing flows expect `@wordpress/boot`, `@wordpress/route`, and `@wordpress/theme` from Core 7.0+ or the Gutenberg plugin.
- **Standalone plugin setup friction**: Confirm npm workspaces, installed `@wordpress/*` metadata, and any hidden transitive requirements noted upstream.

## References

- `references/configuration.md` - `package.json` fields
- `references/directory-structure.md` - Where files go
- `references/packages.md` - Creating JS packages
- `references/routes.md` - Implementing routes and hooks
- `references/navigation.md` - Fullscreen SPA navigation/menu architecture
- `references/page-modes.md` - Fullscreen vs Integrated
- `references/php-integration.md` - PHP helper functions
- `references/widgets.md` - Experimental widget discovery and output
- `references/debugging.md` - Troubleshooting guide
