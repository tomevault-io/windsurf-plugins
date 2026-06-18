---
trigger: always_on
description: >
---


# Shopping List

Manage the household shopping list. Add items with quantities, check them off,
organize by category. Data lives in `skills/shopping-list/data/`.

For full command reference and output formats, read `skills/shopping-list/references/commands.md`.

## Before Every Command

Run these checks before every shopping list operation, in order:

1. If `skills/shopping-list/data/` directory does not exist, create it.
2. If `data/active.json` does not exist, create it with:
   ```json
   {
     "items": [],
     "categories": ["Produce", "Dairy", "Meat", "Pantry", "Frozen", "Beverages", "Household", "Personal"],
     "lastModified": "<current ISO timestamp>"
   }
   ```
3. If `data/active.json` exists but fails to parse as valid JSON, rename it to `data/active.json.corrupt` and create a fresh default file. Tell the user: "Shopping list data was corrupted. Saved backup as active.json.corrupt and started a fresh list."
4. If `data/config.json` does not exist, create it with: `{ "user": null, "snoozes": {} }`
5. If `config.json` has `"user": null`, ask the user: "What's your name? I'll use it to track who added each item." Store their answer (lowercased) in `config.json` before proceeding with the original command.
6. Run the archive process (see Archive section below).

All file paths in this document are relative to `skills/shopping-list/` unless stated otherwise.

## Data Files

### data/active.json

The live shopping list. Contains all items that have not yet been archived.

```json
{
  "items": [
    {
      "id": "F47AC10B-58CC-4372-A567-0E02B2C3D479",
      "name": "Whole Milk",
      "normalizedName": "whole milk",
      "quantity": 2,
      "unit": "gallons",
      "category": "Dairy",
      "checkedOff": false,
      "checkedOffDate": null,
      "addedBy": "aj",
      "addedDate": "2026-02-24T10:00:00Z",
      "notes": null
    }
  ],
  "categories": ["Produce", "Dairy", "Meat", "Pantry", "Frozen", "Beverages", "Household", "Personal"],
  "lastModified": "2026-02-24T10:00:00Z"
}
```

Field rules:

- `id` -- Generate via `uuidgen` in bash. If that command is unavailable, construct an ID from the current ISO timestamp concatenated with 4 random hex characters (e.g. `2026-02-24T10:00:00Z-a3f1`).
- `name` -- The item name as the user provided it, with leading/trailing whitespace trimmed. Preserve original casing for display purposes.
- `normalizedName` -- Always computed as `name.toLowerCase().trim()`. Recompute on every add or edit. This field is used for all matching and deduplication logic.
- `quantity` -- Optional. Defaults to `null` when the user does not specify a quantity. When present, must be a number greater than 0. Fractional values are fine (e.g. 0.5 for half a pound).
- `unit` -- Optional free text (e.g. "gallons", "lbs", "bunch", "bag"). Defaults to `null` when not specified.
- `category` -- One of the values from the `categories` array, or "Uncategorized".
- `checkedOff` -- Boolean. Starts as `false`. Set to `true` when the user checks off the item.
- `checkedOffDate` -- ISO timestamp when the item was checked off. `null` when not checked off.
- `addedBy` -- Lowercase string from `config.json` user field (e.g. "aj" or "shal").
- `addedDate` -- ISO timestamp when the item was first added.
- `notes` -- Optional free text for special instructions ("the organic one", "Costco size"). Defaults to `null`.
- `categories` (top-level array) -- The master list of known categories. Starts with 8 presets. Custom categories are appended when created.
- `lastModified` -- ISO timestamp. Updated on every write to this file.

### data/config.json

Stores session-persistent user configuration.

```json
{ "user": "aj", "snoozes": {} }
```

- `user` -- Set on first interaction, persists across sessions. Lowercase string.
- `snoozes` -- Reserved for Phase 2 restock suggestions. Ignore for now; preserve the field when writing.

### data/history-YYYY-MM.json

Monthly archive of purchased items. One file per calendar month, created on demand.

```json
{
  "month": "2026-02",
  "archivedItems": [
    { "...same fields as active item...", "archivedDate": "2026-02-25T08:00:00Z" }
  ]
}
```

The `archivedDate` field is added to each item when it moves from active to history. All original fields from the active item are preserved. A new file is created automatically when the first item is archived in a month that does not yet have a history file.

## Core Operations

### Adding Items

Parse the user's natural language into name, quantity, and unit for each item.

**Category inference:** Silently infer the category from the item name. Common mappings: milk/cheese/yogurt/butter go to Dairy, chicken/beef/fish to Meat, bananas/lettuce/onions to Produce, rice/pasta/flour to Pantry, dish soap/paper towels to Household, shampoo/toothpaste to Personal, beer/wine/juice to Beverages, frozen pizza/ice cream to Frozen. If the mapping is not obvious, assign "Uncategorized". Never prompt the user for a category. If the user wants to change it, they can say "move X to Y category".


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ajeenkya/shopping-list-skill](https://github.com/ajeenkya/shopping-list-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
