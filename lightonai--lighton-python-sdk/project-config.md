---
trigger: always_on
description: Design decisions for the LightOn Python SDK. Read before changing architecture.
---

# AGENTS.md

Design decisions for the LightOn Python SDK. Read before changing architecture.

> **Maintenance rule:** whenever a change alters file architecture (moving/adding/
> removing modules or packages) or a design-pattern decision recorded here, update
> this file in the **same** change, keep it in sync, don't defer it. If a change
> contradicts a decision below, edit the decision and its rationale, don't just append.

## Layout

```
lighton/
  __init__.py        # public exports: LightOn, LightOnConfiguration, Workspace
  _client.py         # LightOn client: httpx wrapper, _request, lifecycle, composes verb mixins
  utils.py           # request-body helpers (_compact, _ids) shared by the verbs
  verbs/             # one primary verb per file, each a mixin on LightOn
    _base.py         # _VerbClient: declares _request (LightOn provides the real one)
    ask.py search.py parse.py extract.py
  exceptions.py      # exception tree
  _active_record.py  # _ActiveRecord base: shared list/get/refresh/delete/_bind/_api/_absorb
  workspace.py       # Workspace, active-record, lives at root
  apikey.py          # ApiKey / ApiKeyScope, active-record, lives at root
  tag.py             # Tag, active-record (list/create/delete only; no single GET)
  content_type.py    # ContentType/Facet/Attribute, content-type taxonomy + file facets
  file.py            # File, active-record + wait_all(); upload = ingestion
  batch.py           # ingest_many() batch upload behavior: BatchIngestJob (threads/poll)
  job.py             # ParseJob/ExtractJob, client-bound async handles you poll()
  enums.py           # curated StrEnum vocabularies (FileStatus, Role) shared by resources
  types/             # PURE DATA schemas only (no behavior)
    client/configuration.py   # LightOnConfiguration
    batch.py                  # BatchIngest / BatchProgress / FailedIngest (batch results)
    api/__init__.py           # GENERATED pydantic models (do not hand-edit)
tests/               # pytest (offline, MockTransport)
  e2e/cli.py         # live-API smoke CLI (typer), NOT collected by pytest: make e2e
Makefile             # make test, make e2e, make gen-types
```

Rule: `types/` holds pure pydantic data schemas. Anything with logic/behavior
(like `Workspace`, or `BatchIngestJob` in `batch.py`) goes at the package root, not
under `types/`.

**Data carriers are pydantic `BaseModel`s, never `dataclasses`/`NamedTuple`/`TypedDict`.**
One model system across the SDK (validation, `Field(description=...)`, JSON schema, IDE
hints), and pure-data ones live under `types/`. A schema may reference a behavioral model
as a *field type* (e.g. `types/batch.py` embeds `File`), that's fine; it stays pure data
itself. Construct them with keyword args (pydantic rejects positional). Use
`arbitrary_types_allowed` for opaque non-pydantic fields (e.g. `FailedIngest.error: Exception`).

`enums.py` holds hand-curated controlled vocabularies (`FileStatus`, `Role`) used as
model field types. **`StrEnum`, not `Enum`**, members are strings, so `f.status ==
"embedded"` and set-membership keep working without `.value`, and pydantic
serializes them back to plain strings for request bodies. Values mirror the generated
api enums (`StatusEnum`/`RoleEnum`); if the server vocab changes, `make gen-types`
surfaces it and you update `enums.py` by hand. Only enum a field whose full domain is
known, `workspace_type`/`document_upload_method` stay `str` (plain `str` in the schema
too, no documented value set).

## Client

- **Sync only.** `httpx.Client`. No async client until a real event-loop caller needs one, `_request` is the only logic to mirror.
- **One `_request`** does auth header, error mapping (→ raises), and JSON parse. All calls route through it. A 2xx body that isn't JSON → `MalformedResponseError`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lightonai/lighton-python-sdk](https://github.com/lightonai/lighton-python-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
