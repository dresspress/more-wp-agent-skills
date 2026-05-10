# Verification

Use this checklist after implementing or debugging an `@wordpress/build` project.

## Build checks

- Run the project's configured build command.
- Confirm generated PHP files exist and match the current upstream expectations.
- Confirm generated asset files list the dependencies you expect.
- Confirm `build/` and `build-module/` output roles match the current package behavior.

## PHP checks

- Include the generated build entry point from the plugin.
- Verify the actual generated callback names before registering admin menus.
- Verify the selected page mode uses the matching slug and callback.
- Inspect generated route/menu/widget helpers before calling them from custom PHP.

## Runtime checks

- Load the admin page in the browser.
- Check the PHP error log and browser console.
- Confirm script modules resolve and import maps are present.
- Confirm route deep links using the `p` query parameter work where expected.
- Confirm fullscreen-specific features are not being tested in WP-Admin mode unless current source supports them.

## Experimental checks

- Re-check upstream docs/source when adopting routes, pages, or widgets.
- Note any source/docs discrepancy in the implementation summary.
- Avoid declaring an experimental behavior stable just because it worked in one generated build.
