---
trigger: always_on
description: A native Zig inference engine for Apple Silicon serving exactly two models: Qwen3.8-Flash-Next (`qwen4_exp`, EXL3
---

# Sushi — project context for AI

A native Zig inference engine for Apple Silicon serving exactly two models: Qwen3.8-Flash-Next (`qwen4_exp`, EXL3
routed experts resident; the bf16 checkpoint can SSD-stream) and MiMo-V2.6-Flash (`mimo_v2`, experimental, text-only,
MCG EXL3 or streamed MXFP4; public from v1.1). OpenAI/Anthropic-compatible HTTP, no Python at serve time. Fork of ddalcu's mlx-serve.

- **sushi** is this engine's new name (runtime, to be opened to the public): a scripted rename changes the binary
  name, the env prefix and the home dir everywhere; until it lands, write today's names. Treat every change as future
  public code (license, NOTICE, docs); sushi is not scoped to EXL3 or to these two models forever.
- **sashimi** is the PRIVATE quant-creation stack (its own private repo): every converter, allocator, imatrix driver
  and repacker. This repo keeps only the CONSUMER contract — `docs/pack-format.md`, the Zig shard-stamp check, the
  committed `src/fixtures/`, and `sushi kld`. The oracle fixture dumpers (`tests/dump_*_fixtures.py`) stay: they
  verify the engine, they do not make packs. Converter knowledge (recipes, calibration data, research notes) never
  goes into a committed file: it goes to `docs/private/`, and committed files say only that it lives in the private
  repo.
- Everything else in `src/` (other architectures' forwards and loaders in `transformer.zig`/`model.zig`, the dormant
  ANE driver) is INHERITED upstream code: it builds, it is unreachable, and no doc covers it. The loader refuses any
  other `model_type` by name (`model.served_model_types`, `ArchitectureUnsupported` → 503); an unsupported file
  format is refused by name (`ModelFormatUnsupported` → 503; `--model` exits).

<a id="docs-index"></a>
## Docs index

This file holds rules and the map. Knowledge, measurements and lessons live in `docs/<category>-<topic>.md`; read the
doc for the area before changing it, and update it in the same landing.

| doc | what it holds |
|---|---|
| [docs/arch-qwen4exp.md](docs/arch-qwen4exp.md) | Flash-Next trunk, hyper-connections, n-gram PLE table, oracle and ties, packs on this box |
| [docs/arch-mimo-v2.md](docs/arch-mimo-v2.md) | MiMo checkpoint, FP8 trunk, rank-local QKV, routing/sinks, sliding ring, bills, product policy |
| [docs/engine-exl3-experts.md](docs/engine-exl3-experts.md) | EXL3 rate/codebook/window, prefill GEMM, decode chain, f32 SwiGLU, parity bars |
| [docs/engine-expert-streaming.md](docs/engine-expert-streaming.md) | SSD budget ledger, per-layer LRU, slab I/O, imatrix capture, discovery |
| [docs/engine-mtp.md](docs/engine-mtp.md) | native MTP head, verify invariant, draft re-scoring, round-cost table, head KV/norms |
| [docs/engine-qsa-long-context.md](docs/engine-qsa-long-context.md) | QSA arms per query width, indexer and its history, long-context admission and bills |
| [docs/engine-kv-cache.md](docs/engine-kv-cache.md) | kv8 default, kv-quant contract, growth, GDN step, byte-stability settings |
| [docs/engine-prefix-cache.md](docs/engine-prefix-cache.md) | hot cache, hybrid restore, trimming, SSD tier, SSD-first, checkouts, spec state |
| [docs/engine-kernels.md](docs/engine-kernels.md) | decode/MoE/prefill/verify kernels, NAX/MPP pitfalls, how to prove and time a kernel |
| [docs/engine-mlx-gotchas.md](docs/engine-mlx-gotchas.md) | MLX errors, dtype promotion, barriers, views vs copies, allocator pool, Zig and tokenizer traps |
| [docs/engine-memory-admission.md](docs/engine-memory-admission.md) | Metal OOM, preflight, auto-context, prefill chunk, admission |
| [docs/server-http-apis.md](docs/server-http-apis.md) | API contracts, streaming, logprobs/seeds/sampling, reasoning budget, constrained JSON, launcher |
| [docs/server-tool-calling.md](docs/server-tool-calling.md) | templates, tool-call parse chain and invariants, think tags, loop stops |
| [docs/server-lifecycle.md](docs/server-lifecycle.md) | arch gate, weight loader, settings precedence, scheduler/batching, threads, ownership, media |
| [docs/pack-format.md](docs/pack-format.md) | what a pack owes the engine: tensors, `expert_quant`, `__metadata__` stamp, window, g-scale in `suh`, loader rules |
| [docs/perf-baselines.md](docs/perf-baselines.md) | roofline, recorded tok/s tables with binaries and settings, ruled-out levers |
| [docs/quality-kld.md](docs/quality-kld.md) | `kld` tool, teacher fixtures, the 16x512 reading, lossless teacher rule, KLD of every served pack |
| [docs/process-measurement.md](docs/process-measurement.md) | GPU lock, binary stamp, QoS, waiting, baseline lookup, recording a number |
| [tests/CLAUDE.md](tests/CLAUDE.md) | the integration-test matrix (auto-loads in `tests/`) |

**Private, local-only** (`docs/private/`, gitignored): they exist only in the main checkout, so a git worktree does
not contain them; a worker in a worktree reads them from the main checkout's `docs/private/`. Never link to or
quote them from a committed file.

| doc | what it holds |
|---|---|
| `docs/private/sashimi-workflow.md` | how to convert with sashimi: venv, subcommands, served-pack recipes, imatrix files and hashes, stamps, window speeds, wall times, speed work, lessons |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beamivalice/sushi](https://github.com/beamivalice/sushi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
