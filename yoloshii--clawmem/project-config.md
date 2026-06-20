---
trigger: always_on
description: |
---


# ClawMem Agent Reference

## Architecture

Two tiers: **hooks** handle automatic context flow (surfacing, extraction, compaction survival). **MCP tools** handle explicit recall, write, and lifecycle operations.

---

## Inference Services

Three `llama-server` instances for neural inference. The `bin/clawmem` wrapper defaults to `localhost:8088/8089/8090`.

**Default (QMD native combo, any GPU or in-process):**

| Service | Port | Model | VRAM | Protocol |
|---|---|---|---|---|
| Embedding | 8088 | EmbeddingGemma-300M-Q8_0 | ~400MB | `/v1/embeddings` |
| LLM | 8089 | qmd-query-expansion-1.7B-q4_k_m | ~2.2GB | `/v1/chat/completions` |
| Reranker | 8090 | qwen3-reranker-0.6B-Q8_0 | ~1.3GB | `/v1/rerank` |

All three models auto-download via `node-llama-cpp` if no server is running (Metal on Apple Silicon, Vulkan where available, CPU as last resort). Fast with GPU acceleration (Metal/Vulkan); significantly slower on CPU-only.

**SOTA upgrade (12GB+ GPU):** zembed-1-Q4_K_M (embedding, 2560d, ~4.4GB) + zerank-2-Q4_K_M (reranker, ~3.3GB). Total ~10GB with LLM. Distillation-paired via zELO. `-ub` must match `-b` for both. **CC-BY-NC-4.0** — non-commercial only.

**Remote option:** Set `CLAWMEM_EMBED_URL`, `CLAWMEM_LLM_URL`, `CLAWMEM_RERANK_URL` to remote host. Set `CLAWMEM_NO_LOCAL_MODELS=true` to prevent fallback downloads.

**Cloud embedding:** Set `CLAWMEM_EMBED_API_KEY` + `CLAWMEM_EMBED_URL` + `CLAWMEM_EMBED_MODEL` for cloud providers. Supported: Jina AI (`jina-embeddings-v5-text-small`, 1024d), OpenAI, Voyage, Cohere. Batch embedding, TPM-aware pacing, provider-specific params auto-detected.

### Server Setup

```bash
# === Default (QMD native combo) ===

# Embedding (--embeddings flag required)
llama-server -m embeddinggemma-300M-Q8_0.gguf \
  --embeddings --port 8088 --host 0.0.0.0 -ngl 99 -c 2048 --batch-size 2048

# LLM (auto-downloads via node-llama-cpp if no server)
llama-server -m qmd-query-expansion-1.7B-q4_k_m.gguf \
  --port 8089 --host 0.0.0.0 -ngl 99 -c 4096 --batch-size 512

# Reranker (auto-downloads via node-llama-cpp if no server)
llama-server -m Qwen3-Reranker-0.6B-Q8_0.gguf \
  --reranking --port 8090 --host 0.0.0.0 -ngl 99 -c 2048 --batch-size 512

# === SOTA upgrade (12GB+ GPU) — -ub must match -b ===

# Embedding (zembed-1)
llama-server -m zembed-1-Q4_K_M.gguf \
  --embeddings --port 8088 --host 0.0.0.0 -ngl 99 -c 8192 -b 2048 -ub 2048

# Reranker (zerank-2)
llama-server -m zerank-2-Q4_K_M.gguf \
  --reranking --port 8090 --host 0.0.0.0 -ngl 99 -c 2048 -b 2048 -ub 2048
```

### Verify Endpoints

```bash
curl http://host:8088/v1/embeddings -d '{"input":"test","model":"embedding"}' -H 'Content-Type: application/json'
curl http://host:8089/v1/models
curl http://host:8090/v1/models
```

## Environment Variables

| Variable | Default (via wrapper) | Effect |
|---|---|---|
| `CLAWMEM_EMBED_URL` | `http://localhost:8088` | Embedding server. Falls back to in-process `node-llama-cpp` if unset. |
| `CLAWMEM_EMBED_API_KEY` | (none) | API key for cloud embedding. Enables cloud mode: batch embedding, provider-specific params, TPM-aware pacing. |
| `CLAWMEM_EMBED_MODEL` | `embedding` | Model name for embedding requests. Override for cloud providers (e.g. `jina-embeddings-v5-text-small`). |
| `CLAWMEM_EMBED_TPM_LIMIT` | `100000` | Tokens-per-minute limit for cloud embedding pacing. Match to your provider tier. |
| `CLAWMEM_EMBED_DIMENSIONS` | (none) | Output dimensions for OpenAI `text-embedding-3-*` Matryoshka models. |
| `CLAWMEM_LLM_URL` | `http://localhost:8089` | LLM server. Falls to `node-llama-cpp` if unset + `NO_LOCAL_MODELS=false`. |
| `CLAWMEM_RERANK_URL` | `http://localhost:8090` | Reranker server. Falls to `node-llama-cpp` if unset + `NO_LOCAL_MODELS=false`. |
| `CLAWMEM_NO_LOCAL_MODELS` | `false` | Blocks `node-llama-cpp` auto-downloads. Set `true` for remote-only. |
| `CLAWMEM_ENABLE_AMEM` | enabled | A-MEM note construction + link generation during indexing. |
| `CLAWMEM_ENABLE_CONSOLIDATION` | disabled | Light-lane consolidation worker (Phase 1 backfill + Phase 2 merge + Phase 3 deductive synthesis + Phase 4 recall stats). **v0.8.2:** every tick wraps in a `worker_leases` row (`light-consolidation` key) so multiple host processes against the same vault cannot race on Phase 2 merges. Hosted by `clawmem watch` (canonical) or `clawmem mcp` (per-session fallback). |
| `CLAWMEM_CONSOLIDATION_INTERVAL` | 300000 | Worker interval in ms (min 15000). |
| `CLAWMEM_MERGE_SCORE_NORMAL` | `0.93` | **v0.7.1.** Phase 2 merge-safety score threshold when candidate and existing anchors align. |
| `CLAWMEM_MERGE_SCORE_STRICT` | `0.98` | **v0.7.1.** Strictest merge-safety threshold (fallback when anchors are ambiguous). |
| `CLAWMEM_MERGE_GUARD_DRY_RUN` | `false` | **v0.7.1.** When `true`, Phase 2 merge-safety rejections are logged but not enforced — use for calibration. |
| `CLAWMEM_CONTRADICTION_POLICY` | `link` | **v0.7.1.** How the merge-time contradiction gate handles a blocked merge. `link` (default) keeps both rows + inserts `contradicts` edge. `supersede` marks the old row `status='inactive'`. |
| `CLAWMEM_CONTRADICTION_MIN_CONFIDENCE` | `0.5` | **v0.7.1.** Minimum combined heuristic+LLM confidence required before the contradiction gate blocks a merge. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yoloshii/ClawMem](https://github.com/yoloshii/ClawMem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
