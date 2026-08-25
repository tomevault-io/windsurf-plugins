---
trigger: always_on
description: See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines,
---

# Agentic CI

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines,
PR requirements, and harness/backend parity rules.

agentic-ci runs AI coding agents in sandboxed CI environments with pluggable backends and harnesses. Users run `agentic-ci run "prompt"` to execute an agent in an isolated environment with streaming output and OTEL telemetry.

Three **backends** provide execution environments:
- **Local**: Runs the agent directly in the current environment. No container or sandbox — the agent binary must be on PATH. Useful inside existing CI containers (e.g. Prow step images).
- **Podman** (default): Runs the agent in a Podman container. Simple, widely available.
- **OpenShell**: Runs the agent in an [OpenShell](https://github.com/NVIDIA/OpenShell) sandbox with network policy enforcement and filesystem isolation.

Two **harnesses** define which agent CLI to run:
- **claude-code** (default): [Claude Code](https://docs.anthropic.com/en/docs/claude-code) with `stream-json` output format.
- **opencode**: [OpenCode](https://github.com/anomalyco/opencode) with JSON event output format.

## Architecture

```text
src/agentic_ci/
    cli.py              # Entry point, backend/harness selection, OTEL orchestration
    backend.py          # Backend ABC + shared stream processing
    harness.py          # Harness ABC + ClaudeCode/OpenCode implementations
    backends/
        __init__.py     # Backend factory (create_backend)
        local.py        # LocalBackend — direct execution (no container)
        podman.py       # PodmanBackend — container execution
        openshell/
            __init__.py # OpenShellBackend — sandbox execution
            gateway.py  # OpenShell gateway lifecycle
            sandbox.py  # OpenShell sandbox lifecycle
            policy.py   # Policy resolution + built-in default
    config.py           # Project config loader (.agentic-ci/config.yml)
    plugins.py          # Plugin/skill install (build-time) and filtering (runtime)
    stream.py           # Stream parsers for Claude Code and OpenCode output
    otel.py             # OTLP collector + token/cost summary
```

- **`cli.py`**: Argparse entry point with `setup`, `run`, and `stop` subcommands plus `--backend` and `--harness` flags. Creates harness and backend, handles OTEL lifecycle.

- **`backend.py`**: Abstract `Backend` class with `setup()` and `run()` methods. Shared `_process_stream()` helper reads output from a subprocess through the harness's stream processor. When `output_file` is set on the backend, `_process_stream()` tees every raw output line to disk for transcript capture.

- **`harness.py`**: Abstract `Harness` class encapsulating agent-specific CLI args, env vars, credential paths, and stream parsing. Implementations: `ClaudeCodeHarness`, `OpenCodeHarness`.

- **`plugins.py`**: Build-time plugin installation (`install_claude_plugins`, `install_opencode_skills`) and runtime filtering (`enable_plugins`). At build time, installs skills from the skills-registry marketplace into the container image and writes a plugin-to-skill manifest. At runtime, `AGENT_ENABLED_PLUGINS` controls which plugins are active: Claude Code disables plugins in `settings.json`; OpenCode deletes unwanted skill directories from disk (since `permission.skill.deny` doesn't prevent loading with `--dangerously-skip-permissions`).

- **`backends/podman.py`**: `PodmanBackend` — runs the agent in a `podman run` container. Bind-mounts the workdir into the container at `/workspace`, so changes are visible on the host immediately. Mounts gcloud credentials as read-only volumes. Uses `--network host` when OTEL is enabled.

- **`config.py`**: Loads project configuration from `.agentic-ci/config.yml` in the workdir. Currently supports a `setup` key with a list of commands (bare strings or `{name, run}` objects) that run on the host before sandbox upload, enabling dependency installation for repos whose agents need it.

- **`backends/openshell/`**: `OpenShellBackend` — runs the agent in an OpenShell sandbox. Uploads the workdir into the sandbox on `setup()` and downloads it back after `run()` completes. Only changes inside the workdir are reflected back to the host; files written elsewhere in the sandbox are not retrieved. Manages gateway lifecycle, sandbox creation with network policy, credential injection, and setup steps. Submodules: `gateway.py`, `sandbox.py`, `policy.py`.

- **`stream.py`**: `ClaudeCodeStreamProcessor` parses Claude Code's `stream-json` output. `OpenCodeStreamProcessor` parses OpenCode's JSON event output. Both produce human-readable CI logs with colored ANSI output, tool call summaries, and token display.

- **`otel.py`**: Lightweight OTLP HTTP/JSON receiver (stdlib `http.server`) that logs payloads to JSONL, tracks token usage over a sliding window, and prints a token/cost summary.

### Key

- **Authentication** is auto-detected: if `ANTHROPIC_API_KEY` is set, direct API auth is used (no gcloud credentials needed); otherwise Vertex AI with gcloud ADC files.
- **OTEL collector runs on the host**, not inside the sandbox/container. Currently only Claude Code emits OTEL metrics; OpenCode provides token/cost data via its JSON output.

## Container images


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opendatahub-io/agentic-ci](https://github.com/opendatahub-io/agentic-ci) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
