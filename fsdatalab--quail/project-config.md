---
trigger: always_on
description: Write like two engineers talking at a whiteboard.
---

# Writing

Write like two engineers talking at a whiteboard.

- Lead with the answer, then the reasoning. Short sentences, one idea
  each.
- Use bullet points for anything longer than a few sentences.
- Use everyday words. Define an unavoidable technical term once, in
  one sentence, then use it the same way every time.
- No analogies, metaphors, or dramatic shorthand. Say "the run is on
  Modal", not "the flight is in the air"; say "saved", "finished",
  "running", not "banked", "landed", "armed", "healthy".
- Give an example with "For example," and all of its context: "For
  example, imagine a table where each row is an agent trace…".
- Give numbers, and say what each is compared with: "52.3 s, compared
  with 47 s for the unavoidable work alone".
- Do not start a sentence with a number.
- When something failed or is uncertain, say so, and say what would
  settle it.
- Call the KV cache "KV".
- Say "run" or "configuration" for the runs of an experiment, never
  "arm".

## Code

- Comments state constraints the code cannot show, nothing else.
- Docstrings follow Google style: a one-line summary, a blank line,
  then optional Args, Returns, and Raises. Say what the code does, not
  why it was written or what it replaced.
- Use one name for each concept across the planner, the executor, the
  docs, and the metrics.

# Checks

CI runs these on every pull request. Run them before pushing:

    uv run ruff check quail tests experiments tools
    uv run python tools/check_long_strings.py
    uv run vulture
    uv run pytest -q

- Ruff enforces line length 88, import order, naming, and Google style
  docstrings (`[tool.ruff]` in `pyproject.toml`).
- `tools/check_long_strings.py` flags string literals over 200
  characters. Long content is allowed when named as content (a name
  containing PROMPT, TEMPLATE, SQL, QUERY, HTML, or TEXT), kept in a
  prompts module or folder, or plainly HTML or SQL. Docstrings are
  exempt. Shorten long messages and log lines.

# Names

- The project is Quail (QUery-Aware Inference Layer); the package is
  `quail`.
- quail-b, the benchmark, lives in
  https://github.com/fsdatalab/quail-bench and is installed here as
  `quail_b`, pinned to a commit in `pyproject.toml`. `quail/bench/` is
  Quail's runner for it. Change queries or labels there first, then
  move the pin here.
- The execution mechanisms are "pipelining", "token-based admission",
  "KV rewind", and "prefix sharing". "Chain mode" is the code name for
  KV rewind; prefer "KV rewind" in prose.
- The attention paths are "unified" and "tree".
- Compare against "stock vLLM", and say which submission strategy it
  used: operator-at-a-time or pipelining.
- Every `Session` takes an `EngineConfig` that names `model` and
  `device`; neither has a default. `gpus` defaults to 1 and `backend`
  to `"quail"`.

# Scope

- Filter and join queries on Qwen3 4B fp8, Qwen3 32B fp8, or
  DiffusionGemma 26B-A4B fp8, on one H100 per model copy. No
  tensor-parallel weight sharding.
- `AI.CLASSIFY`, `AI.EXTRACT`, and `AI.MAP` are on the roadmap.
  Open-ended generation, speculation, and forking are not supported.

# What Quail builds on and learns from

Quail is built from these projects. Read their code and docs before
reinventing a piece, and hold Quail's code and docs to their
standard.

Built on:

- **vLLM**: model definitions, weight loading, and kernels; also the
  main baseline.
- **FlashAttention**: the attention kernels, over paged KV.
- **DeepGEMM**: the FP8 matrix multiplies.
- **Triton**: Quail's fused kernels (RMSNorm plus quantize, QK-norm
  plus RoPE, SiLU plus quantize, the attention merge).
- **Apache Arrow and Acero**: tables, streams, and the relational
  operators in query plans.
- **Substrait**: the plan format quail-b queries are written in.
- **sqlglot**: SQL parsing.
- **Hugging Face** (`transformers`, `huggingface_hub`, `datasets`):
  tokenizers, checkpoints, and benchmark data.
- **Modal**: every GPU run and the result volumes.
- **SGLang**: a second baseline.

Learned from. Treat these as whole projects to study, not single
features: their APIs, their code layout, their tests, their docs, and
how they release and explain changes.

- **Apache Spark**: the model for Quail as a whole.
  - Catalyst, for logical and physical plans rewritten by named rules
    (Quail's planner rules).
  - The DataFrame and SQL APIs, for Quail's builder and `sql()`.
  - Spark Connect, for a thin client talking to a remote server
    (Quail Server and the remote `Session`).
  - `EXPLAIN` and the Spark UI, for showing a plan and what it cost
    (`explain()`, `explain(analyze=True)`).
- **Apache DataFusion**: an Arrow-native query engine built to be
  extended, with clear extension points (Quail's extension registry,
  custom operators, and rules) and docs for each.
- **DuckDB**: an engine that is easy to embed and whose docs answer a
  question in one short page with a runnable example.
- **Apache Arrow**: columnar data and Arrow Flight for moving results
  between processes.
- **vLLM**: the serving engine Quail runs beside and compares
  against; its paper and docs explain one mechanism per section, such
  as paged KV and prefix caching, with a small example.
- **Modal**: docs that start with what a feature does, then a small

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fsdatalab/quail](https://github.com/fsdatalab/quail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
