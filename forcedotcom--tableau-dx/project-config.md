---
trigger: always_on
description: Rules for using the Tableau Next MCP server with semantic models
---


# Tableau Next MCP — Agent Rules

You have access to a **Tableau Next MCP server** that lets you interact with Salesforce Data Cloud Semantic Models and Tableau Next assets. Follow these rules when the user asks you to work with semantic models, dashboards, visualizations, or analytics data.

## Local Model File Structure

Semantic models retrieved by the **Salesforce Semantic DX** extension are stored as a folder of split JSON files under `Semantic Models/<Model Label>/`:

| File | Contents | Wrapper key |
|---|---|---|
| `model.json` | Model metadata (apiName, label, dataspace, businessPreferences, baseModels, etc.) | _(root object)_ |
| `dataObjects.json` | Data objects / entities | `{ "items": [...] }` |
| `relationships.json` | Relationships between objects | `{ "items": [...] }` |
| `calculatedDimensions.json` | Calculated dimension fields | `{ "items": [...] }` |
| `calculatedMeasurements.json` | Calculated measure fields | `{ "items": [...] }` |
| `groupings.json` | Groupings | `{ "groupings": [...] }` |
| `logicalViews.json` | Logical views | `{ "items": [...] }` |
| `parameters.json` | Parameters | `{ "items": [...] }` |
| `dimensionHierarchies.json` | Dimension hierarchies | `{ "items": [...] }` |
| `metrics.json` | Metrics / KPIs | `{ "items": [...] }` |
| `fieldsOverrides.json` | Field overrides | `{ "items": [...] }` |
| `modelFilters.json` | Model-level filters | `{ "items": [...] }` |
| `modelInfo.json` | Definition counts and hierarchy depth | _(root object)_ |

If the model extends base models, base entity files live under `base/<Base Label>/` with the same file names.

## Assembling the Full Model Object

Before calling `refine_semantic_model`, you **must** assemble the full model from the split files:

1. Read `model.json` — this is the base object.
2. Attach each entity array onto the base using the API key names:
   - `dataObjects.json` → `semanticDataObjects`
   - `relationships.json` → `semanticRelationships`
   - `calculatedDimensions.json` → `semanticCalculatedDimensions`
   - `calculatedMeasurements.json` → `semanticCalculatedMeasurements`
   - `groupings.json` → `semanticGroupings`
   - `logicalViews.json` → `semanticLogicalViews`
   - `parameters.json` → `semanticParameters`
   - `dimensionHierarchies.json` → `semanticDimensionHierarchies`
   - `metrics.json` → `semanticMetrics`
   - `modelInfo.json` → `semanticModelInfo`
   - `fieldsOverrides.json` → `fieldsOverrides`
   - `modelFilters.json` → `semanticModelFilters`
3. For each entity file, unwrap the array from its wrapper key (`items` or `groupings`).
4. If `model.json` has `baseModels`, also read entity files from `base/<label>/` and merge them into the same arrays.

## Identifying the Target Model and Org

There may be **multiple models** in the workspace under `Semantic Models/`, potentially from **different Salesforce orgs**. Before doing any work:

1. List the subdirectories of `Semantic Models/` to discover available local models.
2. If the user's request clearly names or implies a specific model, use that one.
3. If there is **only one** model folder, use it automatically.
4. If there are **multiple models** and it's ambiguous which one the user means, **ask the user** to choose before proceeding.
5. When calling MCP tools that require `idOrApiName`, use the model's `apiName` from its `model.json`.

### Setting the Correct Org

Each model folder contains `metadata/orgInfo.json` with the org it was retrieved from:
```json
{ "orgId": "00D...", "instanceUrl": "https://...", "username": "user@org.com", "alias": "myorg" }
```

**Before calling any MCP tools** for a model, you must:
1. Read `metadata/orgInfo.json` from the model's folder.
2. Call `set_org` with the `username` from that file.
3. This ensures the MCP server targets the correct Salesforce org.

If the model has no `metadata/orgInfo.json`, skip `set_org` — the MCP server will use the default org.

## Using `refine_semantic_model`

This is the **primary tool** for making AI-driven changes to semantic models.

### Input
- `semanticModel` — the **full** assembled model object (see above).
- `utterance` — a natural-language description of the change (e.g., "Add a profit margin calculated field").
- `additionalInstructions` _(optional)_ — extra constraints to guide the AI.

### Output
- `semanticModel` — the **complete updated** model object with changes applied.
- `explanation` — a human-readable summary of what changed.

### Applying Changes — Always Ask First

**Do NOT automatically write changes back to the local files.** After receiving the refined model:

1. Present the `explanation` to the user so they understand what the AI changed.
2. Summarize the key differences (new fields, modified expressions, removed items, etc.).
3. **Ask the user for confirmation** before applying — e.g., "Would you like me to apply these changes to your local model files?"
4. Only after the user confirms, split the returned model back into the local files:
   - Separate metadata from entity arrays using the same key mapping above.
   - Wrap each entity array back into its file format (`{ "items": [...] }` or `{ "groupings": [...] }`).
   - Write each file to the model folder, preserving pretty-printed JSON (`JSON.stringify(data, null, 2)`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/forcedotcom) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
