# Fullscreen SPA Navigation & Menu Architecture

Use this reference when building a fullscreen SPA page with `@wordpress/build` and you need a stable sidebar menu architecture.

## Core pattern

Treat the menu tree as **PHP-first**:

- PHP defines structure, route targets, nesting, and display order.
- JS init modules attach icons or other light metadata after boot.
- `@wordpress/boot` should receive the final ordered tree from PHP on first load.

This avoids race conditions, UI flashes, and fragile client-side menu assembly.

## Generated helper

`@wordpress/build` generates a helper in `build/build.php` with this shape:

```php
{prefix}_register_{page_slug}_menu_item( $id, $label, $to, $parent_id = '', $parent_type = '' );
```

Use it as the canonical API for fullscreen sidebar registration.

## Top-level items

Register standard items with a route path:

```php
{prefix}_register_{page_slug}_menu_item( 'dashboard', __( 'Dashboard', 'text-domain' ), '/' );
{prefix}_register_{page_slug}_menu_item( 'analytics', __( 'Analytics', 'text-domain' ), '/analytics' );
```

Rules:
- PHP call order is the rendered menu order.
- Keep IDs stable and semantic because JS icon mapping depends on them.
- Use translated labels and the plugin text domain.

## Parent/child hierarchies

For parent groups, pass an empty route target and choose a parent mode.

### `dropdown` (recommended default)

Use for most admin/settings/data-management groups where fast switching matters more than immersion.

```php
{prefix}_register_{page_slug}_menu_item( 'data-group', __( 'Data', 'text-domain' ), '', '', 'dropdown' );
{prefix}_register_{page_slug}_menu_item( 'items', __( 'All Items', 'text-domain' ), '/items', 'data-group' );
{prefix}_register_{page_slug}_menu_item( 'categories', __( 'Categories', 'text-domain' ), '/categories', 'data-group' );
```

### `drilldown`

Use when a parent should open a dedicated sliding panel, similar to the Site Editor sidebar experience.

```php
{prefix}_register_{page_slug}_menu_item( 'settings-group', __( 'Settings', 'text-domain' ), '', '', 'drilldown' );
{prefix}_register_{page_slug}_menu_item( 'general', __( 'General', 'text-domain' ), '/settings/general', 'settings-group' );
```

Rules:
- Apply `parent_type` only to items that are themselves parents.
- Child items should usually only set `$parent_id`.
- Do not fake parent nodes in JS.

## Icon mapping in JS

Icons belong in an init module because PHP should not try to emit React SVG components.

```ts
import { dispatch } from '@wordpress/data';
import { store as bootStore } from '@wordpress/boot';
import { home, chartBar, settings } from '@wordpress/icons';

export async function init() {
	dispatch( bootStore ).updateMenuItem( 'dashboard', { icon: home } );
	dispatch( bootStore ).updateMenuItem( 'analytics', { icon: chartBar } );
	dispatch( bootStore ).updateMenuItem( 'settings-group', { icon: settings } );
}
```

Rules:
- Target the same stable IDs registered in PHP.
- Use `updateMenuItem()` for icons and light presentation metadata only.
- Do not use JS to reorder items or construct the hierarchy.

## Debugging heuristics

If navigation is wrong, debug in this order:

1. Check PHP registration order.
2. Check each item's `$id`, `$to`, `$parent_id`, and `$parent_type`.
3. Confirm the route paths actually exist.
4. Only then inspect JS icon mapping or init-module execution.

If the tree is structurally wrong, the bug is usually in PHP, not React.
