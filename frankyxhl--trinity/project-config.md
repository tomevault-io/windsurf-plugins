---
trigger: always_on
description: Multi-model orchestration skill. Dispatch tasks to any LLM provider via background sub-agents with auto-discovery, config overlay, session management, and health monitoring. Use when the user says /trinity or wants to delegate work to another model.
---


# Trinity — Multi-Model Orchestration

Dispatch tasks to external LLM providers via background sub-agents. Providers are auto-discovered from config + agent files. Sessions stored in `.claude/trinity.json`. Health monitoring via output file heartbeat.

## Startup Check (run once per session before first dispatch)

```bash
# 0. Refuse nested dispatch when Trinity is running as a claude-code provider
if [ "${TRINITY_DISABLE_DISPATCH:-}" = "1" ]; then
  echo "trinity: Nested Trinity dispatch is disabled in this Claude Code process (set by the claude-code provider wrapper)."
  exit 1
fi

# 1. Verify python3 is available
command -v python3 >/dev/null 2>&1 || {
  echo "trinity: python3 not found. Install: brew install python3"
  # abort dispatch
}

# 2. Verify scripts are installed and up to date
SCRIPTS_VERSION=$(python3 ~/.claude/skills/trinity/scripts/session.py --version 2>/dev/null)
REQUIRED_VERSION="3.2.0"
if [ "$SCRIPTS_VERSION" != "$REQUIRED_VERSION" ]; then
  echo "trinity: scripts not installed or outdated (found: ${SCRIPTS_VERSION:-none}, need: $REQUIRED_VERSION)"
  echo "Run: make install (from trinity/ repo) or: cp -r trinity/scripts/. ~/.claude/skills/trinity/scripts/"
  # abort dispatch
fi
```

If scripts pass the version check, proceed normally.

## Syntax

```
/trinity <provider>[:<instance>] "task"          # single dispatch
/trinity <p1>[:<i>] "t1" <p2> "t2"              # multi-provider parallel
/trinity <provider>*N "task"                     # N parallel same-provider
/trinity review "task"                           # preset dispatch
/trinity fast-review "task"                      # preset dispatch
/trinity deep-review "task"                      # preset dispatch
/trinity plan <p1> "t1" <p2> "t2"               # plan with diagram, confirm, execute
/trinity plan "high-level description"           # auto-decompose, confirm, execute
/trinity install <provider>                      # install + register provider
/trinity status                                  # sessions + live activity
/trinity heartbeat [<instance>]                  # on-demand liveness check
/trinity clear [<instance> | all]                # clear sessions
/trinity help                                    # show README
```

## Session Transcript Paths

Resolve a provider's on-disk JSONL transcript path for a stored session pointer (token-efficiency audits, replay debugging).

```
trinity session-path <provider>[:<instance>] [--project <dir>]
```

The command reads `<project>/.claude/trinity.json`, looks up the pointer for the given key, dispatches to a per-provider resolver, and prints the absolute JSONL path on stdout. **Path-only** — transcript content is never read or surfaced. The `:default` suffix is stripped at lookup (`glm:default` is equivalent to `glm`); other suffixes (e.g. `glm:experimental`) pass through verbatim.

| Provider | Transcript root |
|----------|-----------------|
| `glm` | `~/.factory/sessions/<encoded-project-path>/<session-id>.jsonl` |
| `codex` | `~/.codex/sessions/<YYYY>/<MM>/<DD>/...<session-id>.jsonl` (index-first via `~/.codex/session_index.jsonl`; broad-glob fallback) |
| `claude-code` | `~/.claude-trinity-claude-code/projects/<PROJECT_SLUG>/<session-id>.jsonl` |
| `deepseek` | `~/.claude-deepseek/projects/<PROJECT_SLUG>/<session-id>.jsonl` |
| `openrouter` | `~/.claude-openrouter/projects/<PROJECT_SLUG>/<session-id>.jsonl` |
| `gemini` | not yet supported (exits 3 with explicit deferral message) |

Path encoding: absolute project path with `/` replaced by `-`. The `glm` resolver KEEPS the leading dash (matches `~/.factory/sessions/-Users-frank-...`); the claude-CLI wrappers (`claude-code` / `deepseek` / `openrouter`) STRIP the leading dash (matches `PROJECT_SLUG=$(echo "$PROJECT_DIR" | sed 's|/|-|g; s|^-||')` per `providers/<name>.md`).

The claude-CLI wrappers themselves are installed at `~/.claude/skills/trinity/bin/<name>` (e.g. `~/.claude/skills/trinity/bin/claude-code`, `~/.claude/skills/trinity/bin/deepseek`, `~/.claude/skills/trinity/bin/openrouter`) — these are the CLI binaries. Their per-wrapper transcript stores listed above are *separate* from the wrapper bin directory.

Exit codes:

- `0` — path printed; file exists on disk.
- `2` — no session pointer for the provided key.
- `3` — provider unsupported, transcript file missing, malformed `session_id`, or codex multi-glob residual (>1 match).

## Provider Discovery

On every dispatch, resolve available providers and presets:
1. Load `~/.claude/trinity.json` → `providers`, `presets`, and `preset_aliases` (global)
2. Load `.claude/trinity.json` → project overlay; project entries win on conflict
3. Verify each provider has a matching agent file: `~/.claude/agents/trinity-<name>.md` or `.claude/agents/trinity-<name>.md`

A provider is **usable** only when it has both a config entry AND an agent file.
- Config entry but no agent file → "⚠️ unregistered (missing agent file)" in `/trinity status`; dispatch blocked
- Agent file but no config entry → "⚠️ unregistered (missing config)" in `/trinity status`; dispatch blocked
- Neither → not listed


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frankyxhl/trinity](https://github.com/frankyxhl/trinity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
