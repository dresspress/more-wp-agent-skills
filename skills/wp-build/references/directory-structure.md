# Directory structure

Use this file when setting up or verifying a `@wordpress/build` project structure.

## Core concepts

`@wordpress/build` uses **fixed directory conventions** that cannot be changed via configuration:

```
my-plugin/
├── packages/           # JavaScript packages
│   └── {name}/
│       ├── package.json
│       └── src/
│           └── index.js
├── routes/             # Admin page routes (experimental)
│   └── {name}/
│       ├── package.json
│       ├── stage.tsx
│       ├── inspector.tsx
│       ├── canvas.tsx
│       └── route.tsx
├── build/              # Auto-generated (do not edit)
│   ├── build.php
│   ├── modules.php
│   ├── scripts.php
│   ├── styles.php
│   ├── routes.php
│   ├── pages.php
│   ├── routes/
│   │   ├── {route-name}/
│   │   │   ├── content.js
│   │   │   └── route.js
│   │   └── registry.php
│   └── pages/
│       └── {page-id}/
│           ├── loader.js
│           ├── page.php
│           └── page-wp-admin.php
├── package.json        # Root config with wpPlugin
└── my-plugin.php       # Plugin entry point
```

## Directory roles

### packages/

Each subdirectory is a standalone JavaScript package:

- Must have its own `package.json`
- Source code in `src/` subdirectory
- Configured via `wpScript` and `wpScriptModuleExports` fields

### routes/

Each subdirectory is an admin page route:

- Must have `package.json` with `route` configuration
- Uses convention-based file names (stage.tsx, inspector.tsx, etc.)
- Lazy-loaded by default

### build/

Auto-generated output directory:

- **Never edit manually** — regenerated on each build
- Contains compiled JS, CSS, and PHP registration files
- Include `build/build.php` in your plugin

## Common mistakes

1. **Wrong directory name**: Must be exactly `packages/` and `routes/`, not `src/` or `app/`
2. **Missing package.json**: Each package/route needs its own `package.json`
3. **Source not in src/**: Package source must be in `src/` subdirectory
4. **Editing build/**: Changes will be overwritten

Upstream reference:

- https://developer.wordpress.org/block-editor/reference-guides/packages/packages-wp-build/
