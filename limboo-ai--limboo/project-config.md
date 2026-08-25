---
trigger: always_on
description: Operational guide and deep context for any AI coding agent (Claude, etc.) working
---

# CLAUDE.md

Operational guide and deep context for any AI coding agent (Claude, etc.) working
in this repository. Read this first. It explains **what Limboo is**, **how the
code is organized**, **the rules you must follow**, and **what is and is not built
yet**.

> Companion document: [`project.md`](project.md) holds the full product/architecture
> vision. `CLAUDE.md` (this file) is the practical, code-level contract for working
> in the repo. When the two disagree about *current reality*, trust `CLAUDE.md`.

---

## 1. What is Limboo?

Limboo is a **local-first desktop application** that acts as the *operating system
for AI software development*. It is **not an AI model**. Instead, it provides the
environment around a connected coding agent: project management, sessions, file
watching, repository indexing, git operations, terminal execution, memory,
permissions, context, search, and UI.

Core idea: **every development task happens inside a Session**. A session bundles a
repository, branch, chat history, agent, terminal history, checkpoints,
permissions, context, memory, tasks, and generated files into one workspace.

Guiding principles (from `project.md` §4): Fast, Local, Private, Modular, Secure,
Responsive, Observable, Predictable, Recoverable. There is **no backend**. Limboo
itself makes exactly **three** kinds of outbound request, and no others may be
added without amending this paragraph:

1. The connected coding agent talking to its AI provider.
2. **Contributor avatars** — so commit history can show a real face. Two steps,
   both gated by the single `settings.git.avatars.enabled` switch:
   - **Identity.** A GitHub *noreply* commit address already encodes the account
     (`<id>+<login>@users.noreply.github.com`), so it needs no lookup. Every
     other address — which is most real history — can only be resolved by
     GitHub, so `GhManager.commitAuthors` calls **one** read-only endpoint,
     `GET /repos/{owner}/{repo}/commits`, through the `gh` CLI. It returns
     `commit.author.email` alongside the resolved `author.{login,avatar_url}`,
     mapping a whole page of history per request. **This tells GitHub which
     repository is being browsed** — the reason the setting exists and says so.
   - **Image.** `main/managers/gh/avatars.ts` fetches the picture from GitHub's
     avatar host: host-allowlisted, https-only, manual redirect screening, byte-
     and time-capped, magic-byte sniffed (no SVG), and screened by
     `isEmbeddedAvatar` before it can reach an `<img src>`.

   `gh api` is reachable **only** from `commitAuthors`, with a fixed endpoint
   built from a remote Limboo parsed itself. It has no IPC channel and no agent
   tool — it can POST, which is also why it stays out of the agent's read-only
   allowlist. Authentication remains the CLI's; Limboo still reads and stores no
   token. `git`, `gh`, and the update checker are separate processes/subsystems
   with their own rules.
3. **Agent-harness setup** — the npm registry, once per harness, to install the
   agent CLI the AI SDK harness path runs. **Consent-gated and off by default.**

   The harness path (`settings.agent.harness`, off behind `legacyClaudeSdk`)
   drives a third-party adapter whose first session bootstraps its own runtime:
   `@ai-sdk/harness-claude-code` writes a `package.json` + lockfile into
   `.harness-bootstrap/` — a **sibling of the worktree, never inside the
   repository** — and runs `pnpm install --frozen-lockfile` plus the CLI's own
   installer there. The adapter hardcodes this; there is no offline mode.

   Four things keep it inside this paragraph's spirit rather than merely
   permitted by it:
   - **The user approves the verbatim commands, once.** They are read from the
     adapter itself (`getBootstrap()`), never hardcoded in the consent surface,
     and the approval is keyed to a **hash of those exact commands**
     (`agent.harness.bootstrapAck`), so an adapter upgrade that changes what
     runs asks again. Same posture as the `limboo.json` ack-hash gate. No ack,
     no run — `assertBootstrapConsent` refuses.
   - **It is refused, with a reason, when it cannot succeed.** A sandbox network
     policy of `off` (or an allowlist without the registry), or a missing
     `pnpm`, is detected before the run instead of surfacing as a bootstrap that
     times out inside the sandbox (`assertBootstrapPossible`).
   - **It reaches nothing but the registry**, and it installs into the sandbox
     state dir, which is the ONLY path outside the worktree the local sandbox
     provider permits (two literal dot-prefixed segments; see
     `LocalWorktreeSandbox.resolvePath`).
   - **It is not the agent's network.** The agent's own provider traffic is item
     1; this is a package install, and no agent tool can trigger it.

   If a future harness needs a different network reach, it does **not** inherit
   this item — amend this paragraph again.

---

## 2. Tech stack (current)

| Layer            | Choice                                      |
| ---------------- | ------------------------------------------- |
| Shell / desktop  | **Electron 42** (via **Electron Forge 7**)  |
| Bundler          | **Vite 5** (`@electron-forge/plugin-vite`)  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [limboo-ai/limboo](https://github.com/limboo-ai/limboo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
