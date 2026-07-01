# Implementation caveats

These are not a replacement for upstream docs. They are reminders of places where agents commonly over-trust examples or stale docs.

## Experimental surfaces

Treat `wpPlugin.pages`, `routes/`, page-mode helpers, and `widgets/` as experimental unless current upstream explicitly says otherwise.

Before depending on them, check:

- `packages/wp-build/CHANGELOG.md`
- `packages/wp-build/lib/build.mjs`
- `packages/wp-build/templates/*.php.template`
- generated files under the project's `build/`

## Routes

Do not assume every documented route behavior is active for every target page. Current source has historically filtered generated route data through active `wpPlugin.pages`; verify this before relying on routes that target pages registered elsewhere.

Do not assume route filenames are `.tsx` only. Check `route-utils.mjs` for the current extension list.

Do not assume a custom `canvas`-only route is registered correctly. Verify whether the current build treats `canvas` as content for route registry purposes.

When documenting `route.tsx` lifecycle hooks, prefer upstream docs. If using hooks not covered by the README, confirm against `packages/boot/src/store/types.ts` and `packages/boot/src/components/app/router.tsx`.

Key route configuration and lifecycle hooks to leverage in `route.tsx`:
- **`title`**: `(context: RouteLoaderContext) => string | Promise<string>` - Dynamically computes the document title, announced to screen readers.
- **`inspector`**: `(context: RouteLoaderContext) => boolean | Promise<boolean>` - Controls whether to render/display the sidebar panel for the active route.

## Page modes

Fullscreen and WP-Admin modes can have different boot inputs and generated callbacks. Verify the generated page templates and the actual `build/pages/{page}/` files before advising on:

- menu slugs
- callback names
- init modules
- menu item registration
- boot dependency filters
- sidebar behavior

Do not describe fullscreen sidebar customization as a WP-Admin-mode capability unless the current generated WP-Admin template passes the needed boot data.

Additionally, as of version 0.14.0+, the generated `build/pages.php` loader uses `file_exists()` checks to guard `require_once` statements. This prevents fatal errors during concurrent/hot deployments when files may briefly be missing on disk during high-traffic writes.

For single-page admin configurations (version 0.15.0+), the generated templates defer the dynamic `import( "@wordpress/boot" )` call until the `DOMContentLoaded` event fires. This ensures the boot script evaluates only after classic (non-module) script dependencies are fully loaded.

## Generated PHP helpers

Generated helper names and signatures have changed before. Never invent them from memory.

After running the build, inspect:

- `build/build.php`
- `build/routes.php`
- `build/pages.php`
- `build/pages/{page}/page.php`
- `build/pages/{page}/page-wp-admin.php`
- `build/widgets.php` when widgets are used (note the presence of `{{PREFIX}}_get_registered_widget_modules()` which memoizes discovered widgets and resolves dynamic handles).

Note: The generated loader files and registration templates (e.g. `module-registration.php.template`) now wrap `require_once` inside `file_exists()` conditionals to avoid fatal errors during concurrent deployment races.

## Widgets

Widget behavior has moved quickly. 

Key 0.15.0+ updates to verify:
- **`widget.json` `presentation` property**: Supports `'framed' | 'content-bleed' | 'full-bleed'` to describe the rendering intent of a widget. `'content-bleed'` keeps the widget header visible while the content area renders edge-to-edge. Check `widget.json` metadata for this key, which will generate `'presentation' => ...` mappings inside `build/widgets/registry.php`.
- **Widget header icons**: The `icon` property exported in `widget.ts` is restricted to SVG components or React components (e.g. from `@wordpress/icons`). Dashicons strings (such as `'wordpress'`) are **no longer supported**.
- **Local tsconfig config**: Widgets directories support local TypeScript compile checking (`@wordpress/*`, JSX, and CSS modules resolution).
- **Local `package.json`**: Each widget directory under `widgets/` supports an optional local `package.json` as an npm dependencies manifest.
- **Dynamic boot dependencies**: Generated page templates (`page.php` and `page-wp-admin.php`) now automatically resolve registered widgets and inject their handles (`widget_module` and `render_module`) as dynamic `$boot_dependencies` to ensure assets load properly on active pages.

As of version 0.14.0, generated CSS Module styles are registered to `@wordpress/style-runtime`. This is critical for ensuring that styles are correctly injected into registered documents, such as **editor iframes**. If you encounter missing styles when a component renders inside an iframe, verify that the project is using a recent version of `@wordpress/build` and that the styles are processed as CSS Modules.

Key styling updates since 0.11.0:
- **`data-wp-hash` calculation (0.11.0+)**: The build system derives the `data-wp-hash` attribute from the *transformed* CSS output instead of the raw source CSS. This prevents style hydration mismatches when multiple asset-processing pipelines modify the same files.
- **`#wpwrap` background removal (0.16.0+)**: The incorrect `#wpwrap` background color styling was removed from the generated wp-admin critical CSS. It now relies on the browser-native `body` background styling to prevent a temporary black flash during initial page load/hydration.

## Dependency and host assumptions

Do not hard-code compatibility claims from this skill. Check current `package.json` peer dependencies, changelog notes, generated asset files, and the target WordPress/Gutenberg runtime.

Key runtime/dependency shifts:
- **Unbundled core packages (0.10.0+)**: Core packages `@wordpress/boot`, `@wordpress/route`, `@wordpress/theme`, and `@wordpress/private-apis` are **no longer bundled** by `@wordpress/build`. They must be provided by the host environment—either **WordPress Core (7.0+)** or an active **Gutenberg plugin**. Plugins relying on these packages for pages or routes will fail to boot on older WordPress versions unless Gutenberg is active.
- **Namespaced import fallback (0.16.0+)**: When a namespaced import (matching your configured `@packageNamespace/*`) resolves to a package that is not installed in the local node dependencies, `getPackageInfo` returns `null` instead of throwing. This prevents build crashes, allowing esbuild to fall through and try to resolve the import using its standard pathing.
- **Getter export reversion (0.16.0+)**: In `@wordpress/build` version 0.14.0, esbuild's getter-based exports were replaced with data properties using a footer shallow copy (`Object.assign({}, globalName)`). However, **this optimization was reverted in version 0.16.0** to restore compatibility for Gutenberg 23.0 builds.

At runtime, `@wordpress/boot` now runs on React 19. If writing UI components for page navigation or dashboard views, migrate from the legacy `@wordpress/components` `Tooltip` to the new `@wordpress/ui` compound components (e.g. `<Tooltip.Root>`, `<Tooltip.Trigger>`, `<Tooltip.Popup>`) as the old version is phased out from the boot shell.
