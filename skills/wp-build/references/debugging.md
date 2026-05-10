# Debugging

Use this guide to troubleshoot common issues when developing with `@wordpress/build`.

## 1. Blank Screen or 404 in Admin

### Check PHP Callbacks
Verify that the callback function in `add_menu_page` or `add_submenu_page` matches exactly what `@wordpress/build` generated.
- Pattern: `{plugin_name}_{page_id}_render_page` (Fullscreen)
- Pattern: `{plugin_name}_{page_id}_wp_admin_render_page` (WP-Admin)

### Check Build Output
Ensure `build/build.php` exists and that you have included it in your plugin.

### Check Script Errors
Open the browser console. If you see "Module not found" or "failed to load script module", it might be:
- Missing `wpScript: true` in a package's `package.json`.
- A dependency not being correctly externalized (check `wpPlugin.externalNamespaces`).
- The build not being run after adding a new package.

## 2. Wrong Layout (Sidebar issues)

### Fullscreen mode shows WP Admin sidebar
- You are likely using the wrong callback or slug. Ensure the slug DOES NOT have `-wp-admin` suffix if you want fullscreen.

### WP-Admin mode hides sidebar or looks broken
- You are likely using the Fullscreen callback (`_render_page`) instead of the WP-Admin callback (`_wp_admin_render_page`).

## 3. Routes not rendering

### "Route not found" message
- Check if the route's `package.json` has the correct `path`.
- Ensure the `page` field in route's `package.json` matches an entry in `wpPlugin.pages`.

### Stage/Inspector not appearing
- Ensure you use **named exports** in `stage.tsx` and `inspector.tsx`:
  ```tsx
  export const stage = () => ...; // CORRECT
  export default () => ...;      // WRONG
  ```
- Check if the file names are exactly `stage.tsx` or `inspector.tsx`.

## 4. Build Failures

### Missing workspaces
- Ensure root `package.json` has `"workspaces": ["packages/*"]`.

### Syntax errors in SCSS
- `@wordpress/build` is strict about SCSS. Check the terminal output for specific line numbers.

### EMFILE: too many open files
- Common on macOS when using `--watch`.
- Fix: `ulimit -n 10240`

## 5. Script Module Issues

### Browser doesn't support import maps
- `@wordpress/build` relies on Script Modules and Import Maps (WordPress 6.5+).
- Ensure you are on a modern browser.
- Ensure WordPress is up to date.

### Page/routing code loads but boot APIs are missing
- As of `@wordpress/build` 0.10.0, page/routing features no longer bundle `@wordpress/boot`, `@wordpress/route`, `@wordpress/theme`, or `@wordpress/private-apis`.
- Verify the host is WordPress Core 7.0+ or the Gutenberg plugin is active.
- If the page works in fullscreen mode but not in WP-Admin mode, confirm you are not relying on fullscreen-only `initModules` or boot sidebar menu items.

### Dependency version mismatch
- Check `.asset.php` files in `build/` to see what dependencies are being registered.
- Ensure all `@wordpress/*` packages in your monorepo are using compatible versions.

## 6. CSS Module Issues

### Styles missing in tests (NODE_ENV=test)
- CSS module output intentionally **skips automatic style injection** when `NODE_ENV` is `test`.
- Node-based DOM implementations (jsdom) do not reliably support modern CSS, so styles are not injected into the DOM during unit tests.
- If your tests need actual styles in the DOM, run them in a real browser environment instead.
- This is expected behavior, not a bug.

### CSS module styles missing in editor iframes
- In Gutenberg plugin versions after `@wordpress/build` 0.13.0, CSS module styles are registered with `@wordpress/style-runtime` so they can be injected across registered documents (including editor iframes).
- If styles work on the main document but not inside an iframe, check whether `@wordpress/style-runtime` is available on your host.
- This feature was introduced after 0.13.0; verify your Gutenberg plugin version includes it before debugging further.
