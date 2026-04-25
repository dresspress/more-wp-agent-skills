---
name: wp-build
description: "Use when developing WordPress plugins with @wordpress/build: the next-generation esbuild-based build tool with convention-over-configuration approach, packages/ and routes/ directory structures, automatic PHP generation, and two page modes (fullscreen vs wp-admin)."
compatibility: "Targets WordPress 6.5+ (Requires Script Modules/Import Maps). Requires npm workspaces, Node.js 20+, and @wordpress/build package."
---

# WP Build (@wordpress/build)

## When to use

Use this skill for plugin development with `@wordpress/build`:

- setting up a new plugin with `@wordpress/build` (not `@wordpress/scripts`)
- organizing code in `packages/` directory with proper `package.json` configuration
- creating admin pages with `routes/` directory (fullscreen or wp-admin mode)
- configuring `wpPlugin` in root `package.json`
- registering page menus in PHP for either fullscreen or wp-admin mode
- implementing route lifecycle hooks (loader, beforeLoad, canvas, inspector)
- creating "Init Modules" for page-level initialization (icons, etc.)
- using `@wordpress/admin-ui` components for standard layout

**Do NOT use this skill for:**
- Block development (use `wp-block-development` skill instead)
- Projects using `@wordpress/scripts` (webpack-based)
- Old WordPress versions (< 6.5) that lack script module support

## Inputs required

- Repo root and target plugin location.
- Desired page mode: fullscreen (Site Editor style) or wp-admin integrated.
- Plugin namespace and naming conventions (`wpPlugin.name`, `packageNamespace`, `handlePrefix`).

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
├── routes/             # Admin page routes (optional)
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
    "name": "my-plugin",
    "scriptGlobal": "myPlugin",
    "packageNamespace": "my-plugin",
    "handlePrefix": "my-plugin",
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

### 4) Use Init Modules for Icons

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

### 5) Register correct page mode in PHP

**CRITICAL: Use the correct slug and callback pair.**

| Mode | Generated File | Page Slug | Callback Function |
|------|----------------|-----------|-------------------|
| **Fullscreen** | `page.php` | `{page_id}` | `{plugin}_{page_id}_render_page` |
| **WP-Admin** | `page-wp-admin.php` | `{page_id}-wp-admin` | `{plugin}_{page_id}_wp_admin_render_page` |

See `references/page-modes.md` for detailed comparison.

### 6) Integrate PHP

```php
require_once plugin_dir_path( __FILE__ ) . 'build/build.php';

add_action( 'admin_menu', function() {
    add_menu_page( 'Title', 'Menu', 'capability', 'my-page', 'my_prefix_my_page_render_page' );
} );
```

## Verification

- [ ] `npm run build` completes without errors
- [ ] `build/build.php` exists and is included
- [ ] Page renders without blank screen
- [ ] Navigation via `?p=/path` works
- [ ] Sidebar icons appear (if using init modules)
- [ ] UI follows WordPress standards (uses `admin-ui` package)

## Failure modes / debugging

- **Blank screen**: Check console for module loading errors; check PHP callback names.
- **Wrong layout**: Mixing Fullscreen callback with WP-Admin slug or vice versa.
- **Route 404**: Check `route.page` in route `package.json`.
- **Icons missing**: Init module not exported correctly or not registered in `wpPlugin.pages`.

## References

- `references/configuration.md` - `package.json` fields
- `references/directory-structure.md` - Where files go
- `references/packages.md` - Creating JS packages
- `references/routes.md` - Implementing routes and hooks
- `references/page-modes.md` - Fullscreen vs Integrated
- `references/php-integration.md` - PHP helper functions
- `references/debugging.md` - Troubleshooting guide
