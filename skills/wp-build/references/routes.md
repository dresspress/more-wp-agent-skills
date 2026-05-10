# Routes

Use this file when creating admin page routes in the experimental `routes/` directory.

## Route structure

Each route is a subdirectory of `routes/` with convention-based files:

```
routes/
└── settings/
    ├── package.json      # Route configuration (required)
    ├── stage.tsx         # Main content (required)
    ├── inspector.tsx     # Sidebar content (optional)
    ├── canvas.tsx        # Custom canvas (optional)
    └── route.tsx         # Lifecycle hooks (optional)
```

## Route package.json

```json
{
  "route": {
    "path": "/settings",
    "page": "my-admin-page"
  }
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `path` | string | URL path for this route (e.g., `/`, `/settings`) |
| `page` | string \| string[] | Page ID(s) this route belongs to |

### Multi-page routes

A route can appear on multiple pages:

```json
{
  "route": {
    "path": "/shared",
    "page": ["page-one", "page-two"]
  }
}
```

`route.page` values should usually match entries declared in `wpPlugin.pages`. Current upstream docs also say they can target existing pages registered by other plugins; treat that as experimental integration behavior and verify it in the exact host/plugin combination you care about.

## Route components

### stage.tsx (required)

Main content area. Must export a named `stage` component:

```tsx
import { __ } from '@wordpress/i18n';

export const stage = () => {
  return (
    <div className="my-plugin-content">
      <h1>{__('Page Title', 'my-plugin')}</h1>
      <p>{__('Main content goes here.', 'my-plugin')}</p>
    </div>
  );
};
```

### inspector.tsx (optional)

Sidebar panel. Must export a named `inspector` component:

```tsx
import { __ } from '@wordpress/i18n';
import { Button } from '@wordpress/components';

export const inspector = () => {
  return (
    <div className="my-plugin-sidebar">
      <h2>{__('Actions', 'my-plugin')}</h2>
      <Button variant="primary">
        {__('Save', 'my-plugin')}
      </Button>
    </div>
  );
};
```

### canvas.tsx (optional)

Custom canvas for editor-like interfaces. Must export a named `canvas` component:

```tsx
export const canvas = () => {
  return (
    <div className="my-plugin-canvas">
      {/* Full-width editor area */}
    </div>
  );
};
```

### route.tsx (optional)

Lifecycle hooks and visibility control for the route:

```tsx
export const route = {
  // Runs before route loads (auth checks, redirects)
  beforeLoad: ({ params, search }) => {
    // ...
  },

  // Load data for the route
  loader: async ({ params, search }) => {
    // ...
    return { data: '...' };
  },

  // Control inspector visibility (optional)
  // Return boolean or Promise<boolean>
  inspector: async ({ params, search }) => {
    // Logic to show/hide the sidebar for this route
    return true; 
  },

  // Control canvas rendering
  canvas: ({ params, search }) => {
    // Return CanvasData object: use default editor with postType/postId
    // Return null: use custom canvas.tsx component
    // Return undefined: no canvas
    return { postType: 'page', postId: params.id };
  },

  // Set the page title dynamically
  title: ({ params, search }) => {
    return 'Dynamic Title';
  }
};
```

## Route URLs

Routes are accessed via query parameter:

- Fullscreen: `admin.php?page={page_id}&p=/settings`
- WP-Admin: `admin.php?page={page_id}-wp-admin&p=/settings`

### Deep linking in PHP

```php
$url = admin_url(
  'admin.php?page=my-admin-page-wp-admin&p=' . urlencode('/settings')
);
```

## Common mistakes

1. **Missing named export**: Must be `export const stage = ...`, not default export
2. **Wrong file name**: Must be exactly `stage.tsx`, `inspector.tsx`, etc.
3. **Missing package.json**: Route directory must have `package.json` with `route` config
4. **Page mismatch**: `route.page` must match an entry in `wpPlugin.pages`
5. **Assuming route APIs are stable**: `routes/` is still experimental; re-check upstream docs when adopting new behavior
6. **Assuming fullscreen-only boot features exist in WP-Admin mode**: current generated `page-wp-admin.php` uses `initSinglePage()` and does not pass `initModules` or sidebar `menuItems`

Upstream reference:

- https://raw.githubusercontent.com/WordPress/gutenberg/trunk/packages/wp-build/README.md
