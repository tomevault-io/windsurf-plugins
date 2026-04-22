---
trigger: always_on
description: This is a Tapestry connector that integrates Miniflux RSS reader into the Tapestry timeline app. It allows users to view unread articles from their Miniflux instance and mark them as read directly from Tapestry.
---

# Tapestry Miniflux Connector

## Project Overview

This is a Tapestry connector that integrates Miniflux RSS reader into the Tapestry timeline app. It allows users to view unread articles from their Miniflux instance and mark them as read directly from Tapestry.

## Architecture

### Connector Structure
```
ch.alienlebarge.miniflux/
├── plugin-config.json    # Connector metadata (id, display_name, check_interval, item_style)
├── ui-config.json         # User configuration inputs (apiToken, limit)
├── plugin.js              # Main connector logic with required functions
├── actions.json           # Available user actions (mark_as_read)
└── README.md              # Detailed user documentation
```

**Important**: The folder name (`ch.alienlebarge.miniflux`) MUST match the `id` field in `plugin-config.json`.

### Configuration Variables

The connector uses these configuration variables (defined in `ui-config.json`):

- **`site`** - Miniflux instance URL (configured via `site_prompt` in plugin-config.json)
- **`apiToken`** - Miniflux API token for authentication
- **`limit`** - Number of articles to fetch (default: 500)

These variables are automatically available as global variables in `plugin.js`.

### Required Tapestry Functions

The connector implements three mandatory functions:

1. **`verify()`** - Validates configuration and authenticates with Miniflux API (`/v1/me` endpoint)
   - Skips verification if `site` or `apiToken` are not yet provided (iOS behavior during typing)
   - Returns user info and sets `displayName` via `processVerification()`

2. **`load()`** - Fetches unread entries from Miniflux API (`/v1/entries`) and converts them to Tapestry Items
   - Uses adaptive time window: 7 days on first load, 4 hours on subsequent loads
   - Uses the user-configured `limit` (default 500)
   - Stores fetch timestamp with `setItem()` for multi-device sync
   - Calls `processResults()` with array of items

3. **`performAction(actionId, actionValue, item)`** - Handles user actions via Miniflux API
   - `mark_as_read` / `mark_as_unread`: Updates entry status via `PUT /v1/entries`
   - `star` / `unstar`: Toggles bookmark via `PUT /v1/entries/{id}/bookmark`
   - Uses `raiseCondition("authorize")` on 401 errors to trigger re-authentication

### Item Creation Pattern

```javascript
// Create item with unique URI and date
var uri = entry.url + "#" + entry.id;
var date = new Date(entry.published_at);
var item = Item.createWithUriDate(uri, date);

// Set basic properties
item.title = entry.title;
item.body = entry.content;  // Full HTML content

// Use Identity API for author (feed name with favicon)
if (entry.feed && entry.feed.title) {
    var author = Identity.createWithName(entry.feed.title);

    // Add favicon using DuckDuckGo icon service
    if (entry.feed.site_url) {
        var domain = entry.feed.site_url.replace(/^https?:\/\//, "").replace(/\/.*$/, "");
        author.avatar = "https://icons.duckduckgo.com/ip3/" + domain + ".ico";
    }

    item.author = author;
}

// Use Identity API for source
if (entry.feed && entry.feed.title) {
    var source = Identity.createWithName(entry.feed.title);

    if (entry.feed.site_url) {
        source.uri = entry.feed.site_url;
    }

    item.source = source;
}

// Add actions using actions dictionary
var actions = {};
var entryId = entry.id.toString();

if (entry.status === "unread") {
    actions["mark_as_read"] = entryId;
} else {
    actions["mark_as_unread"] = entryId;
}

if (entry.starred) {
    actions["unstar"] = entryId;
} else {
    actions["star"] = entryId;
}

item.actions = actions;

// Add media attachments from enclosures
if (entry.enclosures && entry.enclosures.length > 0) {
    var attachments = [];
    for (var j = 0; j < entry.enclosures.length; j++) {
        var enclosure = entry.enclosures[j];
        if (enclosure.url && enclosure.mime_type) {
            var attachment = MediaAttachment.createWithUrl(enclosure.url);
            attachment.mimeType = enclosure.mime_type;
            attachments.push(attachment);
        }
    }
    if (attachments.length > 0) {
        item.attachments = attachments;
    }
}
```

**Critical Guidelines**:
- Never assign `undefined` to item properties. JavaScript will convert it to the string `"undefined"`. Use conditional logic instead.
- Always use `Identity.createWithName()` for `author` and `source` - never use plain objects
- Always convert actionValue to string: `entry.id.toString()`
- Use DuckDuckGo icon service for favicons: `https://icons.duckduckgo.com/ip3/{domain}.ico`

## API Integrations

### Miniflux API
- **Base URL**: User-configured instance URL (e.g., `https://your-instance.miniflux.app`)
- **Authentication**: X-Auth-Token header with API token
  ```javascript
  {
      "X-Auth-Token": apiToken,
      "Content-Type": "application/json"
  }
  ```
- **Key Endpoints**:
  - `GET /v1/me` - Verify authentication and get user info
  - `GET /v1/entries?status=unread&order=published_at&direction=desc&{timeParam}={timestamp}&limit={limit}` - Fetch entries

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alienlebarge) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
