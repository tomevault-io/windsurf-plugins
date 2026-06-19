---
trigger: always_on
description: Use RoutIR — call a running endpoint with the client, stand up a local server that mixes locally-hosted services with services proxied from a remote master server, and extend RoutIR with new bi-encoders, rerankers, and document collections by writing a small config (and optionally one Python file referenced via `file_imports`). No checkout of the routir source is required.
---


# RoutIR skill

RoutIR is an HTTP/gRPC service that hosts retrieval engines (dense, sparse,
rerankers, fusion, query expanders) behind a uniform API and composes them
into multi-stage pipelines with a small DSL. This skill covers the five
things you actually do with it:

1. Call a running RoutIR endpoint with the client.
2. Stand up your own RoutIR server, optionally importing services from a
   master server so users can pipeline local + remote engines together.
3. Wrap a bi-encoder retrieval index.
4. Wrap a cross-encoder reranker.
5. Serve document collections (single- or multi-view).

**Run everything through `uvx`.** This project forbids bare `pip`,
`python`, `pytest`, `ruff`. Every command below uses `uvx` so the deps land
in a throwaway venv instead of your conda environment.

The full reference for extending RoutIR with Python lives in
[`examples/CLAUDE.md`](examples/CLAUDE.md); pipeline / DSL / config reference
in the project [`CLAUDE.md`](CLAUDE.md). This skill is the operational
overview — read those when you need depth.

---

## 1. Use the client (and the pipeline DSL)

`routir.client.Client` is the sync facade (auto-uses gRPC when the server
advertises it, falls back to REST). `AsyncClient` is the async version with
the same surface.

```python
from routir.client import Client

with Client(endpoint="http://compute01:5000", api_key="…optional…") as c:
    print(c.avail())                          # what's available
    print(c.transport)                        # "grpc" or "rest"

    # search a single hosted index
    r = c.search(service="qwen3-neuclir", query="…", limit=20)
    # r["scores"] -> {doc_id: float}

    # score a list of passages against a query
    r = c.score(service="my-reranker", query="…", passages=["p1", "p2", …])
    # r["scores"] -> [float, float, …]   (one per passage)

    # fetch document content
    r = c.content(collection="neuclir", id="doc-id-123", view="asr")
    # r["text"] or r["bytes"]

    # run a composed pipeline (the main entry point)
    r = c.pipeline(
        pipeline="{dense%1000, bm25%1000}RRF%100 >> rerank@asr%20",
        query="…",
        collection="neuclir",                 # needed when a stage reranks
        runtime_kwargs={"rerank": {"some_engine_kwarg": 0.5}},  # optional
    )
```

**Endpoint scheme rules:** `http(s)://…` → REST; `grpc(s)://…` → gRPC;
bare `host:port` is treated as `http://` and may auto-upgrade to gRPC if
the server advertises `grpc_port` via `/avail`. Pass
`transport="rest"` to force REST.

### Pipeline DSL — quick reference

| DSL form                                | Meaning                                                                       |
| --------------------------------------- | ----------------------------------------------------------------------------- |
| `svc%N`                                 | Call `svc`, keep top *N*.                                                     |
| `A%1000 >> B%20`                        | Sequential: A retrieves 1000, B reranks down to 20. B gets role `rerank`.     |
| `{A%K, B%K}Merger%N`                    | Parallel: A and B run concurrently; `Merger.fuse_batch` fuses to top *N*.     |
| `Expander{A%K, B%K}Merger%N`            | Expander makes sub-queries; each runs A and B in parallel; all fused.         |
| `svc[alias]%N`                          | Name a stage so `runtime_kwargs` can target it by alias.                      |
| `svc@view%N`                            | Pick a named view of the collection at rerank time (multi-modal).             |

**Built-in mergers:** `RRF` (reciprocal rank fusion) and `ScoreFusion` are
always available. You only need to write your own fusion engine if you need
something neither covers.

**Aliases:** `pipeline_aliases` in the server config let you give a long
pipeline a short name (`"ragtime2": "{zho%100, rus%100, …}ScoreFusion"`);
the alias is then usable everywhere a service name is. The call-site `%N`
re-caps the alias's outer-most stage.

**Bound the result set.** `/pipeline` ignores a top-level `limit`; the
result-set size is whatever the final `%N` produced. If your pipeline has
no `%N`, you'll get the full inner result.

`scripts/query.py` is the canonical batch-query script that writes the
[JSONL run format documented in `CLAUDE.md`](CLAUDE.md#search-results-output-format).
Copy it when scripting evaluations.

---

## 2. Stand up a local server (and import a master server)

Minimal `my-config.json`:

```json
{
    "services": [
        { "name": "my-bm25", "engine": "PyseriniBM25",
          "config": {"index_path": "/path/to/bm25-index"},
          "cache": 1024, "cache_ttl": 600,
          "batch_size": 16, "max_wait_time": 0.05 }
    ],
    "collections": [
        { "name": "my-corpus", "doc_path": "/data/corpus.jsonl" }
    ],
    "server_imports": [
        "http://master-host:5000"
    ],
    "file_imports": [
        "./my_extension.py"
    ]
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hltcoe/routir](https://github.com/hltcoe/routir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
