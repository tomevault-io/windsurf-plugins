---
trigger: always_on
description: Manages MCP server lifecycle: start, monitor, restart, configure. Generates OpenClaw MCP
---

# CLAUDE.md — osModa

## What this is

osModa: NixOS distribution with AI-native system management.
The agent runs via Claude Code SDK with full root-level access to the entire system via structured daemons.

The agent has FULL system access. Root. All files. All processes. All APIs.
The sandbox exists for UNTRUSTED third-party tools, not for the agent itself.

## Architecture (3 trust tiers)

```
TIER 0: Claude Code + agentd (full system, root-equivalent)
  ↓ grants capabilities to ↓
TIER 1: Approved apps (sandboxed, declared capabilities)
  ↓ even more restricted ↓
TIER 2: Untrusted tools (max isolation, no network, minimal fs)
```

## Components

1. **agentd** (Rust) — system bridge daemon. Unix socket API at `/run/osmoda/agentd.sock`.
   Gives OpenClaw structured access to: processes, services, network, filesystem, NixOS config,
   sysctl parameters, users, firewall. Append-only hash-chained event log in SQLite.
   Memory system endpoints for ingest/recall/store.
   Agent Card (EIP-8004) identity + capability discovery.
   Structured receipts + incident workspaces for auditable troubleshooting.

2. **osmoda-gateway** (TypeScript) — **Modular agent gateway** (v0.2+). HTTP+WS server on port 18789.
   Always the systemd unit. Routes per-agent to a pluggable runtime driver:
   - `claude-code` driver — wraps `claude` CLI (OAuth or API key)
   - `openclaw` driver — spawns `openclaw` binary as child process per session
   - Adding a driver = one file under `src/drivers/` (Codex, Bedrock, …)
   Hot-reloadable config via `agents.json` (SIGHUP re-reads, in-flight sessions keep
   their snapshot — zero WS drops). Encrypted credential store at
   `/var/lib/osmoda/config/credentials.json.enc` (AES-256-GCM). REST `/config/*`
   endpoints (Bearer-authed) let the dashboard edit runtime/credentials/model per
   agent with no SSH or rebuild. Telegram webhook, WebSocket chat, 91 MCP tools.

2b. **osmoda-bridge** (TypeScript) — Legacy OpenClaw plugin. Registers tools via
   `api.registerTool()` factory pattern (91 tools): system_health, system_query,
   system_discover, event_log, memory_store, memory_recall, shell_exec, file_read,
   file_write, directory_list, service_status, journal_logs, network_info,
   wallet_create, wallet_list, wallet_sign, wallet_send, wallet_delete, wallet_receipt,
   wallet_build_tx,
   safe_switch_begin, safe_switch_list, safe_switch_status, safe_switch_commit, safe_switch_rollback,
   watcher_add, watcher_list, routine_add, routine_list, routine_trigger,
   agent_card, receipt_list, incident_create, incident_step,
   voice_status, voice_speak, voice_transcribe, voice_record, voice_listen,
   backup_create, backup_list,
   mesh_identity, mesh_invite_create, mesh_invite_accept, mesh_peers,
   mesh_peer_send, mesh_peer_disconnect, mesh_health,
   mesh_room_create, mesh_room_join, mesh_room_send, mesh_room_history,
   mcp_servers, mcp_server_start, mcp_server_stop, mcp_server_restart,
   teach_status, teach_observations, teach_patterns, teach_knowledge,
   teach_knowledge_create, teach_context, teach_optimize_suggest, teach_optimize_apply,
   teach_skill_candidates, teach_skill_generate, teach_skill_promote,
   teach_observe_action, teach_skill_execution, teach_skill_detect,
   approval_request, approval_pending, approval_approve, approval_check,
   sandbox_exec, capability_mint,
   fleet_propose, fleet_status, fleet_vote, fleet_rollback,
   app_deploy, app_list, app_logs, app_stop, app_restart, app_remove,
   safety_rollback, safety_status, safety_panic, safety_restart.

3. **osmoda-egress** (Rust) — localhost-only HTTP CONNECT proxy. Domain allowlist
   per capability token. Only path to internet for sandboxed tools.

4. **osmoda-keyd** (Rust) — OS-native crypto wallet daemon. Unix socket at `/run/osmoda/keyd.sock`.
   AES-256-GCM encrypted keys, policy-gated signing (daily limits), ETH + SOL wallets.
   Runs with PrivateNetwork=true (zero network access). Keys never leave keyd.
   SignerBackend trait allows future MPC/HSM/Vault integration.

5. **osmoda-watch** (Rust) — SafeSwitch + autopilot watchers. Unix socket at `/run/osmoda/watch.sock`.
   Deploy transactions with timer + health gates + auto-rollback.
   Autopilot watchers: deterministic health checks with escalation (restart → rollback → notify).

6. **osmoda-routines** (Rust) — background cron/event/webhook automation engine.
   Unix socket at `/run/osmoda/routines.sock`.
   Runs scheduled tasks between agent conversations (health checks, service monitors, log scans).
   Default routines match HEARTBEAT.md cadences.

7. **osmoda-voice** (Rust) — Local speech-to-text (whisper.cpp) + text-to-speech (piper).
   Unix socket at `/run/osmoda/voice.sock`. All processing on-device. No cloud APIs.

8. **osmoda-mesh** (Rust) — P2P encrypted agent-to-agent communication daemon. Unix socket at `/run/osmoda/mesh.sock`,
   TCP listener at port 18800. Noise_XX (X25519/ChaChaPoly/BLAKE2s) + ML-KEM-768 hybrid post-quantum.
   Invite-based pairing, no central server. Ed25519 identity signatures.

9. **osmoda-mcpd** (Rust) — MCP server manager daemon. Unix socket at `/run/osmoda/mcpd.sock`.
   Manages MCP server lifecycle: start, monitor, restart, configure. Generates OpenClaw MCP

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bolivian-peru/os-moda](https://github.com/bolivian-peru/os-moda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
