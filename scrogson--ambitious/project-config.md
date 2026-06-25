---
trigger: always_on
description: A native Rust implementation of Erlang/OTP primitives, bringing the power of the BEAM's concurrency model to Rust with full type safety.
---

# Ambitious - Distributed Rust Erlang Abstract Machine

A native Rust implementation of Erlang/OTP primitives, bringing the power of the BEAM's concurrency model to Rust with full type safety.

## Project Vision

Ambitious aims to provide Erlang-style concurrency primitives in Rust:
- **Processes**: Lightweight, isolated units of concurrency with mailboxes
- **Message Passing**: Type-safe send/receive between processes
- **Links & Monitors**: Bidirectional failure propagation and unidirectional observation
- **GenServer**: Request/response pattern with typed state management
- **Supervisor**: Fault-tolerant supervision trees with configurable restart strategies
- **Application**: OTP-style application lifecycle management

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   Application Layer                         │
│  (Application trait, dependency management)                 │
└─────────────────────────────────────────────────────────────┘
                              ▲
┌─────────────────────────────────────────────────────────────┐
│              Supervision Layer                              │
│  (Supervisor, ChildSpec, restart strategies)                │
└─────────────────────────────────────────────────────────────┘
                              ▲
┌─────────────────────────────────────────────────────────────┐
│           GenServer Pattern Layer                           │
│  (GenServer trait, call/cast/info, typed messages)          │
└─────────────────────────────────────────────────────────────┘
                              ▲
┌─────────────────────────────────────────────────────────────┐
│         Process Base Layer                                  │
│  (Pid, spawn, link, monitor, send/receive, mailbox)         │
└─────────────────────────────────────────────────────────────┘
                              ▲
┌─────────────────────────────────────────────────────────────┐
│              Runtime Layer                                  │
│  (Scheduler, process registry, async executor)              │
└─────────────────────────────────────────────────────────────┘
```

## Crate Structure

```
ambitious/
├── crates/
│   ├── ambitious/              # Main crate containing all functionality
│   │   └── src/
│   │       ├── core/           # Core types: Pid, Ref, ExitReason, Atom, Term trait
│   │       ├── runtime/        # Async runtime, scheduler, process registry, task-locals
│   │       ├── process/        # Process spawning, mailboxes, links, monitors
│   │       ├── gen_server/     # GenServer trait and server loop
│   │       ├── supervisor/     # Supervisor trait and restart strategies
│   │       ├── application/    # Application lifecycle management
│   │       ├── timer/          # Timer management (send_after, cancel)
│   │       └── ...             # Registry, channels, distribution, etc.
│   └── ambitious-macros/       # Procedural macros (#[derive(Message)], #[ambitious::main])
```

## Core Types

### Pid (Process Identifier)
```rust
pub struct Pid {
    node: u32,  // For future distribution support
    id: u64,    // Unique process ID
}
```

### Ref (Monitor/Timer Reference)
```rust
pub struct Ref(u64);  // Unique, atomically generated
```

### ExitReason
```rust
pub enum ExitReason {
    Normal,
    Shutdown,
    ShutdownReason(String),
    Killed,
    Error(String),
}
```

## Process API (mirrors Elixir's Process module)

```rust
// Spawning
fn spawn<F, T>(f: F) -> Pid;
fn spawn_link<F, T>(f: F) -> Pid;
fn spawn_monitor<F, T>(f: F) -> (Pid, Ref);

// Links (bidirectional)
fn link(pid: Pid) -> Result<(), SendError>;
fn unlink(pid: Pid);

// Monitors (unidirectional)
fn monitor(pid: Pid) -> Result<Ref, SendError>;
fn demonitor(reference: Ref);

// Messaging
fn send<M: Term>(pid: Pid, msg: &M) -> Result<(), SendError>;
fn send_after<M: Term>(pid: Pid, msg: &M, delay: Duration) -> TimerResult;

// Process flags
fn flag(flag: ProcessFlag, value: bool) -> bool;  // trap_exit, etc.

// Exit signals
fn exit(pid: Pid, reason: ExitReason);

// Info
fn alive(pid: Pid) -> bool;
fn current_pid() -> Pid;

// Registration
fn register(name: String, pid: Pid);
fn whereis(name: &str) -> Option<Pid>;
fn unregister(name: &str);
```

## GenServer API (mirrors Elixir's GenServer)

### Trait Definition
```rust
#[async_trait]
pub trait GenServer: Sized + Send + 'static {
    type Args: Send + 'static;
    type Call: Message + Send + 'static;
    type Cast: Message + Send + 'static;
    type Info: Message + Send + 'static;
    type Reply: Message + Send + 'static;

    async fn init(args: Self::Args) -> Init<Self>;
    async fn handle_call(&mut self, msg: Self::Call, from: From) -> Reply<Self::Reply>;
    async fn handle_cast(&mut self, msg: Self::Cast) -> Status;
    async fn handle_info(&mut self, msg: Self::Info) -> Status;
    async fn terminate(&mut self, _reason: ExitReason) {}
    async fn handle_timeout(&mut self) -> Status { Status::Ok }
    async fn handle_continue(&mut self, _arg: Vec<u8>) -> Status { Status::Ok }
}
```

Key design points:
- The struct IS the state (no separate `State` type)
- Handlers take `&mut self`
- `init` returns `Init<Self>`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scrogson/ambitious](https://github.com/scrogson/ambitious) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
