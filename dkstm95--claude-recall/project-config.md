---
trigger: always_on
description: Claude Code plugin (v6.3.0) that provides a session awareness statusline.
---

# claude-recall

Claude Code plugin (v6.3.0) that provides a session awareness statusline.
Tracks a Haiku-refined focus label, activity, git status, and prompt count for every parallel Claude Code session.

- **Author**: seungilahn
- **License**: MIT
- **Repository**: dkstm95/claude-recall
- **Install**: `/plugin marketplace add dkstm95/claude-recall`

## Build

```bash
npm run build        # TypeScript -> dist/ (tsc)
npm install          # Install dev dependencies (typescript, @types/node)
```

- Source: `src/*.ts` -> Output: `dist/*.js`
- Target: ES2022, Node16 ESM modules
- Node >= 20.0.0 required

## Architecture

```
.claude-plugin/           # Plugin manifest
  plugin.json             #   Name, version, description, author
  marketplace.json        #   Marketplace listing metadata
commands/                 # Slash commands (markdown with frontmatter)
  setup.md                #   /setup — configure statusline & launcher script
hooks/
  hooks.json              # Hook registration (SessionStart, UserPromptSubmit, PreCompact, SessionEnd)
src/                      # TypeScript source
  config.ts               #   StatuslineConfig interface, theme colors, config file reader, legacy slot mapping
  state.ts                #   SessionState interface, read/write JSON, cleanup, async getGitStatus + refreshGitStatus helper (used by both statusline and hooks)
  format.ts               #   3-line statusline formatter, CJK width, bar renderer, progressive truncation
  statusline.ts           #   Entry point: stdin JSON -> formatStatusline() -> stdout
  stdin.ts                #   Async stdin reader utility
  refine.ts               #   Haiku subprocess wrapper: spawnRefinement + triggerFocusRefinement + launchRefinementWorker (detached) + 5s debounce
  refine-worker.ts        #   Detached worker entry — runs `triggerFocusRefinement` outside the 10s hook window
  rate-limits-cache.ts    #   Per-account cache for rate_limits stdin field (omitted on first render)
  context-window-cache.ts #   Per-session cache for context_window stdin field (omitted on first render)
  hooks/
    session-start.ts      #   Initialize/resume session, cleanup old sessions (>7d)
    prompt-submit.ts      #   Track prompts, update git status, trigger focus refinement at power-of-2 turns
    trigger-refinement.ts #   Shared entry for PreCompact + SessionEnd — spawns the detached refine-worker
dist/                     # Compiled JS (committed, do NOT edit directly)
assets/                   # SVG preview images for marketplace
```

## Data Flow

```
SessionStart event -> session-start.ts        -> creates/updates ~/.claude/claude-recall/sessions/{id}.json
UserPromptSubmit   -> prompt-submit.ts         -> increments promptCount, updates git status, triggers focus refinement at 2^k turns (k>=0, so 1,2,4,8,...)
PreCompact         -> trigger-refinement.ts    -> fire-and-forget the refine-worker (natural milestone)
SessionEnd         -> trigger-refinement.ts    -> fire-and-forget the refine-worker (final snapshot)
Statusline render  -> statusline.ts            -> reads session JSON + stdin metrics + calls getGitStatusLive() directly -> 1-3 line statusline output
```

Focus refinement path:
```
trigger hook -> refine.ts::launchRefinementWorker (spawn detached refine-worker.js, unref, return immediately)
             -> refine-worker.ts -> refine.ts::triggerFocusRefinement (5s debounce via lastRefinedAt)
                -> spawn `claude -p --model=haiku --tools "" --no-session-persistence ...`
                   with env CLAUDE_RECALL_REFINING=1 (prevents recursive plugin hook firing in child)
                -> 45s timeout; output text -> state.focus OR refinementError (empty transcript = silent skip, not an error)
```
Why detached: Claude Code's 10s hook timeout would SIGHUP `claude -p` before Haiku responds (~1-5s typical, up to 45s). The hook returns in <50ms; the worker outlives it and writes state asynchronously.

## Session State Schema

Key fields in `~/.claude/claude-recall/sessions/{sessionId}.json`:

| Field | Type | Description |
|-------|------|-------------|
| sessionId | string | Unique session ID |
| focus | string | AI-refined session description (max 60 chars) |
| branch | string | Current git branch (fallback from gitStatus.branch) |
| gitStatus | GitStatus \| null | `{ branch, dirty, ahead, behind, defaultBranch }` |
| cwd | string | Working directory at session start |
| promptCount | number | Total user prompts (excludes slash commands) |
| lastUserPrompt | string | Last prompt text (first 200 chars) |
| sessionStartedAt | string | ISO timestamp when session was first opened (immutable after SessionStart; drives elapsed fallback) |
| lastActivityAt | string | ISO timestamp of last activity (drives 7-day cleanup) |
| lastRefinedAt | string \| null | ISO timestamp of last focus refinement (debounce guard) |
| refinementError | RefinementError \| null | `{ code: 'timeout' \| 'rate_limit' \| 'auth' \| 'unknown', at, durationMs?, stderrTail? }` |
| lastRefinement | LastRefinement \| null | Last refinement attempt record: `{ at, status: 'ok' \| 'error', code?, durationMs, transcriptBytes, stdoutBytes?, stderrTail? }` (diagnostics, survives across successes) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dkstm95/claude-recall](https://github.com/dkstm95/claude-recall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
