---
trigger: always_on
description: Kaibo is a stdio MCP server that provides an assistant agent **for other agents**.
---

# AGENTS.md — kaibo

Kaibo is a stdio MCP server that provides an assistant agent **for other agents**.
It augments a calling agent (Codex, Claude, Gemini, local agents, etc.) with a team of
models, lending one kind of help — *consultation*: grounded, cited, read-only answers
about a codebase. The team
*perceives* what fuses into its reasoning (image input today — `view_image` and image
attachments on the model-driven tools; more modalities as the models gain them), and —
when a cast carries an `image` slot — *produces*: the `generate` tool writes artifacts
into kaibo's own media CAS (content-addressed, provenance-recorded, retrieved by the
operator via `kaibo://cas/<digest>`), **never into the project**. Recording and emitting
are always a specific mediated tool over kaibo's own store, not a general write path;
see the read-only invariant.

## For agents working here

- Keep status explicit. If you change direction, say what changed and why. If a tool or
  sandbox blocks work, report the exact blocker and the safe next option.
- Prefer a worktree for PR work. Use `~/src/wt/<repo>-<topic>` for local worktrees.
- Keep `signoff.md` as local, ignored session memory when useful; melt durable parts into
  checked-in docs before they go stale.
- Use a little 日本語（にほんご）when it fits; add ふりがな for kanji.

**Consultation: one primitive, three tools.** The primitive is `run_phase`
(`consult.rs`): a model + preamble + an *injected toolset*, run as a bounded tool
loop. Each consultation tool is that loop wearing different clothes:

- **`consult`** — the synthesis agent with `{run_kaish, explore′}` + optional caller
  `context`: it reads precise spans directly and delegates broad sweeps to a cheap
  explorer sub-agent, then answers. No rigid explorer→synth hand-off; the model
  chooses. Supplied context is trusted starting evidence — it investigates for
  *more*, not to re-verify. The `explore′` sweep (`report_preamble`) lives *inside*
  this loop now; it is not a standalone tool.
- **`oneshot`** — the synthesis agent with **no tools**: the caller owns the context, so
  it's one upstream request, prompt in / answer out, no codebase access. The thin
  counterpart to `consult` — and the door to a model outside the caller's family.
- **`run_kaish`** — drive the read-only kaish shell directly, no model in the loop.

Both model-driven tools name their cast + answering model(s) in a provenance footer
(`with_provenance` in `server.rs`), so a cross-model study sees which model answered.

A `--no-<tool>` capability flag per tool gates the surface (all on by default; `consult` also
gates `consult_submit`, `batch` gates `batch_submit`, `generate` additionally needs the
media CAS on, and the `job_*` verbs follow their live producers — a deferred `generate`
mints a `job-N` like the other producers). A tool also needs a cast that can **staff**
it, or its route is dropped;
a server left with nothing advertised is refused at startup, by either road. See
`CAST_ENUM_RULES` and `live_tools` in `server.rs`, and "Tool gating" in `docs/config.md`.
Multi-provider over `rig-core`: a **`ProviderKind`** is the wire protocol (keyed
Anthropic / DeepSeek / Gemini / OpenRouter, plus **`openai`** for any OpenAI-compatible
endpoint). A **`[backends.<name>]`**
(`config.rs`) is a *named connection* of a kind with its own base URL and key source —
so two `openai` backends (hosted GPT and a local Gemma/llama.cpp server, say) can be
live at once. A **`[casts.<name>]`** is a model team mapping each reasoning role
(`explorer` / `synth`, with a `vision` pin where a slot reads images) to a
`"backend/model-id"`, freely cross-backend, so one cast can pair a cheap local explorer
with a hosted synth; a call picks its team with the `cast` arg. Backends and casts come from a built-in registry merged under an XDG
`config.toml`, `KAIBO_*` env, then CLI flags (precedence: per-call > CLI > env > file >
built-in); a missing config file is a non-error. See `docs/config.md`, and
`docs/casts.md` for the backends/casts design rationale. kaibo never modifies the
project and cannot run external commands.

## Invariants — do not weaken without a failing-first test

- **Read-only is the product.** Enforced in `src/sandbox.rs` by four *structural*
  levers — there is no hardcoded denylist: (0) a minimal feature surface (only the
  `localfs` axis; `subprocess`/`git`/`host`/`os-integration` are OFF, so
  `exec`/`spawn`/`kill`/`git`/`ps` are never compiled in), (1) a read-only mount
  (every write/delete/`mkdir`/`touch`/`dd of=` is refused at the VFS layer), (2)
  `MemoryFs` at `/` (paths outside the project land in ephemeral scratch, never on
  disk), and (3) external commands disabled. The `Blocked` wrapper survives only for
  the config-driven `[sandbox].disable_builtins`, which can make the box *stricter* —
  see the module doc-comment. Any change here keeps `tests/sandbox.rs` green and adds
  a test that can fail. **The shell writes nothing and kaibo never touches the project**
  (no kaish write path; the four levers unconditional). Read-only is scoped to what the
  *model can steer* — kaish's VFS never sees kaibo's own state. kaibo keeps handler-side
  state (sessions + batch handles, the latter recovered on demand via `job_list`) only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobert/kaibo](https://github.com/tobert/kaibo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
