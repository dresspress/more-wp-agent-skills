# Widgets

Use this reference when a plugin needs experimental widget discovery under root `widgets/`.

Widgets are still experimental. The architecture is actively evolving — widget-to-page wiring was recently **decoupled from wp-build** and moved to surface packages. Treat the directory shape, generated files, and host integration points as subject to change.

## Directory shape

```text
widgets/
└── hello-world/
    ├── widget.json     # Static discovery metadata (required)
    ├── widget.ts       # Runtime schema entry (optional)
    └── render.tsx      # UI render entry (optional)
    └── render.scss     # Optional styles (bundled inline when imported)
```

## Responsibility split

- `widget.json`: static metadata — `name`, `title`, `description`, `category`
- `widget.ts`: runtime schema, typed attributes, i18n labels, `example` data; must export a `default` object with the same `name`
- `render.tsx`: receives `attributes`, supports CSS/SCSS imports; default export is the render component

Rule of thumb:
- Put plain static metadata in `widget.json`.
- Put translated labels or runtime-derived metadata in `widget.ts`.

## Output shape

Build output per widget:

- `build/widgets/{widget-name}/render.js` + `render.min.js`
- `build/widgets/{widget-name}/render.min.asset.php`
- `build/widgets/{widget-name}/widget.js` + `widget.min.js`
- `build/widgets/{widget-name}/widget.min.asset.php`
- `build/widgets/registry.php`
- `build/widgets.php`

Module handle pattern: `{handlePrefix}/widgets/{widget-dir-name}/render` and `.../widget`.

`widgets.php` registers widget entries as script modules via `wp_register_script_module()`. In watch mode, only the changed widget is rebuilt.

## Widget-to-page wiring (decoupled from wp-build)

Widget-to-page injection is **no longer handled by wp-build itself**. Surface packages (e.g., the host page or a dashboard surface) are responsible for wiring registered widgets into their UI.

The build emits module handles in the build manifest; the host surface retrieves them via a getter and injects them into the appropriate page.

Widget types are now registered server-side via a PHP registry and exposed through a REST endpoint (`/wp/v2/widget-types`). Do not assume wp-build auto-injects widgets into any page.

## Boot dependencies filter

Page templates now support a `{page_id}_script_module_dependencies` filter that allows surfaces to append additional script-module dependencies (including widget modules) to a specific page. See `references/php-integration.md` for details.

## Cautions

- Do not present widgets as stable API surface.
- Widget-to-page wiring is the surface package's responsibility, not wp-build's.
- Verify host expectations against the exact Gutenberg/Core version in use.
- If a plugin does not need widget discovery, avoid adding `widgets/` preemptively.
- The server-side widget type registry and REST endpoint are recent additions; check whether your target host version includes them.
