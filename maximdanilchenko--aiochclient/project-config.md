---
trigger: always_on
description: Guidance for working in this repository.
---

# CLAUDE.md

Guidance for working in this repository.

## What this is

`aiochclient` — an async HTTP(S) ClickHouse client for Python. It talks to ClickHouse
over its HTTP interface (default `http://localhost:8123/`), converts types in both
directions, streams results, and decodes rows lazily. The public API is a single class,
`ChClient`, plus the `Record` row object and the `ChClientError` exception.

It has three interchangeable wire engines, selected on `ChClient`:
- **TSV** (default) — text `TSVWithNamesAndTypes`, lazy per-row decoding.
- **RowBinary** (`binary=True`) — the row-oriented binary format; faster, best for
  streaming row-by-row via `iterate`.
- **Native** (`native=True`) — ClickHouse's column-oriented binary format; fastest for
  bulk SELECT/INSERT, decoded whole-column-at-once without numpy.

Package is published to PyPI as `aiochclient`. Current version: `2.8.0` (see `setup.py`).

## Architecture

The package lives in `aiochclient/`:

- **`client.py`** — `ChClient`, the public entry point. Holds connection params/headers,
  builds query strings, picks the right ClickHouse `FORMAT`, and orchestrates fetch/insert.
  Constructor flags: `binary` / `native` (engine), `insert_block_size` (Native INSERT block
  size; 0 = single atomic block). Public methods: `execute`, `fetch`, `fetchrow`, `fetchval`,
  `iterate`, `insert_file`, `is_alive`, `close`. `fetchone`/`cursor` are deprecated aliases.
- **`http_clients/`** — pluggable HTTP backend behind `HttpClientABC` (`abc.py`).
  `choose_http_client()` auto-detects whether the passed session is an `aiohttp.ClientSession`
  or `httpx.AsyncClient` and returns the matching impl (`aiohttp.py` / `httpx.py`). Both
  implement `get` / `post_return_lines` (streaming line generator, TSV/JSON) /
  `post_return_bytes` (streaming byte-chunk generator, RowBinary/Native) / `post_no_return`
  (also accepts an async-generator body for streamed Native INSERT) / `close`. Non-200
  responses raise `ChClientError`.
- **`types.py`** — pure-Python type conversion (the TSV engine and the per-value binary
  read/write primitives). `BaseType` subclasses (one per ClickHouse type) implement `convert`
  (CH bytes → Python), `unconvert` (Python → CH text), and the RowBinary `read`/`write`.
  `CH_TYPES_MAPPING` maps CH type names → classes; `PY_TYPES_MAPPING` maps Python types →
  unconvert fns. Containers parse recursively via `what_py_type`. Also holds the binary
  `Cursor` (with bulk column reads) and `read_column`.
- **`_types.pyx`** — Cython port of `types.py` for a speedup. **Keep in sync with `types.py`**
  — both must implement the same behavior. Also hosts the compiled `Record`,
  `build_records` (column→row transpose), `write_string_column`, and `record_new`. Everything
  imports from `_types` (compiled) and falls back to `types` if the C extension isn't built.
- **`binary.py`** — the **RowBinary** engine. `BinaryReader` (buffered O(n) reader over the
  byte-chunk stream), `RowBinaryFabric` / `rows_from_binary` (read), `rows_to_binary` (write),
  `fetch_column_types` / `fetch_column_header` (LIMIT 0 type probe for INSERTs). Imports the
  `Cursor`, `read_row`, `read_column`, `what_py_type` from `_types`/`types`.
- **`native.py`** — the **Native** (columnar) engine. Read: `decode_column` (whole-column
  decode; bulk numerics via stdlib `array`, strings/dates via the Cursor, `_read_prefix` for
  LowCardinality state), `blocks_from_native` (yields a `Record` list per block, GC suppressed
  during the synchronous decode/build), `rows_from_native`. Write: `encode_column` (bulk
  column encoders), `rows_to_native` (one block) and `rows_to_native_stream` (async generator
  of blocks for streamed INSERT). `_wire_type` simplifies LowCardinality/Nested for the wire.
- **`records.py`** — `Record` (lazy `Mapping`, access by name or index, decodes on first
  access). Re-exports the compiled `Record` + `record_from_decoded` from `_types` when built,
  else defines the pure version. `RecordsFabric` (TSV header rows), `FromJsonFabric` (JSON).
- **`sql.py`** — patches `sqlparse`'s lexer/keywords so ClickHouse-specific syntax
  (`FORMAT`, `DESCRIBE`, `SHOW`, `EXISTS`) parses correctly. Imported for its side effects;
  `client.py` uses `sqlparse` to detect statement type and format.
- **`exceptions.py`** — `ChClientError`, plus `NeedMoreData` (internal, drives the binary
  reader's refill loop).

### Request flow

`_execute()` in `client.py` is the core. It substitutes `params` into the query string,
parses the statement (`_parse_squery`) to decide whether to fetch and which `FORMAT` to
append (`Native` / `RowBinaryWithNamesAndTypes` / `TSVWithNamesAndTypes`, or `JSONEachRow`
when `json=True`). Fetches: the columnar/binary engines stream byte chunks via
`post_return_bytes` and drive `rows_from_native` / `rows_from_binary`; TSV/JSON stream lines
via `post_return_lines`. `fetch` has a fast path that consumes Native results block-by-block.
INSERTs: Native encodes columnar blocks (streamed via `rows_to_native_stream`), RowBinary
encodes via `rows_to_binary` (both after a LIMIT 0 type probe), TSV via `rows2ch`, JSON via
`json2ch`. All public fetch methods delegate to `_execute`.

## Build / test / dev commands

```
make format                  # isort + black

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maximdanilchenko/aiochclient](https://github.com/maximdanilchenko/aiochclient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
