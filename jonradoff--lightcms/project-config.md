---
trigger: always_on
description: LightCMS is a lightweight, self-hosted content management system built in Go. It uses MongoDB for data storage and generates static HTML pages for public content serving.
---

# LightCMS - Claude Code Memory

## Project Overview

LightCMS is a lightweight, self-hosted content management system built in Go. It uses MongoDB for data storage and generates static HTML pages for public content serving.

**Key URLs:**
- Admin Dashboard: `/cm`
- Public Site: `/`
- MCP Server: `bin/lightcms-mcp` (stdio transport)

## MCP Server Integration

**IMPORTANT:** All website content operations MUST go through the MCP server. Do NOT:
- Write scripts to directly modify the database
- Use Go code to create/edit/delete content
- Bypass the MCP server for any content management tasks

If the MCP server is not available or not working, ASK the user for permission before attempting any content changes through other means.

### Starting the MCP Server

If the user requests a content operation and the MCP server is not yet running, start it using the convenience script:

```bash
./bin/lightcms-mcp-wrapper.sh
```

This wrapper script sets up the required environment variables (LIGHTCMS_CONFIG_DIR) and launches the MCP server. The MCP server uses stdio transport, so it will be connected automatically once started.

### Content Operations (REQUIRE MCP or explicit user permission):
- Creating, editing, publishing, or deleting content
- Managing templates and their HTML layouts
- Uploading or managing assets (images, CSS, JS, documents)
- Updating theme settings (colors, fonts, header/footer HTML)
- Managing redirects, folders, and collections
- Viewing or reverting content versions
- Site configuration changes

### Code Changes (allowed without MCP):
- Adding new features to LightCMS itself
- Fixing bugs in the application
- Changing application behavior or logic
- Adding new MCP tools
- Modifying database schemas or indexes
- Security improvements

### ⚠️ IMPORTANT: Rebuild MCP binary after adding tools

The Claude Desktop MCP integration uses a **local binary** (`bin/lightcms-mcp`) that connects to the remote server. The binary encodes all tool definitions — if you add or remove MCP tools without rebuilding it, Claude Desktop will see stale/missing tools.

**After any change to `cmd/mcp/` or `internal/mcp/`:**
```bash
go build -o bin/lightcms-mcp ./cmd/mcp
```

Then **restart Claude Desktop** to reload the MCP server. Until it's restarted, Cowork will still use the old binary.

### Setting Up MCP with Claude Code

Run the setup script from the lightcms directory:

```bash
./setup-mcp.sh
```

This builds the MCP server, creates the wrapper script, and registers it with Claude Code.

**Manual setup** (if needed):
```bash
# Build the MCP server
go build -o bin/lightcms-mcp ./cmd/mcp

# Register with Claude Code (use the wrapper script, not the binary directly)
claude mcp add --transport stdio lightcms-mcp -- /path/to/lightcms/lightcms-mcp-wrapper.sh
```

After registering, restart Claude Code. You can verify the server is connected:
- Run `/mcp` in Claude Code to check status
- Run `claude mcp list` in terminal to see registered servers

Once connected, you can ask Claude to manage your content naturally:
- "Create a new blog post about AI"
- "List all my published content"
- "Update the homepage hero image"
- "Delete the /random page"

### MCP Server Location
Binary: `bin/lightcms-mcp`
Config: Uses same `config.dev.json` or environment variables as main server

### Available MCP Tools (107 total):

**Content (23 tools):** list_content, get_content, create_content, update_content, update_content_by_path, publish_content, publish_multiple, unpublish_content, delete_content, restore_content, preview_content, get_content_versions, get_content_version, revert_to_version, bulk_create_content, bulk_update_content, bulk_field_operation, export_content, get_backlinks

**Templates (5 tools):** list_templates, get_template, create_template, update_template, delete_template

**Snippets (5 tools):** list_snippets, get_snippet, create_snippet, update_snippet, delete_snippet

**Assets (6 tools):** list_assets, list_asset_folders, get_asset, upload_asset, upload_asset_from_url, delete_asset

**Search (7 tools):** search_content, search_replace_preview, search_replace_execute, scoped_search_replace_preview, scoped_search_replace_execute, end_user_search, reindex_embeddings

**Settings (18 tools):** get_theme, update_theme, get_theme_versions, get_theme_version, revert_theme_to_version, pin_theme_version, unpin_theme_version, get_site_config, update_site_config, list_redirects, create_redirect, update_redirect, delete_redirect, list_folders, create_folder, get_folder, delete_folder, list_collections, create_collection, get_collection, update_collection, delete_collection, regenerate_all_content

**Forks (8 tools):** list_forks, create_fork, get_fork, fork_page, remove_fork_page, merge_fork, archive_fork, delete_fork

**Comments (3 tools, v6.0+):** list_comments, create_comment, delete_comment


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonradoff/lightcms](https://github.com/jonradoff/lightcms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
