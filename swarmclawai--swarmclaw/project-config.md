---
trigger: always_on
description: - **`CLAUDE.md` and `AGENTS.md` must stay in sync.** When you add, edit, or remove a rule in one file, apply the same change to the other. They contain the same project guidelines — one for Claude Code, one for other coding agents.
---

# CLAUDE.md

### Keeping Instructions in Sync
- **`CLAUDE.md` and `AGENTS.md` must stay in sync.** When you add, edit, or remove a rule in one file, apply the same change to the other. They contain the same project guidelines — one for Claude Code, one for other coding agents.

### Dev Server
- **NEVER kill the dev server (`npm run dev`, port 3456) without asking the user first.** The user is often actively working against it, and other agents may be running tests against it. Always confirm before stopping, restarting, or killing the dev server process.

### Code Quality

**Lint baseline is a release gate.** Run `npm run lint:baseline` before any production release. The baseline must pass — no net-new lint fingerprints. If you fix existing violations, run `npm run lint:baseline:update` to lock in the improvement.

**Embrace TypeScript and the type system.** Use proper types, interfaces, and generics. Never use `any` — use `unknown`, `Record<string, unknown>`, or define a proper interface. The type system is there to catch bugs at compile time; circumventing it defeats the purpose.

**Lint rules exist to protect us.** Do not suppress, disable, or work around lint rules. Fix the underlying code instead. If a rule is genuinely wrong for the project, change the rule in the lint config with a clear justification — but this should be rare. The default is to fix the code, not silence the linter.

### Architecture

**Prefer simple, maintainable, reliable architectures.** Choose the straightforward approach over the clever one. Code that is easy to read, easy to debug, and easy to delete is better than code that is abstract, configurable, or "elegant." Avoid premature abstraction — three similar lines are better than a premature helper. Build for the current requirement, not hypothetical future ones.

### Commit Messages
- Never reference "Claude", "Anthropic", "Codex", "Co-Authored-By", or any AI tool in commit messages. Write commit messages as if a human authored the code.

### Testing

**Always test with live agents.** After making changes to chat execution, streaming, plugins, connectors, or any agent-facing code path, verify the work by running a live agent chat on the platform. Unit tests and type checks are necessary but not sufficient — the real test is whether an agent can actually hold a conversation, use its plugins, and produce correct results through the running application.

**Lock in working behavior with tests.** When a feature or fix is confirmed working — whether by user verification, live agent testing, or manual QA — add regression tests (frontend and/or backend as appropriate) to prevent it from breaking later. The goal is to ratchet forward: once something works, it stays working. Don't skip this step just because the fix was small or the feature seems simple. A quick unit test today saves a painful debugging session tomorrow.

### UX Philosophy

SwarmClaw serves non-technical users alongside power users. Every UI surface should follow these principles:

**Progressive disclosure.** Hide power-user controls behind expandable sections — don't dump every option on screen at once. Use `AdvancedSettingsSection` (`src/components/shared/advanced-settings-section.tsx`) for collapsible expert panels (routing config, runtime behavior, overrides). Default state is collapsed.

**Smart defaults — never leave blanks.** Every field that can have a sensible default should have one. `setup-defaults.ts` (`src/lib/setup-defaults.ts`) is the single source of truth for provider defaults, starter agent kits, `keyUrl`/`keyLabel` pairs, and default model selections. When adding a new provider or agent preset, add its defaults there — don't scatter magic values across components. `randomSoul()` (`src/lib/soul-suggestions.ts`) provides personality suggestions so the soul field is never empty.

**Contextual help.** Use `HintTip` (`src/components/shared/hint-tip.tsx`) — the `?` tooltip component — next to any field that isn't self-explanatory. For connector config fields, add entries to `FIELD_HINTS` in `connector-sheet.tsx`. Multi-step setup guides (Discord developer portal, Slack app creation, Telegram BotFather, etc.) include exact URLs so users don't have to hunt for them.

**Never leave the user stuck.** If a form requires an API key, link directly to where they get one (`keyUrl` in `setup-defaults.ts`). If a connector needs setup steps, show them inline with clickable links. Error states should say what went wrong and what to do next.

### Zustand Store Updates

**Always use `setIfChanged` for async loaders.** Every async loader that fetches data from the API and writes it to the store must use `setIfChanged` (`src/stores/set-if-changed.ts`) instead of raw `set()`. The API always returns fresh object references, so raw `set()` triggers re-renders in every subscribed component even when the data hasn't changed. `setIfChanged` keeps a JSON fingerprint and skips the write if nothing changed.

```ts
// Wrong — causes render cascade on every poll
set({ agents: freshAgents })

// Right — only triggers re-renders when data actually changed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swarmclawai/swarmclaw](https://github.com/swarmclawai/swarmclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
