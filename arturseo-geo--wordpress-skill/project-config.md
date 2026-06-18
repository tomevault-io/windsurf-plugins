---
trigger: always_on
description: Publish, manage, and optimize WordPress content via the WordPress REST API. Use this skill whenever the user wants to create or update WordPress posts, pages, or custom post types; manage categories, tags, or taxonomies; upload media or set featured images; work with Gutenberg blocks; configure WooCommerce products; query or bulk-edit content; or automate any WordPress workflow. Trigger for any mention of WordPress, WP, WooCommerce, Gutenberg, wp-json, wp-cli, plugins, themes, or publishing to a
---


# WordPress Skill

## Authentication Setup

WordPress REST API requires authentication. Ask the user for:
1. **Site URL** (e.g., `https://example.com`)
2. **Auth method** — prefer Application Passwords (WP 5.6+):
   - Users → Profile → Application Passwords → Add New
   - Returns `username:app-password` — base64 encode for Basic Auth header
3. **API base**: `{site_url}/wp-json/wp/v2/`

```javascript
const headers = {
  'Authorization': 'Basic ' + btoa('username:app-password'),
  'Content-Type': 'application/json'
};
```

### MCP Server Authentication (Recommended)
When a WordPress MCP server is configured, use its tools directly instead of raw HTTP.
MCP tools handle auth automatically — no need to manage headers or tokens.

```jsonc
// .mcp.json — example WordPress MCP config
{
  "mcpServers": {
    "wordpress": {
      "command": "npx",
      "args": ["-y", "@anthropic/wordpress-mcp"],
      "env": {
        "WP_URL": "https://example.com",
        "WP_USERNAME": "admin",
        "WP_APP_PASSWORD": "xxxx xxxx xxxx xxxx"
      }
    }
  }
}
```

### Security Best Practices
- **Never store credentials in code** — use environment variables or `.env` files
- Application Passwords are scoped to the user's role — create a dedicated API user with minimal permissions
- Rotate Application Passwords quarterly
- Use HTTPS exclusively — never send credentials over HTTP
- Add IP allowlisting in `.htaccess` or nginx if the API is only called from known servers
- Disable XML-RPC if not needed: `add_filter('xmlrpc_enabled', '__return_false');`

---

## Core Operations

### Create / Update a Post
```bash
POST /wp-json/wp/v2/posts
{
  "title": "Post Title",
  "content": "<p>HTML or Gutenberg blocks</p>",
  "status": "draft" | "publish" | "private" | "future",
  "date": "2026-03-22T10:00:00",
  "slug": "post-slug",
  "excerpt": "Short summary",
  "categories": [id1, id2],
  "tags": [id1, id2],
  "featured_media": media_id,
  "meta": { "custom_field": "value" },
  "template": "template-name.php"
}

# Update: PATCH /wp-json/wp/v2/posts/{id}
# Delete: DELETE /wp-json/wp/v2/posts/{id}?force=true
```

### Pages
```bash
POST /wp-json/wp/v2/pages
{
  "title": "Page Title",
  "content": "...",
  "status": "publish",
  "parent": 0,
  "template": "page-template.php",
  "menu_order": 1
}
```

### Get Posts (with filtering)
```bash
GET /wp-json/wp/v2/posts?status=publish&per_page=100&page=1
GET /wp-json/wp/v2/posts?search=keyword&categories=5
GET /wp-json/wp/v2/posts?orderby=modified&order=desc
GET /wp-json/wp/v2/posts?after=2026-01-01T00:00:00&before=2026-03-01T00:00:00
GET /wp-json/wp/v2/posts?slug=my-post-slug
GET /wp-json/wp/v2/posts?_embed&_fields=id,title,link,excerpt
```

**Pagination**: Response headers include `X-WP-Total` and `X-WP-TotalPages`.

### Upload Media
```bash
POST /wp-json/wp/v2/media
Content-Disposition: attachment; filename="image.jpg"
Content-Type: image/jpeg
[binary data]

# After upload — update alt text and caption:
PATCH /wp-json/wp/v2/media/{id}
{
  "alt_text": "Descriptive alt text",
  "caption": "Photo credit: Source",
  "title": { "raw": "Image Title" }
}
```

### Manage Taxonomies
```bash
GET  /wp-json/wp/v2/categories
POST /wp-json/wp/v2/categories  { "name": "New Cat", "slug": "new-cat", "parent": 0, "description": "Category description" }
GET  /wp-json/wp/v2/tags
POST /wp-json/wp/v2/tags  { "name": "New Tag", "slug": "new-tag" }

# Custom taxonomies (must have show_in_rest: true)
GET  /wp-json/wp/v2/{taxonomy_slug}
POST /wp-json/wp/v2/{taxonomy_slug}  { "name": "Term Name" }
```

### Custom Post Types
```bash
# CPTs must have show_in_rest: true in registration
GET  /wp-json/wp/v2/{cpt_slug}
POST /wp-json/wp/v2/{cpt_slug}
GET  /wp-json/wp/v2/types            # List all registered post types
GET  /wp-json/wp/v2/types/{slug}     # Get single type details
```

### Custom Fields (ACF and Meta)
```bash
# Native meta fields (must be registered with show_in_rest)
PATCH /wp-json/wp/v2/posts/{id}
{ "meta": { "custom_key": "value" } }

# ACF fields (requires ACF to REST API plugin or ACF PRO 5.11+)
# ACF exposes fields under the "acf" key:
GET /wp-json/wp/v2/posts/{id}?_fields=acf
PATCH /wp-json/wp/v2/posts/{id}
{ "acf": { "field_name": "value", "repeater_field": [{"sub_field": "val"}] } }

# Register meta for REST API visibility:
register_post_meta('post', 'my_field', [
    'show_in_rest' => true,
    'single'       => true,
    'type'         => 'string',
]);
```

---

## Gutenberg Block Patterns

When writing content as Gutenberg blocks (use when site uses block editor):

```html
<!-- wp:paragraph -->
<p>Text here</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":2} -->
<h2 class="wp-block-heading">Heading</h2>
<!-- /wp:heading -->

<!-- wp:image {"id":123,"sizeSlug":"full","linkDestination":"none"} -->

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arturseo-geo/wordpress-skill](https://github.com/arturseo-geo/wordpress-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
