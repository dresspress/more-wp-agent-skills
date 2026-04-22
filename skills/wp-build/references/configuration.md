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
      "@other-plugin": "otherPlugin"
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
    "@wordpress/build": "^1.0.0"
  }
}
```

## Field reference

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `name` | string | Plugin identifier | `"my-awesome-plugin"` |
| `scriptGlobal` | string | Global variable name on window | `"myPlugin"` → `window.myPlugin` |
| `packageNamespace` | string | Package scope (without @) | `"my-plugin"` → `@my-plugin/*` |
| `handlePrefix` | string | WordPress script handle prefix | `"my-plugin"` → `my-plugin-editor` |
| `externalNamespaces` | object | Map external plugin namespaces to globals | `{"@other": "other"}` |
| `pages` | array | Admin pages supporting routes | See below |

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

Init modules must export an `init()` function that runs before routes are registered.

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
