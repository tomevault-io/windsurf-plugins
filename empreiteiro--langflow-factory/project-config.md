---
trigger: always_on
description: Dynamic output in Langflow components can be significantly affected by changing the number of rows in a `TableInput`. This behavior allows components to automatically adapt to data volume, showing relevant outputs based on the amount of available information.
---

# Guide: Dynamic Output with Table Input

## Overview

Dynamic output in Langflow components can be significantly affected by changing the number of rows in a `TableInput`. This behavior allows components to automatically adapt to data volume, showing relevant outputs based on the amount of available information.

## How It Works

### 1. `update_outputs` Method

The `update_outputs` method is the key to implementing dynamic output. It is automatically called when a field with `real_time_refresh=True` is modified.

```python
def update_outputs(self, frontend_node: dict, field_name: str, field_value: Any) -> dict:
    """Updates outputs dynamically based on the number of rows in the table input."""
    if field_name == "data_table":
        # Gets the number of rows in the table
        row_count = len(field_value) if field_value else 0
        
        # Clears existing outputs
        frontend_node["outputs"] = []
        
        # Adds outputs based on the number of rows
        if row_count == 0:
            # No rows - shows only warning
            frontend_node["outputs"].append(
                Output(display_name="Warning", name="warning", method="show_warning")
            )
        elif row_count == 1:
            # One row - shows item details
            frontend_node["outputs"].append(
                Output(display_name="Item Details", name="item_details", method="show_item_details")
            )
        # ... more conditions
        
    return frontend_node
```

### 2. TableInput Configuration

For dynamic output to work, the `TableInput` must have `real_time_refresh=True`:

```python
TableInput(
    name="data_table",
    display_name="Data Table",
    info="Add or remove rows to see how outputs change dynamically.",
    table_schema=[...],
    value=[...],
    real_time_refresh=True,  # Important!
),
```

## Dynamic Output Strategies

### 1. Based on Number of Rows

| Number of Rows | Suggested Outputs | Justification |
|----------------|------------------|---------------|
| 0 | Warning | No data to process |
| 1 | Item Details | Specific information for the single item |
| 2-5 | Summary + Basic Statistics | Simple analysis suitable for the volume |
| 6-10 | Summary + Advanced Statistics + Category Analysis | More detailed analysis |
| 10+ | All available outputs | Complete analysis and reports |

### 2. Based on Data Type

```python
def update_outputs(self, frontend_node: dict, field_name: str, field_value: Any) -> dict:
    if field_name == "data_table":
        # Analyzes the type of data in rows
        has_numeric_data = any(
            str(item.get("value", "")).isdigit() 
            for item in field_value
        )
        
        if has_numeric_data:
            frontend_node["outputs"].append(
                Output(display_name="Statistics", name="stats", method="generate_stats")
            )
        else:
            frontend_node["outputs"].append(
                Output(display_name="Text Analysis", name="text_analysis", method="analyze_text")
            )
    
    return frontend_node
```

### 3. Based on Data Complexity

```python
def update_outputs(self, frontend_node: dict, field_name: str, field_value: Any) -> dict:
    if field_name == "data_table":
        # Counts unique categories
        categories = set(item.get("category", "") for item in field_value)
        
        if len(categories) > 3:
            frontend_node["outputs"].append(
                Output(display_name="Category Analysis", name="category_analysis", method="analyze_categories")
            )
        
        # Checks if there are numeric values
        numeric_values = [item for item in field_value if str(item.get("value", "")).isdigit()]
        if len(numeric_values) > 5:
            frontend_node["outputs"].append(
                Output(display_name="Statistical Analysis", name="statistical_analysis", method="analyze_statistics")
            )
    
    return frontend_node
```

## Practical Examples

### Example 1: Sales Analysis Component

```python
class SalesAnalysisComponent(Component):
    inputs = [
        TableInput(
            name="sales_data",
            display_name="Sales Data",
            table_schema=[
                {"name": "product", "display_name": "Product", "type": "str"},
                {"name": "quantity", "display_name": "Quantity", "type": "str"},
                {"name": "price", "display_name": "Price", "type": "str"},
                {"name": "category", "display_name": "Category", "type": "str"},
            ],
            real_time_refresh=True,
        )
    ]
    
    def update_outputs(self, frontend_node: dict, field_name: str, field_value: Any) -> dict:
        if field_name == "sales_data":
            row_count = len(field_value) if field_value else 0
            
            frontend_node["outputs"] = []
            
            # Always shows processed data
            frontend_node["outputs"].append(
                Output(display_name="Processed Data", name="processed_data", method="process_data")
            )
            

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Empreiteiro/langflow-factory](https://github.com/Empreiteiro/langflow-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
