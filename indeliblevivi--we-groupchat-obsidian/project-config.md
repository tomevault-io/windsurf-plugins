---
trigger: always_on
description: - `app.py` is the macOS menu-bar and py2app application entrypoint. Its
---

# Repository agent contract

## Source ownership

- `app.py` is the macOS menu-bar and py2app application entrypoint. Its
  long-lived process still owns background timers and process-lifetime macOS
  access; menu visibility is not a prerequisite for those jobs. Keep shared
  domain behavior in `core/` rather than making menu presentation its owner.
- `mcp_server.py` is the direct FastMCP entrypoint.
- MCP is an optional legacy read-only compatibility surface. It may query and
  summarize, but must not mutate bookmarks/groups or touch WeChat UI. The old
  send tool names are inert `mcp_send_retired` stubs; legacy send config keys
  remain loadable but inactive. Do not restore sender/policy/confirmation
  modules or a send-side effect path. Ordinary app installation and startup
  must not require the MCP SDK, probe MCP readiness/processes, or manage an MCP
  client. `requirements-mcp.txt` owns the opt-in SDK dependency. Keep explicit
  stdio setup separate and client-neutral; the configured client owns the
  optional server's lifecycle.
- `core/monitor_state.py::MonitorStateStore` is the sole monitor-checkpoint
  write authority. Existing state migrates only after a valid parse; corrupt,
  symlink or non-regular state fails closed. TopicMonitor commits one expected
  revision after successful work and never advances source state after an AI
  failure. A retryable AI failure may CAS-patch only content-free failure
  count/code/timestamps and the bounded backoff deadline against the revision
  that was read; source cursors, inventory bindings and checkpoints must remain
  byte-for-byte unchanged, and a run inside that backoff makes no provider call.
  `core/monitor_source.py` owns the bounded raw-row reader. Canonical progress
  is per chat x logical-shard generation in `source_cursors`; `last_checked_ts`
  is derived compatibility evidence, not source authority. Raw pages merge by
  `create_time` then `source_message_id`, and only actually consumed rows may
  advance tentative cursors. Filtered rows advance without entering an AI
  prompt; `no_messages` requires verified raw EOF under the same complete
  inventory. A generation change or state-revision conflict commits no cursor.
  Knowledge events created before a state conflict reuse their stable
  `source_batch_id` on retry and must not create a second canonical event. If
  that committed event's managed topic Markdown is missing, reuse repairs the
  projection and date indexes without changing canonical event identity.
  Catch-up apply must stop the managed LaunchAgent, acquire the same
  `AppInstanceLock` as the menu app, hold it through backup/drain/projection/
  validation and a durable provisional receipt, release it, and only then
  restore the LaunchAgent. The same `run_id` receipt is atomically finalized
  with the observed restore result; it must not report `complete` while restore
  is pending or after restore failed.
- `core/source_inventory.py::SourceInventoryStore` owns the durable expected
  message-shard set. Logical shard identity is source namespace plus normalized
  relative path; database generation remains separate. Missing, keyless,
  cache-only, unreadable, and changed-generation states must remain explicit.
  Monitor/catch-up may read only a complete inventory. Resource and Direct
  Drive scanners may consume present generations while reporting
  `source_degraded`; they must never relabel that partial observation complete.
- `scripts/health_check.py` is the canonical operator health matrix. Its
  default output is path-free and content-free: no chat identity/body,
  source-relative path, local absolute path, API endpoint, or token. It must
  distinguish monitor healthy/missing/corrupt/conflict, source inventory
  completeness/counts, raw cursor progress, mounted handoff with provider sync
  unknown, separate Direct Drive verification, disabled link preview, legacy
  read-only/send-retired MCP, and the Windows shared storage/credential foundation boundary.
  Health inspection must not scan source, initialize/migrate the source
  inventory, open CAS payload objects, `stat` the protected WeChat container,
  or promote mounted evidence to remote
  verification. Local details require explicit `--sensitive`.
- `setup.py` is the py2app packaging entrypoint. These are the only Python
  files that belong at repository root.
- `core/` owns domain behavior, durable state, privacy boundaries, recovery,
  backup and projection contracts.
- `core/config.py::ConfigStore` is the sole main-config write authority. Writers
  patch the latest locked revision; do not reintroduce whole-snapshot UI/CLI
  saves or non-atomic config writes. `core/app_runtime.py` owns the menu-app
  process singleton.
- `core/url_safety.py` is the sole URL display/export/prompt redaction
  authority. Exact observed URLs and their stable hashes remain unchanged only
  in private durable ledgers. Human-readable projections, snapshots/exports,
  Review Queue, Daily Digest, AI prompts and surfaced errors must use the
  canonical redacted form. The in-process remote link preview path is retired
  and inert: legacy `monitor_fetch_links: true` must still produce
  `link_preview_disabled` with zero network requests. Legacy mounted

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IndelibleVivi/we-groupchat-obsidian](https://github.com/IndelibleVivi/we-groupchat-obsidian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
