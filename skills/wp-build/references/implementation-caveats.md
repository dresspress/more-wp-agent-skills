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

## Page modes

Fullscreen and WP-Admin modes can have different boot inputs and generated callbacks. Verify the generated page templates and the actual `build/pages/{page}/` files before advising on:

- menu slugs
- callback names
- init modules
- menu item registration
- boot dependency filters
- sidebar behavior

Do not describe fullscreen sidebar customization as a WP-Admin-mode capability unless the current generated WP-Admin template passes the needed boot data.

## Generated PHP helpers

Generated helper names and signatures have changed before. Never invent them from memory.

After running the build, inspect:

- `build/build.php`
- `build/routes.php`
- `build/pages.php`
- `build/pages/{page}/page.php`
- `build/pages/{page}/page-wp-admin.php`
- `build/widgets.php` when widgets are used

## Widgets

Widget behavior has moved quickly. Verify whether the current package only discovers/registers widget modules or also wires them into pages.

Do not assume widgets automatically appear in a page UI. Check the host surface and generated dependency/filter behavior.

## Dependency and host assumptions

Do not hard-code compatibility claims from this skill. Check current `package.json` peer dependencies, changelog notes, generated asset files, and the target WordPress/Gutenberg runtime.
