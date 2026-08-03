---
trigger: always_on
description: Dense, recipe-shaped reference for agents using this library. For
---

# AGENTS.md — PBIXRay for LLM agents

Dense, recipe-shaped reference for agents using this library. For
narrative docs see [README.md](README.md); for internals see
[docs/](docs/).

## What this is

Read-only Python parser for Power BI `.pbix` files, Excel `.xlsx`
workbooks with embedded Power Pivot models, and Analysis Services
`.abf` backups. Returns pandas DataFrames. No network, no Power BI /
Excel install required.

## Install + minimal example

```bash
pip install pbixray
```

```python
from pbixray import PBIXRay

model = PBIXRay("data/Adventure Works DW 2020.pbix")  # or .xlsx / .abf
print(model.tables)                  # list of table names
print(model.schema.head())           # column metadata
print(model.get_table(model.tables[0]).head())
```

File type is auto-detected from contents, not extension. Same API for
all three formats. An `.abf` is the same data model as a `.pbix`
without the zip envelope; multi-partition tables (classic SSAS
partitioning and incremental-refresh partitions) are decoded in full —
`get_table` concatenates every partition in storage order.

## Memory model / large files

By default the whole decompressed data model is held in one in-memory
buffer for the life of the `PBIXRay` object, and metadata is loaded
lazily on first property access. For models that approach or exceed host
RAM, construct with `on_disk=True` (optionally `temp_dir=...`): the
decompressed data is streamed to a temp file and `mmap`-ed, so only the
pages a requested table touches are resident. (When the container's
`DataModel` member is stored uncompressed — a raw ABF inside the zip —
`on_disk=True` serves it directly from the `.pbix`/`.xlsx` with no
temp-file copy at all.) `PBIXRay` is a context manager; use
`with PBIXRay(path, on_disk=True) as model:` or call `model.close()` to
release the mapping and temp file deterministically. After `close()`,
`get_table`/`iter_table` and any metadata not yet loaded raise
`RuntimeError`; DataFrames already materialized remain usable.

Per-table levers:

- `get_table(name, columns=[...])` decodes only the listed columns.
- `get_table(name, strings_as_categorical=True)` returns string columns
  as `pd.Categorical` — each distinct value stored once, not once per
  row.
- `iter_table(name, chunk_size=..., columns=[...])` streams the table
  as DataFrame chunks instead of materialising it whole (see below).

Dictionary decode runs on a native Huffman kernel
([xmhuffman](https://github.com/Hugoberry/xmhuffman-cython)) and fans
out across cores automatically for large dictionaries.

## Decision tree — "I want X → use Y"

| I want…                              | Use                                                |
| ------------------------------------ | -------------------------------------------------- |
| List of table names                  | `model.tables`                                     |
| Row data of one table                | `model.get_table(name)`                            |
| Row data, too big to hold at once    | `model.iter_table(name, chunk_size=...)`           |
| Column types per table               | `model.schema`                                     |
| DAX measures                         | `model.dax_measures`                               |
| DAX calculated columns               | `model.dax_columns`                                |
| DAX calculated tables                | `model.dax_tables`                                 |
| M / Power Query source               | `model.power_query`, `model.m_parameters`          |
| M from DirectQuery / native-SQL models | `model.mashup_queries`, `model.data_mashup`      |
| Relationships                        | `model.relationships`                              |
| Aggregations ("Manage aggregations") | `model.aggregations`                               |
| Row-Level Security                   | `model.rls`                                        |
| Object-Level Security                | `model.ols`                                        |
| Perspectives (consolidated members)  | `model.perspectives`                               |
| Report's data-connection manifest    | `model.connections`                                |
| Model build / locale metadata        | `model.metadata`                                   |
| Per-column size breakdown            | `model.statistics`                                 |
| Total model size (bytes, int)        | `model.size`                                       |
| Raw Analysis Services DMV-equivalents| `model.tmschema_*` (40 properties; PBIX/ABF only)  |

## API surface

Source of truth: [pbixray/core.py](pbixray/core.py).

### Core endpoints

| Attribute              | Return              | Notable columns / shape                                                                                                  |
| ---------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `tables`               | `numpy.ndarray[str]`| Table names                                                                                                              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hugoberry/pbixray](https://github.com/Hugoberry/pbixray) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
