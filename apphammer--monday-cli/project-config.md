---
trigger: always_on
description: This is a CLI tool for interacting with the Monday.com GraphQL API.
---

# Claude Instructions for Monday CLI

This is a CLI tool for interacting with the Monday.com GraphQL API.

## Project Structure

```
src/monday_cli/
├── __init__.py          # Package init with version
├── cli.py               # Main CLI entry point with all command group registrations
├── client/
│   ├── client.py        # HTTP client with retry/rate limiting
│   ├── models.py        # Pydantic models for API responses
│   ├── queries.py       # GraphQL query templates
│   └── mutations.py     # GraphQL mutation templates
├── commands/
│   ├── workspaces.py    # Workspace management commands
│   ├── boards.py        # Board management commands
│   ├── groups.py        # Group management commands
│   ├── items.py         # Item management commands
│   ├── subitems.py      # Subitem management commands
│   ├── statuses.py      # Status listing commands
│   ├── updates.py       # Update management commands
│   └── docs.py          # Document management commands
└── utils/
    ├── output.py        # Output formatting utilities (JSON, tables)
    └── error_handler.py # Error handling and custom exceptions
```

## Monday.com API

### Authentication

Get your API token from: https://apphammer.monday.com/admin/integrations/api

Set the token as environment variable:
```bash
export MONDAY_API_TOKEN="your_token_here"
```

### API Endpoint

- Base URL: `https://api.monday.com/v2`
- All requests are POST with GraphQL body
- Authentication via `Authorization: your_api_token` header

### GraphQL Patterns

#### Queries

Items query requires `ID!` type (string, not int):
```graphql
query GetItem($itemIds: [ID!]!) {
  items(ids: $itemIds) {
    id
    name
    board { id name }
    column_values { id text value type }
  }
}
```

Board columns query (includes status settings):
```graphql
query GetBoardColumns($boardIds: [ID!]!) {
  boards(ids: $boardIds) {
    columns {
      id
      title
      type
      settings_str  # JSON string with status labels
    }
  }
}
```

#### Mutations

Create item:
```graphql
mutation CreateItem($boardId: ID!, $itemName: String!, $columnValues: JSON) {
  create_item(board_id: $boardId, item_name: $itemName, column_values: $columnValues) {
    id
    name
  }
}
```

Update column value (for status changes):
```graphql
mutation ChangeColumnValue($boardId: ID!, $itemId: ID!, $columnId: String!, $value: JSON!) {
  change_column_value(board_id: $boardId, item_id: $itemId, column_id: $columnId, value: $value) {
    id
  }
}
```

### Status Columns

Status columns have `settings_str` containing JSON with label mappings:
```json
{
  "labels": {
    "0": "Done",
    "1": "Working on it",
    "2": "Stuck"
  }
}
```

To update a status, use the index in the value:
```json
{"index": 1}  // Sets status to "Working on it"
```

### Rate Limiting

- Monday.com has complexity-based rate limiting
- Track complexity via `complexity { before after }` in queries
- CLI implements 60 calls/minute conservative limit
- Retry logic with exponential backoff (1s, 2s, 4s)

### Common Column Types

| Type | Description | Value Format |
|------|-------------|--------------|
| `status` | Status dropdown | `{"index": N}` |
| `text` | Plain text | `"string value"` |
| `date` | Date picker | `"YYYY-MM-DD"` |
| `numbers` | Numeric | `"123"` or `123` |
| `people` | Person picker | `{"personsAndTeams": [{"id": N, "kind": "person"}]}` |
| `dropdown` | Dropdown | `{"ids": [1, 2]}` |

## Development

### Build Binary

```bash
python build/build_binary.py
```

Creates standalone Linux binary at `dist/monday`.

### Run from Source

```bash
python -m monday_cli --help
```

### Testing

```bash
pytest
```

## CLI Framework

Uses Typer (v0.21.0+) with **8 command groups**:

### Command Groups Overview

1. **`monday workspaces <command>`** - Workspace operations
   - `list` - List workspaces with membership filtering

2. **`monday boards <command>`** - Board operations
   - `list` - List boards with state and workspace filtering

3. **`monday groups <command>`** - Group operations
   - `list` - List groups on a board
   - `create` - Create new group
   - `delete` - Delete group

4. **`monday items <command>`** - Item operations
   - `get` - Get item details
   - `list` - List items with pagination and group filtering
   - `create` - Create new item
   - `update` - Update item column value using human-readable titles
   - `delete` - Delete item with confirmation
   - `list-columns` - List all board columns with types and options

5. **`monday subitems <command>`** - Subitem operations
   - `get` - Get subitem details
   - `list` - List subitems with pagination (by item or board)
   - `create` - Create new subitem
   - `update` - Update subitem column value using human-readable titles
   - `delete` - Delete subitem with confirmation
   - `list-columns` - List all board columns
   - `list-statuses` - List status columns with options

6. **`monday statuses <command>`** - Status operations
   - `list` - List all status columns and options for a board

7. **`monday updates <command>`** - Update operations
   - `create` - Post update to item or subitem

8. **`monday docs <command>`** - Document operations
   - `get` - Get document content as Markdown
   - `append` - Append Markdown content to document (creates if needed)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AppHammer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
