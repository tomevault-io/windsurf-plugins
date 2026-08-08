---
trigger: always_on
description: > **rmux** — Cross-platform, memory-efficient terminal multiplexer GUI, written in Rust.
---

# AGENTS.md — rmux Project Instructions

> **rmux** — Cross-platform, memory-efficient terminal multiplexer GUI, written in Rust.
> Inspired by [cmux](https://github.com/manaflow-ai/cmux). Targets Linux, macOS, Windows.

---

## Quick Reference

| Item | Value |
|---|---|
| Language | Rust (edition 2024) |
| GUI framework | `egui` + `eframe` |
| Terminal emulator | `alacritty_terminal` |
| PTY | `portable-pty` |
| Async runtime | `tokio` |
| Browser pane | `wry` |
| Notifications | `notify-rust` |
| Min RAM target | < 100 MB with 20 panes |
| Platforms | Linux, macOS, Windows |

---

## Project Plan

**The plan is the source of truth.** Read it before doing anything:

→ [`docs/PLAN.md`](docs/PLAN.md)

### How to work on tasks

1. **Read the current phase** in `docs/PLAN.md`
2. **Pick the next unmarked task** (marked with `[ ]`)
3. **Implement it** following the conventions below
4. **Write tests** for the task
5. **Run verification** (see "Verification Checklist" below)
6. **Mark the task done** in `docs/PLAN.md` (change `[ ]` to `[x]`)
7. **Commit and push** after the implementation is complete

### Phase progression

Do NOT skip phases. Each phase depends on the previous one. Within a phase, tasks are ordered — do them sequentially unless they are clearly independent.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    rmux-app (binary)                     │
│  ┌─────────┐  ┌──────────────┐  ┌───────────────────┐   │
│  │   UI    │  │  Workspace   │  │  Notifications    │   │
│  │ (egui)  │  │  Manager     │  │  Manager          │   │
│  └────┬────┘  └──────┬───────┘  └─────────┬─────────┘   │
│       │              │                     │             │
│  ┌────▼──────────────▼─────────────────────▼─────────┐   │
│  │              Application State (app.rs)           │   │
│  └────┬──────────────┬─────────────────────┬─────────┘   │
│       │              │                     │             │
│  ┌────▼────┐  ┌──────▼───────┐  ┌─────────▼─────────┐   │
│  │ rmux-   │  │  rmux-api    │  │  rmux-config      │   │
│  │terminal │  │ (socket srv) │  │ (config mgmt)     │   │
│  └─────────┘  └──────────────┘  └───────────────────┘   │
└─────────────────────────────────────────────────────────┘
         ▲
         │
┌────────▼────────┐
│   rmux-cli      │
│ (separate bin)  │
└─────────────────┘
```

### Crate responsibilities

| Crate | Type | Purpose |
|---|---|---|
| `rmux-app` | binary | Main application. Owns the egui window, event loop, and orchestrates all subsystems. |
| `rmux-terminal` | library | Terminal emulation. Wraps `alacritty_terminal` + `portable-pty`. Owns PTY lifecycle, grid state, scrollback. |
| `rmux-cli` | binary | CLI tool (`rmux-cli` command). Connects to socket, sends commands, prints results. |
| `rmux-api` | library | Socket server. JSON-RPC protocol, method dispatch, event streaming. |
| `rmux-config` | library | Configuration loading/saving. `rmux.json` schema, Ghostty config import. |

### Data flow

```
User keyboard → egui event → TerminalPane → PtyBackend.write()
                                                  │
PTY output → PtyBackend.read() → TermState.feed() → Grid updated
                                                  │
egui render → TerminalRenderer.draw(grid) → Screen pixels
                                                  │
OSC sequence → NotificationManager → Sidebar badge + desktop notification
```

---

## Rust Best Practices (Mandatory)

### Error Handling

```rust
// Library crates: use thiserror
use thiserror::Error;

#[derive(Error, Debug)]
pub enum TermError {
    #[error("PTY spawn failed: {0}")]
    PtySpawn(#[from] std::io::Error),
    #[error("Resize failed: cols={cols}, rows={rows}")]
    Resize { cols: u16, rows: u16 },
}

// Application code: use anyhow
use anyhow::{Context, Result};

fn load_config() -> Result<Config> {
    let path = config_path();
    let content = std::fs::read_to_string(&path)
        .context("Failed to read config file")?;
    serde_json::from_str(&content)
        .context("Failed to parse config JSON")
}
```

**Rules:**
- NEVER use `.unwrap()` in production code
- Use `.expect("specific reason")` only when the invariant is provably true
- Always use `?` or `.context()` to propagate errors
- Log errors with `tracing::error!` before returning them

### Concurrency

```rust
// Prefer channels over shared mutexes
let (tx, mut rx) = tokio::sync::mpsc::channel::<PtyEvent>(256);

// Spawn PTY reader on separate task
tokio::spawn(async move {
    while let Some(event) = rx.recv().await {
        // handle event
    }
});

// NEVER hold a lock across .await
let data = lock.read().await;  // OK: short hold
process(data).await;           // lock is already dropped

// BAD:
let mut data = lock.write().await;
data.update().await;  // holding write lock across await — DEADLOCK RISK
```

**Rules:**
- Use `tokio::sync::mpsc` for producer-consumer patterns
- Use `Arc<RwLock<T>>` only for read-heavy shared state
- NEVER hold a `Mutex` or `RwLock` across `.await`
- Use `tokio::select!` for concurrent event handling

### Module Organization

```rust
// src/ui/mod.rs — public interface
mod sidebar;
mod terminal_pane;
mod workspace_view;


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nakulbh/rmux](https://github.com/nakulbh/rmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
