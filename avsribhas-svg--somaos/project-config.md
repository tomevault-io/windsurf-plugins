---
trigger: always_on
description: > Read this before touching any code. Updated 2026-03-22 (v1.2 complete — Capability Governance + Session Scope + Semantic FS SQLite + soma-media, 70/70 tests passing).
---

# CLAUDE.md — SomaOS

> Read this before touching any code. Updated 2026-03-22 (v1.2 complete — Capability Governance + Session Scope + Semantic FS SQLite + soma-media, 70/70 tests passing).

---

## What This Is

**SomaOS** is an AI-native operating system where the agent is a first-class user of the desktop — not a chatbot bolted on top of Linux. Both the human and the agent are **peer co-inhabitants** of the same environment:

- **Human interface**: pixels — dock clicks, window drags, keyboard shortcuts
- **Agent interface**: structured APIs — `AgentAPI`, IPC messages, capability actions
- **Shared**: same apps, same windows, same data models
- **HITL**: conflict resolution for a shared space, not just a permission gate

This is a thesis project. Every architectural decision should serve the core thesis, not general-purpose OS engineering.

---

## Language & Stack

- **Primary**: Rust workspace (`soma-common`, `soma-agent`, `soma-compositor`, `soma-cli`)
- **Secondary frontend**: React + Tauri 2 in `/soma/` — macOS dev only, not production
- **OS image**: Buildroot (x86_64 + ARM64) in `/buildroot/`
- **CI**: GitHub Actions (`.github/workflows/build.yml`) — builds both arch images on push to main

---

## Workspace Layout

```
soma-common/        Shared IPC types (TaskPlan, BrowserUpdate, CompositorMessage, AgentMessage)
soma-agent/         Agent daemon (tokio, LlmProvider trait, capability registry)
soma-compositor/    Compositor binary (DRM/KMS + tiny-skia + cosmic-text)
soma-cli/           CLI test client
buildroot/          OS image build system (soma_defconfig)
docs/               ANALYSIS.md, BUILD_ARM64.md, BUILD_x86_64.md, WINDOWS_BUILD.md
soma/               React + Tauri 2 macOS dev frontend
```

---

## Current Version: v1.2 (as of 2026-03-22)

### What was built in v1.2

**Capability Governance**
- `meta.list_governance`: lists all built-in + script capabilities with type/version/load metadata
- `meta.promote`: generates a Rust stub at `~/.soma/promotions/<name>.rs` for promoting script caps to built-ins
- Sidebar Settings tab: Reload button for hot-reloading script capabilities from `~/.soma/capabilities/`

**Advanced Session Model**
- `SessionScope { capability_whitelist: Option<Vec<String>>, path_whitelist: Option<Vec<String>> }` added to soma-common
- `AgentModeStarted` now carries `scope: Option<SessionScope>`
- `GetSessionStatus` / `SessionStatusResponse` IPC round-trip
- Scope enforcement in `ipc.rs`: capability whitelist + path whitelist checked before each step execution
- `desktop_agent.start_agent_mode` accepts optional `scope` param
- `desktop_agent.get_session_status` action added
- Sidebar Chat tab shows active session card (task name + scope)

**Semantic FS — SQLite backend**
- `rusqlite` (bundled) replaces `.soma-meta` sidecar files
- `~/.soma/index.db`: SQLite database with `files` table (path, description, tags, created_by, created_at, history)
- Best-effort migration from existing `.soma-meta` sidecar files on first run
- Same `Capability` interface (action names/params unchanged) — only the storage backend changed

**soma-media (third dual-interface app)**
- `MediaApp` implementing `NativeAppContent`: prompt bar (top), image display (center), status strip (bottom)
- `on_key("Enter")` submits prompt; `set_image` accepts base64 PNG bytes
- `media` capability: `generate` (command pattern → `AppAction`), `describe`, `save`
- `WindowContentType::Media`, dock entry, event routing in `event_handler.rs`
- Layer 0 fast-path patterns for `media_generate`, `media_describe`, `media_save`

**Test Suite**
- 61 → 70 scenarios covering all 14 capability modules (70/70 passing)

---

### What was built in v1.1

**AgentAPI + Dual-Interface Apps**
- `AgentAPI` / `NativeAppContent` trait: `describe_state`, `execute_action`, `on_char/on_key/on_click/render`
- `WindowContent::NativeApp(Box<dyn NativeAppContent>)` — first dual-interface window type
- `AppState`, `AppStateCache`, `AppStateChanged`, `AppAction` IPC — compositor pushes state to agent on every edit
- `soma-sheets`: spreadsheet with formula evaluator (SUM/AVG/MIN/MAX/COUNT, cell refs, ranges), Tab/Enter/Arrow navigation, formula bar, number right-align, text left-align
- `soma-docs`: block-based document editor (paragraphs, headings, code blocks), same dual-interface pattern
- `sheets` agent capability: `create`, `describe`, `read_range`, `write_cell`, `apply_formula`
- `docs` agent capability: `create`, `describe`, `write_block`, `read_blocks`
- `semantic_fs` capability: `tag`, `annotate`, `find_by_intent`, `list_tagged`, `describe_file`, `get_history`

**Test Suite + Agent Robustness**
- 61/61 scenario integration test suite covering all 13 capability modules
- Layer 0 fast-path: 30+ keyword interceptors — unambiguous intents never hit the LLM (0ms)
- Ollama text-fallback parser: handles bare params, `{"function":}`, `{"cmd":}`, `{" functions":[]}` wrappers
- Session tracking in `ipc.rs`: `Session` + `SessionStep` persisted to `~/.soma/sessions/`

---

### What was built in v1.0

**Multi-provider LLM brain**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avsribhas-svg/SomaOS](https://github.com/avsribhas-svg/SomaOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
