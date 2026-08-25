---
trigger: always_on
description: provides `CUA_LITE_ENV_SERVER_URL` plus `CUA_LITE_ENV_SERVER_TOKEN`. Subagents
---

# AGENTS.md

This is the repo-local operating guide for AI coding agents. Keep this file as
the short execution contract: it is self-contained, and must not depend on
scratch planning documents, which come and go.

## Non-Negotiables

- Read the surrounding code before editing. Reuse the owner module and the local
  file-family style instead of inventing a parallel path.
- Do not `git add` or commit unless the user explicitly asks.
- Do not modify `slime/` unless the user explicitly asks.
- Do not pull top-level `README.md` or `docs/**` into readability/cleanup slices.
  Touch them only when the user directly scopes those files.
- Do not modify `.venv/`, `__pycache__/`, `.pytest_cache/`, or other generated
  caches.
- Do not use any memory system. Do not read from or write to a memory directory.
- Never stop, kill, exec into, or otherwise interfere with a Docker container or
  server you did not start. Clean up only resources owned by your session/token
  scope.
- Markdown links to repo files should use repo-root absolute paths, such as
  `/pyproject.toml`, so they work from any GitHub page.

## Engineering North Star

Use these code-shape rules before choosing a local fix. They are meant to keep a
small contract cleanup from growing into new aliases, hidden behavior, or a
general loop that knows too much about one downstream case.

### One Owner, One Vocabulary

Each durable fact should have one owning module and one representation. If two
shapes express the same fact, fix the producer or shared contract instead of
adding another branch.

Do not create distributed aliases for one concept. This includes near-synonym
names, helpers that only rename an operation, batch-tool / GUI-batch aliases for
action-batch, and native-wrapper vocabulary outside provider/model-family
projections.

Do not keep old spellings alive through import re-exports, wrapper functions,
test-only aliases, or facade shims unless the old path is documented public API;
otherwise update callers directly.

When naming a concept, choose the layer first and keep that spelling everywhere:
canonical Lite contracts, provider projections, env/server transport, and
debug/log artifacts are different layers. Do not let one layer's vocabulary leak
into another layer unless the code is explicitly projecting between them.

### No Implicit Backward Compatibility

Do not preserve backward compatibility for internal contracts, old helper names,
pre-refactor tests, or unpublished data shapes. Once a contract is chosen, make
the repo use that contract directly. Update callers, fixtures, and tests instead
of adding compatibility branches, fallback readers, or defensive "old shape"
handling.

Legacy repair is allowed only in the explicit migration/preproc owner for
already-published external data, and it must end before normal runtime,
training, rollout, stage, or export readers. Do not add compatibility shims to
general loops, adapters, or core facades unless the old path is documented public
API and the user explicitly scopes support for it. Treat unrequested backward
compatibility as over-defense: it keeps invalid states representable and spreads
the old contract into new code.

### Boundary Ownership

Keep general layers general. Shared runtime, core, and adapter code must not
carry downstream-specific escape hatches for one model family, env, dataset, or
rollout quirk. Put specific behavior at the owning boundary, then call that
owner from the general loop.

General loops should be downstream-agnostic. They may call a narrow owner hook
only when current production callers prove the hook owns a real shared boundary
decision; otherwise keep simple local logic direct. Loops should not contain
concrete model-family names, env IDs, dataset names, provider incident
workarounds, or special cases for one rollout source. Future or near-term users
are not evidence.

### Proportional Design

Simple changes should stay direct. A small mechanical migration should read like
a small mechanical migration: direct field access, direct builder calls, and
local control flow are often clearer than extra helper layers.

Before adding a helper, answer all three questions:

1. Does it own a real policy, boundary transformation, or repeated nontrivial
   logic?
2. Is it used by more than one real downstream owner, not just one call site or
   one test?
3. Is its name more revealing than the code it replaces?

Add a helper only when all three answers are yes. If any answer is no, keep the
logic local. Do not add helpers whose only effect is to rename, unpack/repack,
forward one call, or hide a one-line selection such as `indices[-1:]`, a direct
constructor call, or `call["function"]` access.

Large pure-addition diffs are suspicious for migrations and cleanup. When the
task is mostly `flat -> nested`, readability, or owner cleanup, expect many
edits to be direct replacements, deletions, or small local restructures. New
frameworks, compatibility branches, and broad helper surfaces need explicit
justification.

Net simplification is evidence that the owner and contract likely became
clearer, not a score to optimize. Do not delete structure, names, tests, or
comments just to reduce diff size if they still carry real policy, boundary

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cua-lite/cua-lite](https://github.com/cua-lite/cua-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
