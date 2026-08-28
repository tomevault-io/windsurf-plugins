---
trigger: always_on
description: pnpm install          # Install all dependencies
---

# AGENTS.md — Stratos

## Quick Start

```bash
pnpm install          # Install all dependencies
pnpm build            # Build all 3 packages (core, ui, desktop)
pnpm test             # Run tests across all packages
pnpm --filter @stratosapp/desktop dev         # Dev mode with HMR
pnpm --filter @stratosapp/desktop dev:debug   # Dev mode + CDP (port auto-derived from git root)
```

## Testing

```bash
pnpm test                                          # Run all tests (core + ui + desktop)
pnpm --filter @stratosapp/core test                # Core unit + integration tests only
pnpm --filter @stratosapp/ui test                  # UI component tests only
pnpm --filter @stratosapp/desktop test             # Desktop unit + integration tests only
```

**CI runs automatically** on every push via GitHub Actions (`.github/workflows/ci.yml`):

- `lint` — ESLint across all packages
- `typecheck` — `tsc --noEmit` after a full build
- `test` — Vitest across all packages

**Pre-commit hook** (Husky + lint-staged) runs `prettier --write` on staged `*.{ts,tsx,json,md}` files before every commit.

## Mandatory: Visually Verify Every UI Change

**You MUST test every UI change using Chrome DevTools MCP before considering it done.** Do not ask the user to verify — verify it yourself.

```bash
# 1. Start the app with CDP enabled (if not already running)
pnpm --filter @stratosapp/desktop dev:debug

# 2. Snapshot the UI to get element UIDs
#    REQUIRED before any click/fill/press_key — UIDs are session-scoped
take_snapshot

# 3. Interact with the feature you changed
click uid=...
fill uid=... value="..."
press_key key="Enter"

# 4. Screenshot + snapshot to verify the result
take_screenshot filePath="/tmp/verify.png"
take_snapshot
```

**Rules:**

- Every `take_snapshot` starts a new session. You MUST call it before any interaction.
- After interacting, WAIT 2-5 seconds, then screenshot + snapshot to capture the updated state.
- READ the screenshot image to confirm visual correctness. READ the snapshot text to confirm content/structure.
- If something looks wrong, fix it and re-verify. Do not move on with broken UI.
- **Typing into React inputs:** `fill` sets the DOM value but does NOT trigger React state updates. Use `evaluate_script` with native setter + `input` event:
  ```js
  (el) => {
    const setter = Object.getOwnPropertyDescriptor(
      window.HTMLTextAreaElement.prototype,
      "value",
    ).set;
    setter.call(el, "your text here");
    el.dispatchEvent(new Event("input", { bubbles: true }));
  };
  ```
- **InputBar is a `contentEditable` div (not a textarea):** Set `el.textContent` directly then dispatch `new Event('input', { bubbles: true })`.

## CDP Configuration

- **Port:** Always auto-derived from the git root path using SHA256: `PORT = 9200 + (first 4 hex chars of hash % 799)`. Both the Electron app and `scripts/cdp-mcp.sh` use the same derivation, so they always agree. Override with `CDP_PORT` env var if needed.
- **How it works:** `dev:debug` sets `ENABLE_CDP=1` which makes Electron open `--remote-debugging-port` on the derived port. The `.mcp.json` MCP config runs `scripts/cdp-mcp.sh` which derives the same port and connects `chrome-devtools-mcp` to it.
- **Check your port:** The Electron console logs `[worktree] CDP port=XXXX` on startup, or run: `ROOT=$(git rev-parse --show-toplevel) && HASH=$(echo -n "$ROOT" | shasum -a 256 | cut -c1-4) && echo $((9200 + 16#$HASH % 799))`
- **First-time setup:** When Codex prompts to enable the `chrome-devtools` MCP server, accept it. To pre-allow all CDP tools, create `.Codex/settings.local.json`:
  ```json
  {
    "permissions": {
      "allow": [
        "mcp__chrome-devtools__list_pages",
        "mcp__chrome-devtools__take_snapshot",
        "mcp__chrome-devtools__click",
        "mcp__chrome-devtools__fill",
        "mcp__chrome-devtools__press_key",
        "mcp__chrome-devtools__take_screenshot",
        "mcp__chrome-devtools__evaluate_script"
      ]
    },
    "enableAllProjectMcpServers": true
  }
  ```
- **Troubleshooting:** If MCP can't connect, ensure the app is running with `dev:debug` and that the Electron process and MCP script resolve the same git root (check with `git rev-parse --show-toplevel`).

## Project Structure

Monorepo with 3 packages managed by pnpm workspaces + turborepo:

| Package               | Path               | Description                                                         |
| --------------------- | ------------------ | ------------------------------------------------------------------- |
| `@stratosapp/core`    | `packages/core`    | Provider abstraction, storage adapters, trace store, worktree utils |
| `@stratosapp/ui`      | `packages/ui`      | React components, bridge system, hooks (zero Electron dependency)   |
| `@stratosapp/desktop` | `packages/desktop` | Electron shell, IPC bridge (60+ channels), preload, renderer        |

## Key Files

| Area            | File                                             |
| --------------- | ------------------------------------------------ |
| Electron main   | `packages/desktop/src/main/index.ts`             |
| Agent manager   | `packages/desktop/src/main/agent-manager.ts`     |
| IPC channels    | `packages/desktop/src/common/ipc-channels.ts`    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ContextSphere/stratos](https://github.com/ContextSphere/stratos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
