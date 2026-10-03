---
trigger: always_on
description: OpenAgent is a Tauri 2 desktop app. SvelteKit owns presentation and interaction
---

# OpenAgent contributor map
OpenAgent is a Tauri 2 desktop app. SvelteKit owns presentation and interaction
state, the private `sdk` submodule owns runtime behavior and typed transports,
and `src-tauri` is a thin desktop host. The project is in active debugging; do
not add compatibility paths unless the task explicitly requires them.

Keep this file at or below 150 lines. It is a routing map and repository
boundary, not a complete development manual. Put product behavior, architecture,
repeatable procedures, and fragile subsystem invariants in focused workspace
skills; keep private SDK internals in the SDK repository.

## Route the task before editing
Read every applicable owner before changing files:

| Intent | Starting skill |
| --- | --- |
| Understand repository structure and ownership | `.agents/skills/project-orientation/SKILL.md` |
| Change prompt assembly, skill routing, or agent documentation validation | `.agents/skills/agent-prompt-infrastructure/SKILL.md` |
| Implement product behavior or subsystem changes | The matching `openagent-*` owner below |
| Debug or inspect the real native desktop app | `.agents/skills/native-app-debugging/SKILL.md` |
| Verify browser-visible behavior | `.agents/skills/playwright/SKILL.md` |
| Deliver repository changes | `.agents/skills/deliver-via-pr/SKILL.md` |

The `openagent-*` names below are implementation owners, not a single
catch-all category. The starting skills above separate orientation, coding,
debugging, verification, and delivery so agents can choose the right scope.

| Scope | Source of truth |
| --- | --- |
| Repository delivery, documentation ownership, commits, worktrees, PRs, CI handoff | `.agents/skills/deliver-via-pr/SKILL.md` |
| Chat transcript, composer, streaming/final reconciliation, restore, attachments, chat events, streamed rendering | `.agents/skills/openagent-chat-frontend/SKILL.md` |
| Tauri host, native windows, single instance, IPC adapters, desktop verification | `.agents/skills/openagent-desktop-host/SKILL.md` |
| Configuration, databases, memory, migrations, destructive data transitions | `.agents/skills/openagent-persistence/SKILL.md` |
| Workflows, releases, CI classification, helper packaging, bundle qualification | `.agents/skills/openagent-release-engineering/SKILL.md` |
| Browser-reproducible UI verification | `.agents/skills/playwright/SKILL.md` |
| Product design and component language, including `DESIGN.md` | `.agents/skills/openagent-design-system/` |
| Messaging channels and remote gateway | `.agents/skills/openagent-channel-integrations/` |
| Agent Plugin package behavior | `.agents/skills/openagent-plugin-development/` |
| Public Harness client and server protocol | `.agents/skills/openagent-harness-sdk/` |
| Modular frontend, Runtime, and shell updates | `.agents/skills/openagent-update-delivery/` |
| Embedding resource provenance and activation | `.agents/skills/openagent-embedding-resources/` |
| Windows development environment | `.agents/skills/openagent-windows-development/` |
| Private SDK changes | `sdk/AGENTS.md` and every skill it requires |

Treat implementation and agent-facing documentation as one change. Identify
the affected behavior or invariant and its primary owner before editing; do not
duplicate the same detailed rule across this file, a skill, and README files.

## Commands and verification
Use Bun for JavaScript dependencies and scripts. Prefix shell commands with
`rtk` as required by the global instructions.

```bash
bun run dev                              # Vite on an available port
bun run build
bun run check                            # Svelte + TypeScript
bun run check:tests                      # Type-check Bun tests
bun run check:skills                     # Validate skill metadata and routing
bun run test:browser:preview             # Browser smoke test for the streaming preview
bun run preflight                        # Diff-selected fast checks
bun run prepare:windows-sandbox:dev      # Pinned Windows helpers
bun tauri dev                            # Tauri with the selected Vite URL
bun run tauri:build
cd src-tauri && cargo check
```

Before committing, inspect the complete diff, stage intended new files, and run
`bun run preflight`. It evaluates the branch, index, worktree, and untracked
filenames against `origin/master`. Use `--dry-run` to inspect its plan and
`--base <ref>` only when the target is not `master`.

Do not manually duplicate CI lint, test, check, or build commands. Artifact
validation, implementation-time interactive checks, and checks explicitly
requested by the user remain allowed. Do not edit generated `build/`,
`.svelte-kit/`, or `target/` output.

Debug Tauri runs use `~/.openagent-dev`. Set `OPENAGENT_HOME` only for a
task-specific fixture; routine development must not touch installed release
state under `~/.openagent`.

## Repository boundaries
- `sdk/` is a pinned private submodule. Commit and push SDK changes there first,
  then commit only its gitlink and required host/frontend integration here.
- `src-tauri/` contains thin entry points, Tauri plugins and commands, event
  adapters, desktop capabilities, build configuration, and packaging metadata.
  Runtime state machines and transport ownership stay in the SDK.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BANG404/openagent](https://github.com/BANG404/openagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
