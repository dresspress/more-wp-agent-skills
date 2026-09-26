# Verification

Use this checklist after implementing or debugging an `@wordpress/build` project.

## Build checks

- Run the project's configured build command (`wp-build` or npm script).
- **JSX file extension check**: Confirm no files containing JSX use a plain `.js` extension (must be `.jsx` or `.tsx` as of 0.23.0+).
- Confirm generated PHP files exist in `build/` and match current upstream expectations.
- Confirm generated asset files (`*.asset.php`) list the expected script module dependencies.
- Confirm `build/widgets/registry.php` correctly reflects declarative fields from `widget.json` (attributes, title, description, help, category, presentation, actions).

## PHP checks

- Include the generated build entry point from the plugin.
- **Capability and access check**: For full-page standalone modes, verify the generated `{{PREFIX}}_{{PAGE_SLUG_UNDERSCORE}}_intercept_render()` enforces user authentication and the expected capability (configured via `"capability"` in `package.json` under `wpPlugin.pages`, defaulting to `manage_options`).
- Verify that the capability passed to `add_menu_page()` or `add_submenu_page()` matches the capability configured in `package.json`.
- Verify the actual generated callback names before registering admin menus.
- Inspect generated route/menu/widget helpers before calling them from custom PHP.
- Confirm widgets' script module handles (render and widget modules) are registered and resolved properly, and verify they are injected as dynamic `$boot_dependencies` in generated page templates.
- Confirm that when the plugin does not provide its own boot module, the template falls back to Core's bundled `@wordpress/boot`.

## Runtime checks

- Load the admin page in the browser.
- Check the PHP error log and browser console for script module resolution failures.
- Confirm script modules resolve and import maps (`wp_script_modules()->print_import_map()`) are present.
- Confirm route deep links using the `p` query parameter work as expected.
- If using `@wordpress/ui` components outside standard block editor screens, confirm `@wordpress/theme/design-tokens.css` is loaded and the layout root element has `isolation: isolate` for portaled popovers.
- Test with JavaScript disabled to confirm the `.hide-if-js` error notice renders visibly.
- Confirm the host environment provides `@wordpress/boot`, `@wordpress/route`, `@wordpress/theme`, and `@wordpress/private-apis` (WordPress Core 7.0+ or active Gutenberg plugin).

## Experimental checks

- Re-check upstream docs/source when adopting routes, pages, or widgets.
- Note any source/docs discrepancy in the implementation summary.
- Avoid declaring an experimental behavior stable just because it worked in one generated build.
- Validate that namespaced imports resolving to non-installed local packages compile successfully and fall through to esbuild resolution without crashing the build.
