---
trigger: always_on
description: Use this to write better GAQL queries
---

# Google Ads Query Language (GAQL) Guidelines

## Overview

The Google Ads Query Language (GAQL) is a powerful tool for querying the Google Ads API that allows you to retrieve:

1. **Resources** and their related attributes, segments, and metrics using `GoogleAdsService.Search` or `GoogleAdsService.SearchStream`
2. **Metadata** about available fields and resources using `GoogleAdsFieldService`

## Field Categories

Understanding field categories is essential for building effective GAQL queries:

1. **RESOURCE**: Represents a primary entity (e.g., `campaign`, `ad_group`) that can be used in the FROM clause
2. **ATTRIBUTE**: Properties of a resource (e.g., `campaign.id`, `campaign.name`). Including these may segment results depending on the resource relationship
3. **SEGMENT**: Fields that always segment search queries (e.g., `segments.date`, `segments.device`)
4. **METRIC**: Performance data fields (e.g., `metrics.impressions`, `metrics.clicks`) that never segment search queries

## Query Structure

A GAQL query consists of the following components:

```
SELECT
  <field_1>,
  <field_2>,
  ...
FROM <resource>
WHERE <condition_1> AND <condition_2> AND ...
ORDER BY <field_1> [ASC|DESC], <field_2> [ASC|DESC], ...
LIMIT <number_of_results>
```

### SELECT Clause

The `SELECT` clause specifies the fields to return in the query results:

```
SELECT
  campaign.id,
  campaign.name,
  metrics.impressions,
  segments.device
```

Only fields that are marked as `selectable: true` in the `GoogleAdsField` metadata can be used in the SELECT clause.

### FROM Clause

The `FROM` clause specifies the primary resource type to query from. Only one resource can be specified, and it must have the category `RESOURCE`.

```
FROM campaign
```

### WHERE Clause (optional)

The `WHERE` clause specifies conditions to filter the results. Only fields marked as `filterable: true` in the `GoogleAdsField` metadata can be used for filtering.

```
WHERE 
  campaign.status = 'ENABLED'
  AND metrics.impressions > 1000
  AND segments.date DURING LAST_30_DAYS
```

### ORDER BY Clause (optional)

The `ORDER BY` clause specifies how to sort the results. Only fields marked as `sortable: true` in the `GoogleAdsField` metadata can be used for sorting.

```
ORDER BY metrics.impressions DESC, campaign.id
```

### LIMIT Clause (optional)

The `LIMIT` clause restricts the number of results returned.

```
LIMIT 100
```

## Field Metadata Exploration

To explore available fields and their properties, use the `GoogleAdsFieldService`:

```
SELECT
  name,
  category,
  selectable,
  filterable,
  sortable,
  selectable_with,
  attribute_resources,
  metrics,
  segments,
  data_type,
  enum_values,
  is_repeated
WHERE name = "campaign.id"
```

Key metadata properties to understand:

- **`selectable`**: Whether the field can be used in a SELECT clause
- **`filterable`**: Whether the field can be used in a WHERE clause
- **`sortable`**: Whether the field can be used in an ORDER BY clause
- **`selectable_with`**: Lists resources, segments, and metrics that are selectable with this field
- **`attribute_resources`**: For RESOURCE fields, lists the resources that are selectable with this resource and don't segment metrics
- **`metrics`**: For RESOURCE fields, lists metrics that are selectable when this resource is in the FROM clause
- **`segments`**: For RESOURCE fields, lists fields that segment metrics when this resource is used in the FROM clause
- **`data_type`**: Determines which operators can be used with the field in WHERE clauses
- **`enum_values`**: Lists possible values for ENUM type fields
- **`is_repeated`**: Whether the field can contain multiple values

## Data Types and Operators

Different field data types support different operators in WHERE clauses:

### String Fields
- `=`, `!=`, `IN`, `NOT IN`
- `LIKE`, `NOT LIKE` (case-sensitive string matching)
- `CONTAINS ANY`, `CONTAINS ALL`, `CONTAINS NONE` (for repeated fields)

### Numeric Fields
- `=`, `!=`, `<`, `<=`, `>`, `>=`
- `IN`, `NOT IN`

### Date Fields
- `=`, `!=`, `<`, `<=`, `>`, `>=`
- `DURING` (with named date ranges)
- `BETWEEN` (with date literals)

### Enum Fields
- `=`, `!=`, `IN`, `NOT IN`
- Values must match exactly as listed in `enum_values`

### Boolean Fields
- `=`, `!=`
- Values must be `TRUE` or `FALSE`

## Date Ranges

### Literal Date Ranges
```
WHERE segments.date BETWEEN '2020-01-01' AND '2020-01-31'
```

### Named Date Ranges
```
WHERE segments.date DURING LAST_7_DAYS
WHERE segments.date DURING LAST_14_DAYS
WHERE segments.date DURING LAST_30_DAYS
WHERE segments.date DURING LAST_90_DAYS
WHERE segments.date DURING THIS_MONTH
WHERE segments.date DURING LAST_MONTH
WHERE segments.date DURING THIS_QUARTER
```

### Date Functions
```
WHERE segments.date = YESTERDAY
WHERE segments.date = TODAY
```

## Case Sensitivity Rules

1. **Field and resource names**: Case-sensitive (`campaign.id` not `Campaign.Id`)
2. **Enumeration values**: Case-sensitive (`'ENABLED'` not `'enabled'`)
3. **String literals in conditions**:
   - Case-insensitive by default (`WHERE campaign.name = 'brand campaign'`)
   - Use `LIKE` for case-sensitive matching (`WHERE campaign.name LIKE 'Brand Campaign'`)

## Ordering and Limiting Results

### Ordering
- Results can be ordered by one or more fields

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cohnen/mcp-google-ads](https://github.com/cohnen/mcp-google-ads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
