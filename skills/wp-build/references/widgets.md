# Widgets

Use this reference when a plugin needs experimental widget discovery under root `widgets/`.

Widgets are still experimental. Treat the directory shape, generated files, and host integration points as subject to change.

## Directory shape

```text
widgets/
└── hello-world/
    ├── widget.json
    ├── widget.ts
    └── render.tsx
```

## Responsibility split

- `widget.json`: static metadata that can live in plain JSON
- `widget.ts`: metadata that needs runtime logic, types, or i18n
- `render.tsx`: UI render entry

Rule of thumb:
- Put plain static metadata in `widget.json`.
- Put translated labels or runtime-derived metadata in `widget.ts`.

## Output shape

Current upstream README documents generated files such as:

- `build/widgets/{widget-name}/render.js`
- `build/widgets/{widget-name}/render.min.js`
- `build/widgets/{widget-name}/render.min.asset.php`
- `build/widgets/{widget-name}/widget.js`
- `build/widgets/{widget-name}/widget.min.js`
- `build/widgets/{widget-name}/widget.min.asset.php`
- `build/widgets/registry.php`
- `build/widgets.php`

`widgets.php` registers widget entries as script modules via `wp_register_script_module()`.

## Cautions

- Do not present widgets as stable API surface.
- Verify host expectations against the exact Gutenberg/Core version in use.
- If a plugin does not need widget discovery, avoid adding `widgets/` preemptively.
