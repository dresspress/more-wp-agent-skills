# Configuration

Use this file when configuring `@wordpress/build` in package.json files.

## Root package.json (wpPlugin)

The root `package.json` contains plugin-wide settings in the `wpPlugin` block:

```json
{
  "name": "my-awesome-plugin",
  "version": "1.0.0",
    "wpPlugin": {
      "name": "my-awesome-plugin",
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
        "simple-page",
      {
        "id": "complex-page",
        "init": ["@my-plugin/page-init"]
      }
    ]
  },
  "scripts": {
    "build": "wp-build",
    "dev": "wp-build --watch"
  },
  "devDependencies": {
    "@wordpress/build": "^0.13.0"
  }
}
```

## Field reference

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `name` | string | Prefix for generated PHP functions; must be a valid PHP function-name prefix | `"my_plugin"` |
| `scriptGlobal` | string \| false | Global variable name on `window`, or `false` to disable global exposure | `"myPlugin"` → `window.myPlugin` |
| `packageNamespace` | string | Package scope (without `@`) used for global exposure matching | `"my-plugin"` → `@my-plugin/*` |
| `handlePrefix` | string | WordPress script handle prefix; defaults to `packageNamespace` | `"my-plugin"` → `my-plugin-editor` |
| `externalNamespaces` | object | Map namespace keys to `{ global, handlePrefix? }` objects for cross-plugin externals | `{"woo":{"global":"woo","handlePrefix":"woocommerce"}}` |
| `pages` | array | Experimental admin pages supporting routes | See below |

## Package package.json fields

Each package in `packages/` can have these fields:

| Field | Type | Description |
|-------|------|-------------|
| `wpScript` | boolean | If `true`, bundles as a script with global exposure. |
| `wpScriptModuleExports` | string \| object | Script module export entry, either a string shorthand for `"."` or a map of export paths to file paths. |
| `wpScriptDefaultExport` | boolean | If `true`, wraps the default export. |
| `wpScriptExtraDependencies` | array | Extra WordPress script handles to depend on. |
| `wpStyleEntryPoints` | object/array | Custom SASS entry points. |
| `wpCopyFiles` | array | Files to copy, with optional `transform: "php"`. |
| `wpWorkers` | object | Web Worker entry points; supports string or object values. |

### Example: Advanced package config
```json
{
  "wpScript": true,
  "wpStyleEntryPoints": {
    "editor": "src/editor.scss",
    "view": "src/view.scss"
  },
  "wpCopyFiles": [
    { "from": "src/meta.php", "to": "build/meta.php", "transform": "php" }
  ]
}
```

## Pages configuration

Pages can be strings or objects:

```json
{
  "pages": [
    "simple-page",
    {
      "id": "page-with-init",
      "init": ["@my-plugin/page-init"]
    }
  ]
}
```

- **String format**: Simple page with no init modules
- **Object format**: Page with initialization modules
- **Status**: Experimental; expect API churn
- **Current-source fields**: object entries also support `title` and `experimental`, even though upstream README coverage is still thin

Init modules must export an `init()` function that runs before routes are registered.

## Compatibility notes

- `@wordpress/build` 0.13.0 declares `node >=20.10.0` and `npm >=10.2.3`.
- Script Modules and Import Maps require WordPress 6.5+.
- Since `@wordpress/build` 0.10.0, page/routing flows no longer bundle `@wordpress/boot`, `@wordpress/route`, `@wordpress/theme`, or `@wordpress/private-apis`; those must come from WordPress Core 7.0+ or the Gutenberg plugin.

## Naming conventions

Given this configuration:
```json
{
  "wpPlugin": {
    "name": "my-plugin",
    "scriptGlobal": "myPlugin",
    "packageNamespace": "my-plugin",
    "handlePrefix": "mp"
  }
}
```

Results in:
- Package `@my-plugin/editor` → `window.myPlugin.editor`
- Script handle: `mp-editor`
- PHP callback: `my_plugin_{page_id}_render_page`

## externalNamespaces structure

`externalNamespaces` does not take a bare string. Each namespace key maps to an object:

```json
{
  "wpPlugin": {
    "externalNamespaces": {
      "woo": {
        "global": "woo",
        "handlePrefix": "woocommerce"
      },
      "acme": {
        "global": "acme"
      }
    }
  }
}
```

- `global` is required and controls the browser global used for imports such as `@woo/cart` → `window.woo.cart`
- `handlePrefix` is optional; if omitted, it defaults to the namespace key

## npm workspaces (required)

Root `package.json` must configure workspaces:

```json
{
  "workspaces": [
    "packages/*"
  ]
}
```

Upstream reference:

- https://developer.wordpress.org/block-editor/reference-guides/packages/packages-wp-build/
