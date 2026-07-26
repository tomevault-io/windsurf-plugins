---
trigger: always_on
description: Excel-VBA-Dicts is a VBA library that brings **functional programming** idioms (map, filter, reduce, groupBy, leftJoin, etc.) to Excel/VBA. It implements native data structures (Dicts, Lists, HashSets, TreeSets) entirely in VBA, with full macOS-Excel compatibility, JSON support, and spreadsheet I/O.
---

# CLAUDE.md — Excel-VBA-Dicts

## Project Overview

Excel-VBA-Dicts is a VBA library that brings **functional programming** idioms (map, filter, reduce, groupBy, leftJoin, etc.) to Excel/VBA. It implements native data structures (Dicts, Lists, HashSets, TreeSets) entirely in VBA, with full macOS-Excel compatibility, JSON support, and spreadsheet I/O.

A companion Python module (`xlDataStructure.py`) provides an equivalent `xlDict` class using `win32com` for automation on Windows.

---

## Repository Structure

```
Excel-VBA-Dicts/
├── Dicts.bas          # Core dictionary class (TreeMap-like, sorted keys)
├── Lists.bas          # Dynamic array list with functional ops
├── HashSets.bas       # Hash set backed by Lists
├── TreeSets.bas       # Sorted set (BST) backed by Nodes
├── Nodes.bas          # BST node used by TreeSets
├── util.bas           # Utility: filesystem workbook iteration helper
├── Test.bas           # Unit tests using Debug.Assert
├── xlDataStructure.py # Python xlDict class (win32com automation)
├── README.md
└── LICENSE            # MIT
```

All `.bas` files are VBA class/module files meant to be imported into an Excel workbook via the VBA IDE (Alt+F11 → File → Import File).

---

## Module Descriptions

### `Dicts.bas` — Core Dictionary

A sorted dictionary (TreeMap-like) using `HashSets` for keys and a `Collection` for values.

**Dependencies:** `Lists`, `HashSets`, `Nodes`
**Author:** Xiou Yang
**Last updated:** 20.04.2023

**Key enums:**
```vba
Enum ProcessWith
    key = 0
    value = 1
    RangedValue = 2
End Enum

Enum AggregateMethod
    AggMap = 0
    AggReduce = 1
    Aggfilter = 2
End Enum
```

**Core API:**

| Method/Property | Description |
|---|---|
| `add(k, v)` | Add key-value pair; returns `Me` (chainable) |
| `Item(key)` | Get/set by key (default property) |
| `exists(key)` | Check if key exists |
| `Remove(e)` / `RemoveAll()` / `clear()` | Remove entries |
| `Count` | Number of entries (optional recursive) |
| `keysArr` / `valsArr` | Keys/values as arrays |
| `label` / `setLabel(rng)` | Column labels (for DataFrame-like usage) |
| `getByLabel(k, label)` | Retrieve element by key + column label |

**Functional operations (inline expression syntax):**

| Method | Description |
|---|---|
| `map(operation)` | Transform values using string expression |
| `mapKey(operation)` | Transform keys |
| `filter(operation)` | Filter by value |
| `filterKey(operation)` | Filter by key |
| `reduce(operation, initialVal)` | Reduce values |
| `reduceKey(operation, initialVal)` | Reduce keys |
| `ranged(operation, aggregate)` | Windowed/rolling aggregation |
| `groupBy(attr, valCol)` | Group by attribute |
| `leftJoin(dict2)` | Left join two dicts |
| `diff(dict2)` | Keys in self not in dict2 |
| `union(dict2)` | Merge (keep original values by default) |
| `intersect(dict2)` | Keys in both |
| `mergeMap(op, other)` | Element-wise binary operation across two dicts |
| `sort(isAscending)` | Sort by key |

**Callback variants (for complex logic):**

| Method | Description |
|---|---|
| `mapX(callback)` | map with external VBA function |
| `filterX(callback)` | filter with external VBA function |
| `reduceX(callback, initVal)` | reduce with external VBA function |
| `productX(other, callback)` | cartesian product with callback |

**Spreadsheet I/O:**

| Method | Description |
|---|---|
| `load(sht, keyCol, valCol, ...)` | Load from worksheet (vertical) |
| `loadH(sht, keyRow, valRow, ...)` | Load from worksheet (horizontal) |
| `loadStruct(sht, keyCol1, keyCol2, valCol, ...)` | Load 2-level keyed structure |
| `unload(sht, keyPos, ...)` | Write values back to sheet |
| `dump(sht, ...)` | Write keys + values to sheet |
| `fromRng(rng)` | Load from a Range object |
| `fromMatrix(l)` | Build from 2D Lists |
| `fromArray(arr)` | Build from array/Lists/Collection |

**Utilities:**

| Method | Description |
|---|---|
| `p()` | Debug.Print the dict |
| `pk()` | Debug.Print keys only |
| `toString()` | String representation |
| `toJSON(exportTo)` | Serialize to JSON |
| `fromString(s)` | Deserialize from JSON-like string |
| `rng(start, ending, steps)` | Generate a range array |
| `x(sht, row)` / `y(sht, col)` | Get last used column/row on sheet |
| `reg(pattern, flag)` | Create a VBScript RegExp object |
| `frequencyCount(rng)` | Count occurrences |
| `feed(d, isIncremental)` | Merge another dict's values |
| `reset(v)` | Reset all values to a constant |
| `nulls(toVal)` | Handle null values |
| `isDict(obj)` / `isInstanceOf(obj, type)` | Type checking |

---

### `Lists.bas` — Dynamic Array List

A dynamically resizing array with functional programming support.

**Last updated:** 13.08.2024

**Core API:**

| Method/Property | Description |
|---|---|
| `add(ele)` | Append element; returns `Me` |
| `addAll(arr)` | Append all from array/collection |
| `addAt(ele, index)` | Insert at index |
| `remove(ele)` / `removeAt(index)` | Remove element |
| `getVal(index, index2)` | Get element (supports 2D indexing) |
| `setVal(index, ele)` | Set element at index |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [6234456/Excel-VBA-Dicts](https://github.com/6234456/Excel-VBA-Dicts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
