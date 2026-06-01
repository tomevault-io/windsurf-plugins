---
trigger: always_on
description: This file provides guidance to Claude Code (or any other AI tool -- aider, cursor, continue, etc) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (or any other AI tool -- aider, cursor, continue, etc) when working with code in this repository.

## Project Purpose
The A2A Test Suite is a comprehensive testing framework for the Agent-to-Agent (A2A) protocol, which enables standardized communication between AI agent systems. The project provides validation, property testing, mock server, client implementation, and fuzzing tools to ensure protocol compliance.

## Development Methodology

**IMPORTANT: THIS PROJECT MUST ALWAYS FOLLOW TEST-DRIVEN DEVELOPMENT**

- Write tests before implementing features
- Run `RUSTFLAGS="-A warnings" cargo build` often to verify the feature is building (ALWAYS use `RUSTFLAGS="-A warnings"`)
- Implement features "slowly" and meticulously
- Always verify tests pass before considering work complete
- Ensure builds work with `RUSTFLAGS="-A warnings" cargo test && RUSTFLAGS="-A warnings" cargo build` before submitting changes (ALWAYS use `RUSTFLAGS="-A warnings"`)
- Never skip testing or make untested changes
- Implement features iteratively: small, testable units
- Keep the `start_server_and_test_client.sh` script updated with new features for end-to-end testing

## Documentation
- **README.md**: Overview of A2A protocol and test suite components
- **docs/schema_overview.md**: Detailed A2A protocol schema documentation
- **src/client/README.md**: Comprehensive client feature documentation and examples
- **src/client/tests/integration_test.rs**: Example code demonstrating client features

## Build & Test Commands
- Generate schema types: `cargo run --quiet -- config generate-types`
- Set schema version: `cargo run --quiet -- config set-schema-version [version]`
- Build (ALWAYS use RUSTFLAGS): `RUSTFLAGS="-A warnings" cargo build --quiet`
- Run: `cargo run --quiet -- [subcommand]`
- Test all (ALWAYS use RUSTFLAGS): `RUSTFLAGS="-A warnings" cargo test --quiet`
- Test single (ALWAYS use RUSTFLAGS): `RUSTFLAGS="-A warnings" cargo test --quiet [test_name]`
- Property tests: `cargo run --quiet -- test --cases [number]`
- Validate: `cargo run --quiet -- validate --file [path]`
- Mock server: `cargo run --quiet -- server --port [port]`
- Reference server: `cargo run --quiet -- reference-server --port [port]`
- Fuzzing: `cargo run --quiet -- fuzz --target [target] --time [seconds]`
- Run integration tests: `cargo run --quiet -- run-tests`
- REPL client: `cargo run --quiet -- repl-client --config [config_file]`
- Client commands:
  - Get agent card: `cargo run --quiet -- client get-agent-card --url [url]`
  - Send task: `cargo run --quiet -- client send-task --url [url] --message [text] [--metadata '{"_mock_delay_ms": 2000}'] [--header "header_name"] [--value "auth_value"]`
  - Send task with simulated state machine: `cargo run --quiet -- client send-task --url [url] --message [text] --metadata '{"_mock_duration_ms": 5000, "_mock_require_input": true}'`
  - Send task with file: `cargo run --quiet -- client send-task-with-file --url [url] --message [text] --file-path [path]`
  - Send task with data: `cargo run --quiet -- client send-task-with-data --url [url] --message [text] --data [json]`
  - Get task: `cargo run --quiet -- client get-task --url [url] --id [task_id] [--header "header_name"] [--value "auth_value"]`
  - Get artifacts: `cargo run --quiet -- client get-artifacts --url [url] --id [task_id] --output-dir [dir]`
  - Cancel task: `cargo run --quiet -- client cancel-task --url [url] --id [task_id] [--header "header_name"] [--value "auth_value"]`
  - Validate auth: `cargo run --quiet -- client validate-auth --url [url] --header "header_name" --value "auth_value"`
  - Stream task: `cargo run --quiet -- client stream-task --url [url] --message [text] [--metadata '{"_mock_chunk_delay_ms": 1000}']` 
  - Stream with dynamic content: `cargo run --quiet -- client stream-task --url [url] --message [text] --metadata '{"_mock_stream_text_chunks": 5, "_mock_stream_artifact_types": ["text", "data"]}'`
  - Resubscribe: `cargo run --quiet -- client resubscribe-task --url [url] --id [task_id] [--metadata '{"_mock_stream_final_state": "failed"}']`
  - Set push notification: `cargo run --quiet -- client set-push-notification --url [url] --id [task_id] --webhook [url] --auth-scheme [scheme] --token [token]`
  - Get push notification: `cargo run --quiet -- client get-push-notification --url [url] --id [task_id]`
  - Get state history: `cargo run --quiet -- client get-state-history --url [url] --id [task_id]`
  - Get state metrics: `cargo run --quiet -- client get-state-metrics --url [url] --id [task_id]`
  - Create task batch: `cargo run --quiet -- client create-batch --url [url] --tasks "task 1,task 2,task 3" --name [batch_name]`
  - Get batch: `cargo run --quiet -- client get-batch --url [url] --id [batch_id]`
  - Get batch status: `cargo run --quiet -- client get-batch-status --url [url] --id [batch_id]`
  - Cancel batch: `cargo run --quiet -- client cancel-batch --url [url] --id [batch_id]`
  - List skills: `cargo run --quiet -- client list-skills --url [url] --tags [optional_tags]`
  - Get skill details: `cargo run --quiet -- client get-skill-details --url [url] --id [skill_id]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elliot-at-liminalnook/A2A](https://github.com/elliot-at-liminalnook/A2A) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
