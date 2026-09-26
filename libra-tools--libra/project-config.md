---
trigger: always_on
description: `libra agent` is an intentionally different external-agent capture extension,
---

# Agent Command Development

`libra agent` is an intentionally different external-agent capture extension,
not a Git-compatible command.

OpenCode content capture on **macOS** is assembled through seatbelt
(`sandbox-exec`) via `SandboxManager::transform`; the seatbelt backend carries
a deprecation (**弃用**) risk if Apple removes `sandbox-exec`. See
[`../tracing/agent.md`](../tracing/agent.md) §5 for the read-isolation
asymmetry and fail-closed metadata-only degrade when the backend is missing.

The shared `run_bounded_exporter` Unix `pre_exec` sets both soft and hard
`RLIMIT_CORE` to zero next to the existing per-platform `RLIMIT_FSIZE`.
Failure to set either limit fails spawn with context. Core limits propagate
to exporter descendants without changing the parent process; SIGXFSZ
disposition, byte caps, deadlines and sandbox controls are unchanged.
This also suppresses cores from unexpected exporter crashes. Piped core
handlers decide whether to honor the limit. systemd-coredump v259 honors
it when `core_pattern` passes the limit through `%c`; brief signal metadata
can remain. The regression
`opencode_export_core_limits_are_zero_in_child_and_descendants` exercises
the real runner and confirms child/descendant limits and unchanged parent
limits. It does not establish the cause of any historical linker SIGKILL.

The active development contract, backlog, and compatibility guardrails live in
[`../tracing/agent.md`](../tracing/agent.md). Keep this file as the command
development index entry so `docs/development/commands/README.md` can list every
public CLI command without duplicating the Agent planning document.

## Deferred / Non-goal parity

The following external-agent parity surfaces are decided **non-goals** for the
current wave. Each is recorded — with its handling and restart condition — in
the 「还未实现的功能」 table of [`../tracing/agent.md`](../tracing/agent.md)
(the canonical Agent contract); they are surfaced to users in
[`docs/commands/agent.md`](../../commands/agent.md) and in the `agent` row of
[`COMPATIBILITY.md`](../../../COMPATIBILITY.md):

1. **`agent add`/`remove` `--local-dev` / `--force`** — unpublished; canonical
   `status` / `enable` / `disable` (+ `add` / `remove` aliases) only. If
   implemented, each must hang on both the canonical verb and its alias.
2. **Provider-specific transcript compaction/reassemble trait** — deferred parity
   on top of the landed manifest-relative chunking (no provider-specific
   compactor yet).
3. **Optional capability traits** (`ProtectedFilesProvider`, `TranscriptCompactor`,
   `HookResponseWriter`, `RestoredSessionPathResolver`, …) beyond the landed
   `DeclaredAgentCaps` set — no public behavior yet.
4. **External-RPC method family beyond the v2 `info`/capability gate** —
   undeclared capabilities stay fail-closed.
5. **Non-first-batch supported roster** — `gemini` / `cursor` / `copilot` /
   `factory-ai` stay `supported=false` (unsupported, not hook-installable, not
   launchable) and are omitted from `agent list` entirely; the first batch is
   `claude-code` / `codex` / `opencode`. The omission is pinned by
   `tests/command/agent_roster_test.rs::agent_roster_surface`; the unsupported
   registry classification stays pinned by
   `tests/compat/agent_capability_matrix_pin.rs`.

## Historical import contract

The active DR-05/M4 contract is implemented by `libra agent import` and is
specified canonically in [`../tracing/agent.md`](../tracing/agent.md): explicit
consent before content access/export, provider-root descriptor authorization,
typed redaction before persistence, current-repository ownership, coverage
claim + import identity fencing, and local erase tombstones. The default
`agent list --json` remains schema v1; callers opt into the method matrix with
`--schema-version 2`. Batch limits charge bytes actually read from the held
source even when a candidate later fails validation, and the absolute deadline
begins before discovery, bounds reservation/object/CAS work, and releases every
owned uncommitted import lease on expiry. Transaction commit awaits are not
cancelled: the deadline is checked immediately before commit and the resulting
success is authoritative even if observation finishes after the deadline. A
failed abandonment is chained into the surfaced error with a doctor-repair hint.
Repository ownership is the canonical shared Libra
storage identity, so sibling linked worktrees are accepted while cross-repo
sources remain rejected. Import attempt markers are created in the reservation
transaction; live, export, and subagent writers use the same fail-closed
pre-object registration. The effective per-source read cap is
`min(agent.max_transcript_read_bytes, 16 MiB)`; explicit larger settings emit
the actual effective value. Discovery traversal/open and held-descriptor file
reads run in private, kill-on-timeout helper processes wrapped by the command's
absolute deadline. Provider roots are
opened component-by-component; Claude sources and each nested Codex date
directory are opened relative to pinned no-follow descriptors before consent. Each new OID is added
as a durable provisional preclaim before its loose-object write, but becomes
deletion-eligible only after this writer wins publication and records it in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libra-tools/libra](https://github.com/libra-tools/libra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
