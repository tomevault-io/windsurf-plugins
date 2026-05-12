---
trigger: always_on
description: **remix-mcp** is a Rust MCP (Model Context Protocol) server that enables AI assistants to control Ableton Live via OSC. It uses [AbletonOSC](https://github.com/ideoforms/AbletonOSC) as a bridge to Ableton's Live Object Model.
---

# CLAUDE.md

## Project Overview

**remix-mcp** is a Rust MCP (Model Context Protocol) server that enables AI assistants to control Ableton Live via OSC. It uses [AbletonOSC](https://github.com/ideoforms/AbletonOSC) as a bridge to Ableton's Live Object Model.

## Architecture

```
Claude/Client <--stdio/JSON-RPC--> remix-mcp <--UDP/OSC--> AbletonOSC <--> Ableton Live
```

- **MCP Server**: Rust binary using `rmcp` crate, communicates via stdio
- **OSC Client**: Async UDP client using `rosc` crate, lazy-initialized via `OscHandle`
- **AbletonOSC**: Python Remote Script running inside Ableton (bundled as git submodule, [our fork](https://github.com/christopherwxyz/AbletonOSC))

### Why OSC?

Ableton's Python API (Live Object Model) runs inside Ableton's sandboxed Python interpreter. External processes cannot call it directly. AbletonOSC bridges this gap via UDP/OSC on port 11000.

### Lazy OSC & Multi-Instance Support

`AbletonServer::new()` is sync and infallible — no sockets are opened until the first tool call. This ensures the MCP handshake always succeeds even when Ableton is not running.

The OSC layer uses a single UDP socket per instance (ephemeral port, no fixed port). Our AbletonOSC fork replies to the sender's actual address instead of hardcoded port 11001, so multiple remix-mcp instances (e.g. Claude Desktop + Claude Code) can control Ableton simultaneously without port contention.

## Development Commands

```bash
# Build
cargo build
cargo build --release

# Test
cargo test                    # Unit tests (no Ableton required)
cargo nextest run             # Faster test runner (recommended)
cargo insta test              # Run with snapshot review

# Lint
cargo clippy
cargo fmt
cargo deny check              # License/vulnerability checking

# Run (requires Ableton Live with AbletonOSC enabled)
cargo run

# Install AbletonOSC Remote Script
cargo run -- install

# Check installation status
cargo run -- status

# Integration tests (requires Ableton Live with AbletonOSC)
cargo test --test integration -- --ignored --test-threads=1
```

## Key Dependencies

- **CLI**: `clap` (argument parsing), `indicatif` (progress bars), `console` (styling)
- **Async**: `tokio` (runtime), `rosc` (OSC protocol)
- **Errors**: `color-eyre` (colorful error reports), `thiserror` (error types)
- **Logging**: `tracing` + `tracing-subscriber`
- **Serialization**: `serde` + `serde_json`
- **Testing**: `insta` (snapshots), `rstest` (parameterized tests)

## Project Structure

```
src/
├── main.rs          # CLI entry point (serve, install, status commands)
├── lib.rs           # Library exports
├── server.rs        # MCP ServerHandler impl, router composition
├── error.rs         # Error types
├── installer.rs     # AbletonOSC installer logic
├── osc/
│   ├── client.rs    # OscClient (single-socket UDP) + OscHandle (lazy init wrapper)
│   ├── message.rs   # OSC message helpers
│   └── response.rs  # FromOsc trait for parsing responses
├── tools/
│   ├── transport.rs # play, stop, record, tempo, etc.
│   ├── tracks.rs    # track volume, pan, mute, solo, sends
│   ├── clips.rs     # clip control, MIDI notes, clip properties
│   ├── scenes.rs    # scene firing and management
│   ├── devices.rs   # device parameters
│   ├── song.rs      # song info, undo/redo, loop settings
│   ├── view.rs      # selection (track, scene, clip, device)
│   ├── cue_points.rs # cue point navigation
│   └── browser.rs   # browser navigation, loading instruments/effects/presets
└── types/
    ├── params.rs      # TrackInfo, ClipInfo, MidiNote, etc.
    ├── tool_params.rs # Tool parameter structs with JsonSchema derive
    └── ids.rs         # ID types
```

## Code Conventions

### Adding New Tools

Tools are defined using rmcp's `#[tool]` and `#[tool_router]` procedural macros:

1. Add the tool method to the appropriate `#[tool_router]` impl block in `src/tools/*.rs`
2. If the tool takes parameters, add a params struct to `src/types/tool_params.rs`

Tool definition pattern:
```rust
// In src/types/tool_params.rs (if tool has parameters)
#[derive(Debug, Deserialize, JsonSchema)]
pub struct MyToolParams {
    #[schemars(description = "Description of the parameter")]
    pub my_param: i32,
}

// In src/tools/<category>.rs
#[tool_router(router = category_router, vis = "pub")]
impl AbletonServer {
    #[tool(description = "Description of what the tool does")]
    pub async fn my_tool(
        &self,
        Parameters(params): Parameters<MyToolParams>,
    ) -> Result<String, Error> {
        // Implementation
        Ok("Success".to_string())
    }
}
```

The macros automatically:
- Generate JSON schemas from the `JsonSchema` derive
- Handle parameter deserialization via `Parameters<T>` wrapper
- Register the tool in the router

### OSC Communication

- **Send only**: `self.osc.send("/live/path", vec![OscType::Int(x)]).await?`
- **Query single value**: `self.osc.query::<T>("/live/path", args).await?`
- **Query multiple values**: `self.osc.query_all("/live/path", args).await?` returns `Vec<OscPacket>`

### Error Handling

Use `crate::error::Error` variants:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [christopherwxyz/remix-mcp](https://github.com/christopherwxyz/remix-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
