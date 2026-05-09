---
trigger: always_on
description: Momen.app's database architecture and GraphQL schema generation rules.
---


# Momen.app Database to GraphQL Schema Mapping

The GraphQL schema is automatically generated directly from the PostgreSQL data model.

> **CRITICAL CATCH-ALL RULE:**
> If you are ever in doubt about the exact structure, available fields, Enum values (like `[Enum:ROUNDING_MODE]`), or specific arguments for an endpoint, **you must use the webfetch/curl tool to introspect the live GraphQL schema** via the provided endpoint URL before making assumptions.
 

## 1. Naming Conventions & Root Operations

Each table generates corresponding root query, mutation, and subscription fields. For a table named `[table]`:

### Queries
*   `[table]`: Fetch lists (supports `where`, `order_by`, `distinct_on`, `limit`, `offset`).
*   `[table]_by_pk`: Fetch single record by primary key (`id`).
*   `[table]_aggregate`: Aggregate queries (`count`, `sum`, `avg`, `min`, `max`).
*   `[table]_group_by`: Group records based on specified fields.
*   `fz_[table]_by_[column]`: Auto-generated spatial proximity search if the table contains a `geo_point` column.
*   **Relay API**: If configured, generates a cursor-based pagination query returning a `Connection_[table]` object.

### Mutations
*   `insert_[table]`, `insert_[table]_one`: Create records. Supports nested inserts and `on_conflict` (requires `constraint` and `update_columns`).
*   `update_[table]`, `update_[table]_by_pk`: Modify records. Uses `_set` (replace) and `_inc` (increment numeric). `where` is required (`!`) for bulk updates. *(Note: Hasura JSONB update operators like `_append`/`_delete_key` are not supported).*
*   `delete_[table]`, `delete_[table]_by_pk`: Remove records. `where` is required (`!`) for bulk deletes.
*   `export_[table]`: Trigger a data export task for the table.

### Subscriptions
*   `[table]`, `[table]_by_pk`, `[table]_aggregate`: Live queries mirroring their standard query counterparts.

## 2. Column Types and Data Mappings

### Primitive Types
*   `text` → `String`, `integer` → `Int`, `bigint`, `bigserial` → `bigint`, `float8` → `Float8`, `decimal` → `Decimal`, `boolean` → `Boolean`, `jsonb` → `jsonb`.
*   **Time/Date**: `timestamptz`, `timetz`, `date`, `interval`.
*   **Geo**: `geo_point` → `geography`.

### Composite (Media) Types
Media columns (`image`, `file`, `video`) are structurally 1:N relations but act as fields. 
*   **Single Media**: Stored as `[column]_id` (e.g., `cover_image_id`) referencing system asset tables (`FZ_Image`, `FZ_File`, `FZ_Video`).
*   **Media Lists**: Types like `image_list`, `video_list`, `file_list` map to GraphQL Arrays and are stored as `[column]_ids` (e.g., `gallery_images_ids`).

### System-Managed Columns
`id`, `created_at`, `updated_at` are read-only and automatically managed. They cannot be used in mutation inputs (`_set`, `_inc`, insert inputs).

## 3. Relationships

Relationships are defined by foreign keys and determine GraphQL nested fields:
*   **1:1 (One-to-One)**: Yields a single nested object (e.g., `meta: post_meta`).
*   **1:N (One-to-Many)**: Yields an array (e.g., `post_tags: [post_tag]`) and an aggregate object (e.g., `post_tags_aggregate`).

## 4. Filtering (`where` clauses)

Filters rely on `[table]_bool_exp`.

### Logical Operators
*   `_and: [bool_exp]`, `_or: [bool_exp]`, `_not: bool_exp`

### Relation Filters
Navigate relationships using the relationship field name directly. The value is a nested `[related_table]_bool_exp` object.
*   **To-One Relationships (1:1, N:1)**: Filters the parent record based on the single related record's fields.
    *(e.g., Find posts where the author's name is "John": `author: { name: { _eq: ... } }`)*
*   **To-Many Relationships (1:N, N:M)**: Uses `EXISTS` semantics. The parent record is returned if **any** related record in the array matches the nested condition.
    *(e.g., Find posts that have at least one tag named "Tech": `post_tags: { tag: { name: { _eq: ... } } }`)*

### Comparison Predicates (Strict Pattern)
Momen uses a strict **Operator-First Pattern**. A predicate must start with the operator. If it's a generic operator, the operand wrapper type must match the *final evaluated type*, not necessarily the column type.

**Structure:**
```json
{
  "_operator": {
    "operand_type": {
      "left_operand": { ... },
      "right_operand": { ... }
    }
  }
}
```

*   **Operators:** 
    *   **Comparison (Generic):** `_eq`, `_neq`, `_gt`, `_lt`, `_gte`, `_lte`
    *   **Array (Generic):** `_in`, `_nin`
    *   **Nullity (Generic, Unary):** `_is_null`, `_is_not_null`
    *   **Text (String Pattern):** `_like`, `_nlike`, `_ilike`, `_nilike`, `_similar`, `_nsimilar`
    *   **JSONB:** `_contains`, `_contained_in`, `_has_key`, `_has_keys_any`, `_has_keys_all`
    *   **Boolean:** `_is_true`, `_is_false`
    *   **Collection:** `_is_empty`, `_is_not_empty`
*   **Operand Definitions (`left_operand` / `right_operand`):**
    *   **Literal**: `{"literal": value}`
    *   **Column**: `{"column": "field_name"}`
    *   **Function**: `{"function_name": { ...args }}`
*   **Operand Types (Determined by Final Value):** `bigint_operand`, `text_operand`, `boolean_operand`, `timestamptz_operand`, etc.

**Example Predicate (Extract Month from Timestamp and check if = 12):**
```json
{
  "_eq": {
    "bigint_operand": {
      "left_operand": {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [momen-tech-org/momen-cursor-rules](https://github.com/momen-tech-org/momen-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
