---
trigger: always_on
description: > Project Guide for AI Coding Agents
---

# AGENTS.md

> Project Guide for AI Coding Agents

---

## Project Overview

**url-shortener** is a high-performance URL shortening service built with Rust.

### Core Features

| Feature | Description |
|---------|-------------|
| URL Shortening | Collision-free short key generation using Base62 encoding |
| Deep Links | iOS/Android app deep links + platform-specific fallback URLs |
| OG Tags | Social media link previews |
| Webhooks | Real-time notifications on URL access (Semaphore concurrency control) |
| Redis Caching | High-speed caching with MessagePack serialization |
| JWT Auth | Token-based API authentication |
| Rate Limiting | SmartIP-based request throttling |

### Tech Stack

| Area | Technology |
|------|------------|
| Language | Rust 2021 Edition |
| Web Framework | Axum 0.8 |
| Async Runtime | Tokio |
| Database | PostgreSQL (SQLx) |
| Cache | Redis (deadpool-redis) + MessagePack |
| Templates | Askama |
| Error Handling | thiserror |
| Logging | tracing |
| Hashing | xxhash-rust (xxh3_128) |
| Memory Allocator | mimalloc |

---

## Architecture

```mermaid
flowchart TB
    subgraph Client["🌐 Client"]
        Browser[Browser]
        Mobile[Mobile App]
    end

    subgraph Server["⚡ API Server (Axum)"]
        Router[Router]
        RateLimit[Rate Limiter]
        Auth[JWT Auth Middleware]
        Handler[Handler]
    end

    subgraph Storage["💾 Storage"]
        Redis[(Redis Cache)]
        PostgreSQL[(PostgreSQL)]
    end

    subgraph External["🔔 External"]
        Webhook[Webhook Endpoint]
        Sentry[Sentry Error Tracking]
    end

    Browser --> Router
    Mobile --> Router
    Router --> RateLimit
    RateLimit --> Auth
    Auth --> Handler
    Handler <--> Redis
    Handler <--> PostgreSQL
    Handler -.->|async| Webhook
    Handler -.->|error report| Sentry
```

### Data Flow

```mermaid
flowchart LR
    subgraph Create["URL Creation"]
        C1[API Request] --> C2[JWT Validation]
        C2 --> C3[Input Validation]
        C3 --> C4[xxHash Generation]
        C4 --> C5[DB Save<br/>ON CONFLICT]
        C5 --> C6[Return Short Key]
    end
```

```mermaid
flowchart LR
    subgraph Redirect["URL Redirect"]
        R1[Short Key Request] --> R2[Parse short_key]
        R2 --> R3{Cache Lookup}
        R3 -->|Hit| R5[Validate random_key]
        R3 -->|Miss| R4[DB Query → Update Cache]
        R4 --> R5
        R5 --> R6[Platform Detection]
        R6 --> R7[Webhook Call<br/>async]
        R6 --> R8[Redirect Page]
    end
```

---

## Project Structure

```
src/
├── main.rs                 # Entry point, server bootstrap
├── lib.rs                  # Library crate
├── error.rs                # Centralized error types (AppError, AppResult)
├── api/
│   ├── mod.rs              # API module exports
│   ├── handlers.rs         # HTTP handlers (index, create_short_url, redirect)
│   ├── routes.rs           # Route definitions + middleware setup
│   ├── schemas.rs          # Request/Response DTOs + validation
│   ├── middlewares.rs      # JWT auth middleware (AuthUser)
│   └── state.rs            # AppState (DB pool, Redis pool)
├── config/
│   ├── mod.rs              # Config module exports
│   ├── env.rs              # Environment variable loading (APP_CONFIG)
│   ├── db.rs               # PostgreSQL connection pool
│   └── cache.rs            # Redis connection pool
├── models/
│   ├── mod.rs              # Models module exports
│   └── url.rs              # Url, UrlCacheData, UrlRepository
└── utils/
    ├── mod.rs              # Utils module exports
    ├── jwt.rs              # JWT generation/parsing (gen_token, parse_token)
    ├── rand.rs             # Random string generation
    └── short_key.rs        # Base62 encoding/decoding

tests/
└── integration_test.rs     # Integration tests (runnable without DB)

migrations/                 # SQLx migration files
views/                      # Askama HTML templates
```

---

## Core Modules

### Error Handling (`src/error.rs`)

```mermaid
flowchart TD
    subgraph Errors["Error Types"]
        BadRequest["BadRequest (400)"]
        Unauthorized["Unauthorized (401)"]
        NotFound["NotFound (404)"]
        Validation["Validation (400)"]
        Internal["Internal (500)"]
    end

    subgraph From["Auto Conversion (#from)"]
        SqlxError[sqlx::Error]
        RedisError[redis::RedisError]
        JwtError[jsonwebtoken::Error]
        AskamaError[askama::Error]
        ReqwestError[reqwest::Error]
    end

    SqlxError --> Internal
    RedisError --> Internal
    JwtError --> Unauthorized
    AskamaError --> Internal
    ReqwestError --> Internal
```

```rust
#[derive(Error, Debug)]
pub enum AppError {
    BadRequest(String),      // 400
    Unauthorized(String),    // 401
    NotFound(String),        // 404
    Validation(String),      // 400
    Internal(String),        // 500
    Database(#[from] sqlx::Error),
    Redis(#[from] deadpool_redis::redis::RedisError),
    // ...
}

pub type AppResult<T> = Result<T, AppError>;
```

### Handler Pattern (`src/api/handlers.rs`)

```rust
pub async fn handler_name(
    State(state): State<AppState>,
    Extension(auth_user): Extension<AuthUser>,  // if auth required
    Json(body): Json<RequestType>,
) -> AppResult<impl IntoResponse> {
    // 1. Validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lee-lou2/url-shortener](https://github.com/lee-lou2/url-shortener) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
