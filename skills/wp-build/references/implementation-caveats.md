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

Key routing updates since v0.20.0+:
- **Extension support (0.24.0+)**: Route files support `.tsx`, `.ts`, `.jsx`, `.mts`, `.cts`, `.mjs`, and `.cjs`. Note that as of 0.23.0+, **JSX syntax is no longer parsed in `.js` files**; rename them to `.jsx` or `.tsx`.
- **Full file imports (0.24.0+)**: Route entry points import stage, inspector, and canvas using their full emitted filenames instead of relying on esbuild extensionless path resolution.
- **Canvas-only routes (0.22.0+)**: Routes that only export a `canvas` (without stage or inspector) now properly register their content module and render custom canvases.

When documenting `route.tsx` lifecycle hooks, prefer upstream docs. If using hooks not covered by the README, confirm against `packages/boot/src/store/types.ts` and `packages/boot/src/components/app/router.tsx`.

Key route configuration and lifecycle hooks to leverage in `route.tsx`:
- **`title`**: `(context: RouteLoaderContext) => string | Promise<string>` - Dynamically computes the document title, announced to screen readers.
- **`inspector`**: `(context: RouteLoaderContext) => boolean | Promise<boolean>` - Controls whether to render/display the sidebar panel for the active route.

## Page modes

Fullscreen and WP-Admin modes have different boot inputs and generated callbacks. Verify the generated page templates and the actual `build/pages/{page}/` files before advising on:

- menu slugs and capabilities
- callback names
- init modules
- boot dependency filters
- sidebar behavior

Key page updates (v0.21.0 - v0.24.0):
- **Authentication & Capability enforcement (0.23.0+)**: Full-page standalone pages render via `admin_init` interception. The generated `{{PREFIX}}_{{PAGE_SLUG_UNDERSCORE}}_intercept_render()` function strictly enforces `if ( ! is_user_logged_in() ) { auth_redirect(); }` and `current_user_can( $capability )`. Without this, unauthenticated requests reaching `admin_init` (e.g. `admin-post.php`) would render the page.
- **Configurable `capability` setting (0.23.0+)**: Under `wpPlugin.pages`, page objects accept a `"capability"` property (defaults to `"manage_options"`):
  ```json
  {
    "id": "my-admin-page",
    "capability": "edit_theme_options",
    "experimental": false
  }
  ```
- **Fallback to Core Boot module (0.22.0+)**: If a plugin does not build its own boot module under `modules/boot/`, generated templates automatically fall back to WordPress Core's bundled `@wordpress/boot` module (`ABSPATH . WPINC . '/js/dist/script-modules/boot/index.min.asset.php'`).
- **No-JS fallback notice (0.21.0+)**: Generated templates include a `.no-js` body class and render a `<noscript>` / `.hide-if-js` error notice explaining that JavaScript is required. Critical styles are scoped to `body.js` so the notice remains styled if scripts fail.
- **Single-page DOMContentLoaded deferral (0.15.0+)**: Single-page admin templates defer dynamic `import("@wordpress/boot")` until `DOMContentLoaded` so boot evaluates after classic scripts are loaded.

## Generated PHP helpers

Generated helper names and signatures change across versions. Never invent them from memory.

After running the build, inspect:

- `build/build.php`
- `build/routes.php`
- `build/pages.php`
- `build/pages/{page}/page.php` (contains standalone interceptor and render callback)
- `build/pages/{page}/page-wp-admin.php`
- `build/widgets.php` and `build/widgets/registry.php`

Note: Generated loader files wrap `require_once` in `file_exists()` checks to prevent fatal errors during concurrent deployment file writes.

## Widgets

Widgets follow a dual-entry architecture splitting discovery metadata from runtime code:
1. `widget.json`: Static discovery metadata, category, presentation, and translatable strings.
2. `widget.ts`: Runtime contract, DataViews attribute schema with `relevance`, icon, and example.
3. `render.tsx`: UI component receiving typed attributes. Supports `.tsx`, `.ts`, `.jsx`, `.js`, `.mjs`.

Key widget capabilities (v0.19.0 - v0.24.0):
- **Declarative `attributes` in `widget.json` (0.24.0+)**: Carried directly into `build/widgets/registry.php`.
- **Translatable metadata in `widget.json` (0.19.0+ - 0.21.0+)**: `title`, `description`, `help` (with `content` and `links`), `keywords`, `category`, and `textdomain` are forwarded to the PHP registry so the host can translate fields server-side without a JS runtime.
- **Declarative icons and actions (0.20.0+, 0.21.0+)**: Declarative `icon` and `actions` in `widget.json` are passed to `build/widgets/registry.php`. Icons in `widget.ts` must be React/SVG components (e.g. `@wordpress/icons`), not Dashicons strings.
- **Relevance tiers (0.22.0+)**: Attribute fields support `relevance: 'high' | 'medium' | 'low'` (defaults to `'low'`).
- **Presentation modes (0.15.0+)**: Supports `'framed' | 'content-bleed' | 'full-bleed'`.
- **Dynamic boot dependencies**: Page templates resolve registered widgets and inject their handles as dynamic `$boot_dependencies`.

## Dependency and host assumptions

Do not hard-code compatibility claims. Check current `package.json` peer dependencies and changelog notes.

Key runtime/dependency shifts:
- **No JSX parsing in `.js` files (0.23.0+)**: Breaking change. `@wordpress/build` no longer parses JSX in `.js` files. All JSX syntax must be in `.jsx` or `.tsx` files.
- **Unbundled core packages (0.10.0+)**: `@wordpress/boot`, `@wordpress/route`, `@wordpress/theme`, and `@wordpress/private-apis` are not bundled. They must be provided by WordPress Core (7.0+) or Gutenberg plugin.
- **`@wordpress/theme` peer range (0.19.0+, 0.23.0+)**: Peer dependency range is widened to `>=0.8.0 <3.0.0` (supporting 1.x and 2.x).
- **`@wordpress/ui` adoption in Boot**:
  - Modern `@wordpress/boot` admin pages and dashboards migrate away from legacy `@wordpress/components` toward `@wordpress/ui` compound components (e.g. `<Tooltip.Root>`, `<Dialog.Root>`, `<EmptyState>`).
  - **Critical architectural distinction**: Unlike `@wordpress/components`, `@wordpress/ui` is **NOT exposed on `window.wp.ui`**. It is an npm package adhering to semver.
  - When used outside standard editor screens, `@wordpress/ui` requires `@wordpress/theme/design-tokens.css` for design token custom properties, and `.root { isolation: isolate; }` on the layout root for portaled popovers.
- **CSS Modules in iframe (0.14.0+)**: CSS Modules register with `@wordpress/style-runtime` to ensure injection into editor iframes.
