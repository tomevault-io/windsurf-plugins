---
trigger: always_on
description: This document provides comprehensive guidelines for writing OpenAPI annotations in the Sayna Rust/Axum codebase using the `utoipa` crate.
---

# OpenAPI Documentation Guidelines

This document provides comprehensive guidelines for writing OpenAPI annotations in the Sayna Rust/Axum codebase using the `utoipa` crate.

## Overview

Sayna uses `utoipa` (v5.3+, latest is v5.4) for OpenAPI 3.1 specification generation. All OpenAPI-related code is feature-gated behind the `openapi` feature flag to keep dependencies minimal in production builds.

### Key Principles

1. **Feature-Gated**: All OpenAPI code must be conditionally compiled with `#[cfg_attr(feature = "openapi", ...)]`
2. **Centralized Documentation**: All paths, schemas, and tags are registered in `src/docs/openapi.rs`
3. **Complete Examples**: Every field should have meaningful examples that reflect real-world usage
4. **Consistent Style**: Follow established patterns for naming, descriptions, and error responses
5. **Type Safety**: Leverage Rust's type system to ensure documentation matches implementation

## Feature Flag Setup

### Cargo.toml Configuration

```toml
[features]
openapi = [
    "dep:utoipa",
]

[dependencies]
utoipa = { version = "5.3", optional = true, features = ["axum_extras"] }

# Optional: For more ergonomic route registration
# utoipa-axum = { version = "0.2", optional = true }
```

### Optional: Enhanced Axum Integration

Consider adding `utoipa-axum` for more ergonomic route registration:

```rust
use utoipa_axum::{routes, router::OpenApiRouter};

let (router, api) = OpenApiRouter::new()
    .routes(routes!(health_check, list_voices, speak_handler))
    .split_for_parts();
```

### Running with OpenAPI

```bash
# Run server with OpenAPI endpoints
cargo run --features openapi

# Generate OpenAPI spec to file
cargo run --features openapi -- openapi -o docs/openapi.yaml

# Generate JSON format
cargo run --features openapi -- openapi --format json -o docs/openapi.json
```

## Schema Annotations

### Basic Schema Definition

Use `#[cfg_attr(feature = "openapi", derive(utoipa::ToSchema))]` on all types exposed in the API:

```rust
use serde::{Deserialize, Serialize};

/// Health check response
#[derive(Debug, Serialize, Deserialize)]
#[cfg_attr(feature = "openapi", derive(utoipa::ToSchema))]
pub struct HealthResponse {
    /// Server status
    #[cfg_attr(feature = "openapi", schema(example = "OK"))]
    pub status: String,
}
```

### Field-Level Annotations

#### Examples

Always provide realistic examples for every field:

```rust
#[derive(Debug, Deserialize)]
#[cfg_attr(feature = "openapi", derive(utoipa::ToSchema))]
pub struct TokenRequest {
    /// The LiveKit room name to generate a token for
    #[cfg_attr(feature = "openapi", schema(example = "conversation-room-123"))]
    pub room_name: String,

    /// Display name for the participant (e.g., "John Doe")
    #[cfg_attr(feature = "openapi", schema(example = "Alice Smith"))]
    pub participant_name: String,

    /// Unique identifier for the participant (e.g., "user-123")
    #[cfg_attr(feature = "openapi", schema(example = "user-alice-456"))]
    pub participant_identity: String,
}
```

#### Optional Fields and Defaults

For optional fields, use serde attributes to control serialization:

```rust
#[derive(Debug, Deserialize, Serialize, Clone)]
#[cfg_attr(feature = "openapi", derive(utoipa::ToSchema))]
pub struct TTSWebSocketConfig {
    /// Provider name (e.g., "deepgram")
    #[cfg_attr(feature = "openapi", schema(example = "deepgram"))]
    pub provider: String,

    /// Voice ID or name to use for synthesis
    #[serde(skip_serializing_if = "Option::is_none")]
    #[cfg_attr(feature = "openapi", schema(example = "aura-asteria-en"))]
    pub voice_id: Option<String>,

    /// Speaking rate (0.25 to 4.0, 1.0 is normal)
    #[cfg_attr(feature = "openapi", schema(example = 1.0))]
    pub speaking_rate: Option<f32>,

    /// List of participants (defaults to empty for all participants)
    #[serde(default, skip_serializing_if = "Vec::is_empty")]
    pub listen_participants: Vec<String>,
}
```

#### Numeric Constraints

Use schema attributes for numeric validations:

```rust
#[cfg_attr(feature = "openapi", derive(utoipa::ToSchema))]
pub struct AudioConfig {
    /// Sample rate of the audio in Hz
    #[cfg_attr(feature = "openapi", schema(example = 16000, minimum = 8000, maximum = 48000))]
    pub sample_rate: u32,

    /// Number of audio channels (1 for mono, 2 for stereo)
    #[cfg_attr(feature = "openapi", schema(example = 1, minimum = 1, maximum = 2))]
    pub channels: u16,
}
```

### Complex Types

#### Enums

For enum types used in discriminated unions:

```rust
#[derive(Debug, Serialize, Deserialize)]
#[cfg_attr(feature = "openapi", derive(utoipa::ToSchema))]
#[serde(tag = "type")]
pub enum IncomingMessage {
    #[serde(rename = "config")]
    Config {
        /// Enable audio processing (STT/TTS). Defaults to true if not specified.
        #[serde(default = "default_audio_enabled")]
        audio: Option<bool>,

        /// STT configuration (required only when audio=true)
        #[serde(skip_serializing_if = "Option::is_none")]
        stt_config: Option<STTWebSocketConfig>,

        /// TTS configuration (required only when audio=true)
        #[serde(skip_serializing_if = "Option::is_none")]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SaynaAI/sayna](https://github.com/SaynaAI/sayna) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
