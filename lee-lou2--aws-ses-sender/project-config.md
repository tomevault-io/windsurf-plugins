---
trigger: always_on
description: > Project Guide for AI Coding Agents
---

# AGENTS.md

> Project Guide for AI Coding Agents

---

## 📋 Project Overview

**aws-ses-sender** is a high-performance bulk email sending service via AWS SES.

### Key Features
- 🚀 **Bulk Email Sending**: Process up to 10,000 emails per request
- ⏰ **Scheduled Sending**: Schedule future deliveries via `scheduled_at` field
- 📊 **Event Tracking**: Receive Bounce/Complaint/Delivery events via AWS SNS
- 👀 **Open Tracking**: Track email opens via 1x1 transparent pixel
- ⚡ **Rate Limiting**: Token Bucket + Semaphore-based sends per second control
- 🔐 **API Key Authentication**: Secure API access via X-API-KEY header
- 📊 **Sentry Integration**: Real-time error tracking and monitoring
- 🚀 **High-Performance Allocator**: Uses mimalloc

### Tech Stack
| Area | Technology |
|------|------|
| Language | Rust 2021 Edition |
| Web Framework | Axum 0.8 |
| Async Runtime | Tokio |
| Database | SQLite (WAL mode) |
| Email Service | AWS SES v2 |
| Authentication | X-API-KEY header |
| Error Tracking | Sentry |
| Memory Allocator | mimalloc |

---

## 🏗 Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  API Server │────▶│  Scheduler  │────▶│   Sender    │────▶│  AWS SES    │
│   (Axum)    │     │  (Batch)    │     │ (Rate Limit)│     │             │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
       │                   │                   │                   │
       │                   ▼                   ▼                   ▼
       │            ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
       └───────────▶│   SQLite    │◀────│ Post-Proc   │◀────│   AWS SNS   │
                    │   (WAL)     │     │  (Batch)    │     │  (Events)   │
                    └─────────────┘     └─────────────┘     └─────────────┘
```

### Data Flow
1. **Immediate Sending**: API → Batch INSERT → `try_send()` to channel → Rate-limited sending → CASE WHEN batch update
2. **Scheduled Sending**: API → Batch INSERT (Created) → Two-phase scheduler (UPDATE...RETURNING + JOIN) → Send channel → Sending → Result update

---

## 📁 Project Structure

```
├── migrations/             # SQLx database migrations (auto-applied on startup)
│   └── 20241228000000_initial_schema.sql
├── src/
│   ├── main.rs                 # Entry point, mimalloc, graceful shutdown
│   ├── app.rs                  # Axum router setup
│   ├── error.rs                # Centralized error handling (AppError, AppResult)
│   ├── constants.rs            # Shared constants (BATCH_INSERT_SIZE)
│   ├── state.rs                # AppState definition (DB pool, channels)
│   ├── config/                 # Configuration module
│   │   ├── mod.rs              # Module exports
│   │   ├── env.rs              # Environment variable loading (AppConfig)
│   │   └── db.rs               # Database connection pool management
│   ├── handlers/               # HTTP request handlers
│   │   ├── mod.rs
│   │   ├── message_handlers.rs # POST /v1/messages
│   │   ├── event_handlers.rs   # GET/POST /v1/events/*
│   │   ├── health_handlers.rs  # GET /health, /ready
│   │   └── topic_handlers.rs   # GET/DELETE /v1/topics/{id}
│   ├── services/               # Background services
│   │   ├── mod.rs
│   │   ├── scheduler.rs        # Scheduled email polling (10-second interval)
│   │   ├── receiver.rs         # Rate-limited sending + batch DB updates
│   │   └── sender.rs           # AWS SES API calls (singleton client, retry logic)
│   ├── models/                 # Data models
│   │   ├── mod.rs
│   │   ├── content.rs          # EmailContent (subject, content storage)
│   │   ├── request.rs          # EmailRequest, EmailMessageStatus
│   │   └── result.rs           # EmailResult
│   └── middlewares/            # HTTP middlewares
│       ├── mod.rs
│       └── auth_middlewares.rs # API Key authentication
└── Cargo.toml
```

---

## 🔑 Core Modules

### `src/main.rs`
- Application entry point
- mimalloc global allocator setup (non-MSVC targets)
- Logger, Sentry initialization
- SQLx migrations auto-applied via config/db.rs
- Spawns 3 background tasks
- Graceful shutdown with `tokio::signal::ctrl_c()`

### `src/error.rs`
**Centralized Error Handling** - All errors are converted to `AppError`

```rust
#[derive(Error, Debug)]
pub enum AppError {
    BadRequest(String),      // 400
    Unauthorized(String),    // 401
    NotFound(String),        // 404
    Validation(String),      // 400
    Internal(String),        // 500
    Database(#[from] sqlx::Error),
    Email(String),
    ChannelClosed,
}

pub type AppResult<T> = Result<T, AppError>;
```

### `src/config/`
**Configuration Module** - Environment variables and database management

```rust
// env.rs - Environment variable helpers
pub fn get_env(key: &str, default: Option<&str>) -> String;
pub fn get_env_parsed<T: FromStr>(key: &str, default: T) -> T;

// AppConfig - All settings in one struct
pub struct AppConfig {
    pub server_port: String,
    pub max_send_per_second: i32,
    pub db_max_connections: u32,
    // ...
}

pub static APP_CONFIG: Lazy<AppConfig> = Lazy::new(AppConfig::from_env);

// db.rs - Database pool management
pub async fn init_db() -> Result<SqlitePool, sqlx::Error>;
pub async fn close_db();
```

### `src/services/receiver.rs`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lee-lou2/aws-ses-sender](https://github.com/lee-lou2/aws-ses-sender) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
