---
trigger: always_on
description: Read this before changing code. It covers what a contributor doc can't: the
---

# AGENTS.md — orientation for AI agents working on omind

Read this before changing code. It covers what a contributor doc can't: the
invariants that break *silently*, the current state of in-flight work, and the
dead ends someone already walked down.

For dev setup, the four quality gates, the module map, SemVer rules, and PR
conventions, see **[CONTRIBUTING.md](CONTRIBUTING.md)** — this file does not
repeat them.

---

## The one-paragraph version

omind gives AI agents durable memory as **plain Markdown notes** in an Obsidian
vault, replicated peer-to-peer over git, exposed to agents through a local MCP
server (`omind node`) and enforced by a hook-based guard. Roughly a third of the
repo is memory; the rest is agent enforcement (`guard.py`, `policy.py`,
`verify.py`), fleet provisioning (`provision.py`, `agents.py`), and mesh
replication (`mesh.py`, `merge.py`). Know which subsystem you are in.

## Invariants — violate these and something breaks quietly

1. **The Markdown files are the source of truth.** Indexes, caches, and vectors
   are derived and disposable. Anything derived lives in the state dir
   (`paths.state_dir()`), never in the vault: vault contents are committed and
   mesh-synced, so a derived file there would replicate machine-specific junk
   fleet-wide.
2. **Retrieval fails open.** Every layer of search returns `None` on any problem
   and the caller falls back to the older, dumber path. A missing model, a
   corrupt index, a locked database, a missing FTS5 build — all degrade search;
   none may break it. Test the failure branch, not just the happy one.
3. **Multi-note writes go through `omind.txn`.** One note at a time is already
   atomic (same-dir temp + `os.replace`). Two or more is not, and a crash
   between them used to leave partial state with no way back. Any new operation
   that writes several notes must journal its pre-images through
   `txn.Transaction` under `store.write_lock()`, or it reintroduces the gap
   `omind recover` exists to close. Recovery refuses to overwrite a note edited
   after the crash — that edit is newer than the pre-image.
4. **All note writes go through `OmiStore`** (`notes.upsert_note` for external
   writers) — flock, atomic rename, Lamport `Rev:` stamping, soft delete.
   Deletes archive (`Disabled: true`); only `omind mesh purge` truly removes.
5. **`OmiStore.safe_name` guards every read and write.** Path traversal must stay
   impossible; there are tests enforcing this. Don't route around them.
6. **`store.py` stays framework-free.** No FastAPI, no MCP. The CLI and the web
   app both build on it.
7. **Credential notes are de-prioritised** in search and in the gate's
   suggestions unless the query is itself about credentials
   (`retrieve._CREDENTIAL_PENALTY`). The gate must never steer an agent into the
   secrets notes. This is load-bearing, not decoration.
8. **No MCP tool may return unbounded output.** Every list-shaped tool pages
   (`limit`, `offset`, `total`, `has_more`) via `server._page`. `list-notes` once
   returned ~90,800 tokens in a single result; `tests/test_server.py::
   test_every_list_tool_is_bounded` exists so a new tool cannot regress that.
9. **Reserved files are not memories.** `index.md` and `Memory Template.md` are
   scaffolding; reading one does *not* clear the consult gate
   (`paths.NON_CONSULT_FILENAMES` — an anti-dodge measure, issue #109).

---

## Handoff: the 2026-08-27 multi-agent review (shipped)

The 2026-07-24 retrieval handoff below is history: `feat/hybrid-retrieval-token-budgets`
landed long ago, and #167–#178 all shipped (see BACKLOG.md). Current state at this
writing: **v8.8.0 on `main`**, 1,000+ tests + ruff + mypy strict green. Trust
**BACKLOG.md + CHANGELOG.md** for what's open — not older handoffs.

### What changed and why

A nine-slice multi-agent review (report:
[docs/reviews/2026-08-27-multi-agent-review.md](docs/reviews/2026-08-27-multi-agent-review.md))
found 35 issues; all are fixed in the working tree. The classes worth remembering:

- **Locking is total now, and the pattern is uniform.** Any read-modify-write of a
  small state file (gate sentinels, re-close/off-topic counters, `loop_guard.json`,
  `node.json`, `backup.json`, harness settings.json) goes through
  `filelock.exclusive()` on a SIBLING `.lock` path — never on the data file itself,
  because the data file is replaced atomically and a flock on a replaced inode
  protects nothing. Apply this to any new state file.
- **Optimistic concurrency is the coordination story.** `upsert_note` captures the
  version BEFORE reading existing fields; checkpoint/okf/consolidate pin versions;
  `edit-note` without a token answers `concurrency: "unverified"` instead of
  silently last-write-wins.
- **Deletion yields to newer edits.** Purge tombstones carry the purge-time Rev;
  `_apply_tombstones` keeps and reports a note whose live Rev dominates it.
- **Merge LWW losses are visible in the vault** (`merge-lww` tag), not just daemon
  stderr. If you touch `merge.py`, keep scalar losses tagged.
- **The index stores the encoder IDENTITY** (`embed.model_identity`: name +
  embedding-space digest), not the model name — same name/different snapshot used

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CryptoJones/omind](https://github.com/CryptoJones/omind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
