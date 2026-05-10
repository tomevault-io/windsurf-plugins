---
trigger: always_on
description: This document describes workflows for autonomous agents working on **prokube.ai OpenCode UI** - a standalone Web UI for OpenCode that runs in Kubeflow Notebooks.
---

# Agent Workflow

This document describes workflows for autonomous agents working on **prokube.ai OpenCode UI** - a standalone Web UI for OpenCode that runs in Kubeflow Notebooks.

- **Worker Agent**: Executes individual tasks
- **Supervisor Agent**: Coordinates multiple workers, handles escalations
- **Triage Agent**: Captures requests, writes self-contained issues, breaks down large work

---

# Project Context

## Tech Stack

- **Frontend**: SolidJS with TypeScript, Tailwind CSS
- **Backend**: OpenCode API server (separate process, not part of this repo)
- **Build**: Bun, esbuild
- **Deployment**: Docker image for Kubeflow Notebooks
- **Process Supervision**: s6-overlay

## Repository Structure

```
/
├── app-prefixable/     # SolidJS frontend
│   ├── src/
│   │   ├── components/ # UI components
│   │   ├── context/    # State management (SDK, MCP, etc.)
│   │   ├── pages/      # Page components
│   │   ├── sdk/        # OpenCode SDK (local copy)
│   │   └── utils/      # Utilities
│   ├── dev.ts          # Dev server
│   └── build.ts        # Production build
│
├── docker/             # Kubeflow notebook image
│   ├── Dockerfile
│   ├── serve-ui.ts     # Production server
│   └── s6/             # s6-overlay config
│
├── shared/             # Shared code between servers
│   └── prokube-endpoints.ts
│
├── .github/            # CI/CD workflows
├── AGENTS.md           # This file
└── README.md           # Project overview
```

## Key Resources

- `AGENTS.md` - This file (workflows and conventions)
- `docker/` - Docker image for Kubeflow deployment
- `app-prefixable/` - Custom SolidJS frontend
- `shared/` - Shared code between dev and production servers

## Reference Resources

When unsure how to implement a feature, consult the upstream OpenCode project:

- **Upstream Repo**: https://github.com/anomalyco/opencode
- Clone locally if needed: `git clone https://github.com/anomalyco/opencode /tmp/opencode-ref`
- Look for similar patterns, but adapt solutions to our prefix-aware architecture
- Do not copy code verbatim - understand and reimplement as needed

---

# Repository Rules

## Remote

| Remote   | Repository                      | Agent Access     |
| -------- | ------------------------------- | ---------------- |
| `origin` | `prokube/pk-opencode` (private) | **Push allowed** |

## Branches

| Branch        | Purpose                           |
| ------------- | --------------------------------- |
| `main`        | Main development branch (default) |
| `decouple-ui` | Current feature branch            |

---

# Conventions

## Branding

- The product name is **prokube.ai** (always lowercase, with ".ai" suffix)
- Never write "ProKube", "Prokube", or "prokube" without the ".ai" suffix in user-facing text

## HTTP Base Path Configuration

**CRITICAL**: Never hardcode paths in the frontend code!

```typescript
// CORRECT - Use prefix() from base-path context
import { useBasePath } from "../context/base-path";
const { prefix } = useBasePath();
const url = prefix("/api/session");

// CORRECT - Use serverUrl from path utils for SDK
import { serverUrl } from "../utils/path";
const client = createClient({ serverUrl });

// WRONG - Hardcoded path
fetch("/api/session");

// WRONG - Hardcoded prefix
fetch("/notebook/ns/name/api/session");
```

## Code Style

- Keep things in one function unless composable or reusable
- Avoid unnecessary destructuring. Use `obj.a` instead of `const { a } = obj`
- Avoid `try`/`catch` where possible
- Avoid using the `any` type
- Prefer single word variable names where possible
- Use Bun APIs when possible, like `Bun.file()`
- Rely on type inference; avoid explicit type annotations unless necessary

### Avoid let statements

```typescript
// Good
const foo = condition ? 1 : 2;

// Bad
let foo;
if (condition) foo = 1;
else foo = 2;
```

### Avoid else statements

```typescript
// Good
function foo() {
  if (condition) return 1;
  return 2;
}

// Bad
function foo() {
  if (condition) return 1;
  else return 2;
}
```

## Git Workflow

### Commits

```bash
# Stage specific files only (NEVER use git add -A or git add .)
git add app-prefixable/src/pages/session.tsx

# Commit with descriptive message
git commit -m "fix: description of what changed"
```

### Commit Message Format

- `feat:` New feature
- `fix:` Bug fix
- `refactor:` Code restructure
- `docs:` Documentation
- `chore:` Maintenance

### When to Push

**Do NOT push automatically after every commit.** Pushing triggers CI builds.

- Push only when the user explicitly requests it, or when a feature is complete
- For local development, commit locally but wait for user approval before pushing
- If unsure, ask the user: "Should I push these changes now?"

## Required GitHub CLI Extensions

The workflow requires the `gh-copilot-review` extension for requesting Copilot code reviews on PRs.

**Check if installed:**

```bash
gh extension list | grep copilot-review
```

**Install if missing:**

```bash
gh extension install ChrisCarini/gh-copilot-review
```

**Security Note:** Installing GitHub CLI extensions from third parties carries supply chain risk. Before installing, verify:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prokube/pk-opencode-webui](https://github.com/prokube/pk-opencode-webui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
