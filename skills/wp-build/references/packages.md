# Packages

Use this file when creating or configuring packages in the `packages/` directory.

## Package structure

Each package is a subdirectory of `packages/` with its own `package.json`:

```
packages/
└── editor/
    ├── package.json
    └── src/
        ├── index.js
        └── style.scss
```

## Package package.json

```json
{
  "name": "@my-plugin/editor",
  "version": "1.0.0",
  "main": "src/index.js",
  "wpScript": true,
  "wpScriptModuleExports": "./build-module/index.mjs"
}
```

## Configuration fields

### wpScript

```json
{
  "wpScript": true
}
```

When `true`:
- Registered as WordPress script
- Available at `window.{scriptGlobal}.{packageName}`
- Handle: `{handlePrefix}-{packageName}`

When `false` or omitted:
- Package is only available as dependency for other packages
- Not exposed globally

### wpScriptModuleExports

String shorthand for the root export:

```json
{
  "wpScriptModuleExports": "./build-module/index.mjs"
}
```

Object form for multiple export paths:

```json
{
  "wpScriptModuleExports": {
    ".": "./build-module/index.mjs",
    "./utils": "./build-module/utils.mjs"
  }
}
```

Defines ES module entry points. Keys are export paths, values are file paths.

### wpWorkers

```json
{
  "wpWorkers": {
    "./my-worker": "./src/worker.js"
  }
}
```

Defines Web Worker entry points as self-contained bundles.

Object values can also be used when a worker needs module resolution redirects during bundling:

```json
{
  "wpWorkers": {
    "./my-worker": {
      "entry": "./src/worker.js",
      "resolve": {
        "vips-es6.js": "vips.js"
      }
    }
  }
}
```

## Auto-externalization

`@wordpress/build` automatically externalizes dependencies:

```javascript
// Your code
import { Button } from '@wordpress/components';
import { __ } from '@wordpress/i18n';

// Becomes
// window.wp.components.Button
// window.wp.i18n.__
```

Your own packages are also externalized:

```javascript
// packages/utils/src/index.js
export function formatDate(date) { ... }

// packages/editor/src/index.js
import { formatDate } from '@my-plugin/utils';
// → window.myPlugin.utils.formatDate
```

## Common package types

### Script package (runs in browser)

```json
{
  "name": "@my-plugin/editor",
  "wpScript": true
}
```

### Library package (dependency only)

```json
{
  "name": "@my-plugin/utils"
}
```

No `wpScript` — only used as import by other packages.

### Init module (page initialization)

```json
{
  "name": "@my-plugin/page-init",
  "wpScriptModuleExports": {
    ".": "./build-module/index.mjs"
  }
}
```

Must export `init()` function:

```typescript
export async function init() {
  // Runs before routes are registered
}
```

Upstream reference:

- https://developer.wordpress.org/block-editor/reference-guides/packages/packages-wp-build/
