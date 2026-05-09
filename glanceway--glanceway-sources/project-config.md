---
trigger: always_on
description: Community repository for creating and distributing information sources for [Glanceway](https://glanceway.app), a macOS menu bar app that displays information items. Sources periodically fetch data from APIs/feeds and emit items for display.
---

# CLAUDE.md

## Project Overview

Community repository for creating and distributing information sources for [Glanceway](https://glanceway.app), a macOS menu bar app that displays information items. Sources periodically fetch data from APIs/feeds and emit items for display.

## Commands

```bash
npm install                                          # Install dependencies
npm run build-sources                                # Build all sources into dist/
npm run build-sources -- --source author/name        # Build a single source (also verifies compilation)
npm run generate-sources-json                        # Regenerate sources.json metadata
npm run generate-readme                              # Regenerate README.md
```

There is no test framework. Build the source to verify it compiles. There is no linter or formatter configured.

## Creating a Source

### Step 1: Create Source Directory

Create a `latest` directory with two files: `manifest.yaml` and `index.js`.

```
sources/<author>/<source-name>/latest/
├── manifest.yaml
└── index.js
```

To submit to this repository, place it under `sources/<author>/<source-name>/latest/` (e.g., `sources/myname/my-source/latest/`). When releasing a new version, update the files in `latest/` and bump the version in `manifest.yaml`. To preserve an older version, copy it to a version-numbered directory (e.g., `1.0.0/`) before updating `latest/`.

### Step 2: Implement index.js

Edit `index.js` to implement source logic. See the API Reference below and use existing sources as examples.

### Step 3: Update manifest.yaml Config

Add `config` entries for any values the user needs to provide (API tokens, usernames, etc.):

```yaml
config:
  - key: API_TOKEN
    name: API Token
    type: secret # string, number, boolean, secret, select, list, or multiselect
    required: true
    description: Your API token
  - key: USERNAME
    name: Username
    type: string
    required: false
    description: Optional username filter
  - key: SORT
    name: Sort Order
    type: select
    required: false
    default: hot
    description: Sort order for results
    options:
      - hot
      - new
      - top
  - key: CURRENCY
    name: Currency
    type: select
    required: false
    default: usd
    description: Display currency
    options:                              # options support label/value objects
      - label: US Dollar
        value: usd
      - label: Euro
        value: eur
  - key: CATEGORIES
    name: Categories
    type: multiselect                     # like select but allows multiple; stored as JSON array
    required: false
    description: Categories to show
    options:
      - label: Technology
        value: tech
      - label: Science
        value: science
      - label: Finance
        value: finance
```

### Step 4: Build and Verify

```bash
npm run build-sources -- --source myname/my-source
```

## Source Development Constraints

**NO external imports.** Sources cannot use `import` or `require` for external packages. All functionality is provided through the `api` parameter. Use `module.exports` to export the source function:

```javascript
module.exports = async (api) => {
  async function fetchData() {
    /* fetch, transform, emit */
  }

  // Start phase: initial fetch
  await fetchData();

  return {
    refresh: fetchData,
    stop() {
      /* optional cleanup */
    },
  };
};
```

## API Reference

All methods are available on the `api` parameter.

### api.emit(items)

Send items to Glanceway for display. Each item has these fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier |
| `title` | string | Yes | Main display text |
| `subtitle` | string | No | Secondary text below title |
| `url` | string | No | Link opened on click |
| `timestamp` | Date / string / number | No | ISO string, Unix timestamp, or Date |
| `notify` | boolean | No | If `true`, content changes re-mark read items as unread so users notice updates (default: `false`) |
| `tickerText` | string | No | Text shown in the menu bar scrolling ticker; falls back to `title` if omitted |

Use `notify: true` for notification-style items where one record represents an ongoing thread (e.g. a PR or issue) — without it, once a user reads the item, subsequent content updates remain marked as read. Use `tickerText` when `title` is too long or not suited for the menu bar ticker; keep it short so it reads cleanly when scrolling.

### api.fetch(url, options?)

Make HTTP requests.

Options: `method` (default `"GET"`), `headers`, `body`, `timeout` (default 30000ms).

Response: `ok` (boolean), `status`, `headers`, `text`, `json` (parsed if valid).

Example:

```javascript
const response = await api.fetch("https://api.example.com/data", {
  headers: { Authorization: `Bearer ${token}` },
});
if (response.ok && response.json) {
  // use response.json
}
```

### api.config.get(key)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [glanceway/glanceway-sources](https://github.com/glanceway/glanceway-sources) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
