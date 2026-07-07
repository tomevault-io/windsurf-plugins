---
trigger: always_on
description: `acrawl` is a native-Rust LLM-driven web crawler. A user provides a natural-language goal; the agent plans, navigates, and extracts structured data via a 42-tool toolbox (31 browser + 4 agent-control + 7 script). It ships as a single binary with three modes: an interactive Ratatui TUI REPL (requires a TTY), non-interactive `prompt` (one-shot) / `--resume` (slash-command replay), and `mcp` (built-in MCP server over stdio).
---

# AGENTS.md

## Project

`acrawl` is a native-Rust LLM-driven web crawler. A user provides a natural-language goal; the agent plans, navigates, and extracts structured data via a 42-tool toolbox (31 browser + 4 agent-control + 7 script). It ships as a single binary with three modes: an interactive Ratatui TUI REPL (requires a TTY), non-interactive `prompt` (one-shot) / `--resume` (slash-command replay), and `mcp` (built-in MCP server over stdio).

## Commands

```bash
cargo build --release                                        # produce ./target/release/acrawl
cargo test --workspace                                       # run full test suite (~1,100 tests)
cargo test -p <crate> <test_name>                            # run a single test (e.g. -p agent mvp_tool_specs_contains_expected_42_tools)
cargo clippy --workspace --all-targets -- -D warnings        # lints must be clean (workspace lints set pedantic = warn)
cargo fmt --check                                            # format check

./target/release/acrawl                                      # launch REPL
./target/release/acrawl prompt "scrape all titles from example.com"   # one-shot
./target/release/acrawl mcp                                  # launch MCP server (stdio)
./target/release/acrawl mcp install                          # interactive IDE installer
./target/release/acrawl --resume session.json /status /compact        # non-interactive session maintenance

# Non-interactive provider setup (agent/CI use)
acrawl auth anthropic --api-key "sk-ant-..."      # configure credentials
acrawl auth openai --api-key "sk-..."             # other providers same pattern
acrawl auth amazon-bedrock --access-key AKIA... --secret-key ... --region us-east-1
acrawl config set model anthropic/claude-sonnet-4-6  # set default model
acrawl auth status --check anthropic              # gate: exit 0 if ready, 3 if not
acrawl auth status                                # show all configured providers
acrawl auth list                                  # list all available providers
acrawl config get headless                        # read a setting
acrawl config set headless false                  # write a setting
acrawl mcp install --client opencode             # install MCP for one IDE
acrawl mcp install --all --yes                   # install for all IDEs
# Exit codes: 0=ok  1=error  2=usage/config  3=not-configured
```

The CLI reads LLM credentials from `~/.acrawl/credentials.json` (managed by `acrawl auth`) and runtime settings from `~/.acrawl/settings.json`. Both paths respect the `ACRAWL_CONFIG_HOME` env var override. Run `acrawl auth [anthropic|openai|other]` to configure a provider.

## Workspace layout

Eleven crates under `crates/`, compiled with `resolver = "2"`:

- **core** (`acrawl-core`) — shared types, traits, and error hierarchy used across the workspace. Defines `ToolSpec`, `ToolEffect`, `AssistantEvent`, `RuntimeObserver`, `ContentBlock`/`ConversationMessage`/`MessageRole`/`TokenUsage`, `ToolOutcome`, `ApiClient`/`ApiRequest`, `config_home_dir`, and `OAuthConfig`.
- **api** — HTTP + SSE clients for Anthropic (`client.rs`), OpenAI-compatible (`openai.rs`), and Codex OAuth (`codex.rs`). `sse.rs` is the shared streaming frame parser; `types.rs` holds the Anthropic message schema. `oauth.rs` contains OAuth PKCE helpers, credential persistence, and token exchange types. `provider/registry.rs` and `provider/factory.rs` handle provider discovery and client construction.
- **browser** — browser automation layer. `PlaywrightBridge` (CloakBrowser headless Chromium), `ExtensionBridge` (Chrome extension backend via CDP), `FetchRouter` (HTTP→browser escalation), `BrowserContext` (tab/URL state), and `WsBridgeServer` (WebSocket server for extension communication). `browser_backend.rs` defines the `BrowserBackend` trait that both bridges implement.
- **agent** — agent orchestration and the 42-tool toolbox (31 browser + 4 agent-control + 7 script). `agent.rs` drives the agent loop; `tools/` contains individual tool handlers; `manager.rs` manages sub-agent fork/join lifecycle; `prompt.rs` builds the system prompt; `state.rs` holds `CrawlState`; `url_claim.rs` coordinates URL claims across agents.
- **runtime** — `ConversationRuntime` (the core turn loop), `Session` persistence, system-prompt builder, compaction, usage/pricing, `config/` subdirectory (loader, MCP config, features), and a full MCP client stack in `mcp/` (`client.rs`, `types.rs`, `server_manager.rs`, `process.rs`, `naming.rs`).
- **render** — markdown/terminal rendering (`markdown.rs`), tool call output formatting (`tool_format.rs`), output format selection (`format.rs`), and the `OutputSink` trait + implementations (`sink.rs`) that bridge runtime events to the UI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mingye-Lu/AgenticCrawler](https://github.com/Mingye-Lu/AgenticCrawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
