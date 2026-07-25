---
trigger: always_on
description: TypeScript codebase. Use TypeScript for all new code unless explicitly told otherwise.
---

# Claude Instructions

## Language

TypeScript codebase. Use TypeScript for all new code unless explicitly told otherwise.

## Build & Workspaces

```bash
yarn install && yarn start   # Dev mode
yarn build                   # Rollup compile to app/
yarn lint && yarn test       # Lint + test
yarn workspaces:build        # Build all workspaces
```

**NEVER** run `yarn build` directly in workspace directories — creates incorrect output structures. Always use root commands.

After building `desktop-release-action`, remove the nested dist: `rm -rf workspaces/desktop-release-action/dist/dist` — the action only needs `workspaces/desktop-release-action/dist/index.js`.

## Patching npm Packages (CRITICAL)

Two different patching mechanisms — do NOT confuse them:

| Mechanism | Location | Used For |
|-----------|----------|----------|
| **Yarn patch protocol** | `.yarn/patches/` | `@ewsjs/xhr` (configured in package.json) |
| **patch-package** | `patches/` | `@kayahr/jest-electron-runner` |

**NEVER add `@ewsjs/xhr` patches to `patches/`** — causes CI failures due to conflicts.

## Windows Builds

Always include all architectures: x64, ia32, arm64.

Code signing uses Google Cloud KMS in two phases:
1. Build packages without signing (empty env vars)
2. Sign built packages using jsign with Google Cloud KMS

This prevents MSI build failures from KMS CNG provider conflicts.

## UI — Fuselage Design System

**MANDATORY: Use Fuselage components** for all UI work. Only create custom components when Fuselage doesn't provide what's needed.

- Import from `@rocket.chat/fuselage`
- Check `Theme.d.ts` for valid color tokens
- Reference: [Fuselage Storybook](https://rocketchat.github.io/fuselage) and [Rocket.Chat main repo](https://github.com/RocketChat/Rocket.Chat) for usage patterns

## Testing

- `*.spec.ts` — Renderer process tests
- `*.main.spec.ts` — Main process tests
- Renderer specs must live in a Jest-matched nested path, e.g. `src/<module>/<subdir>/*.spec.ts(x)` or `src/<module>/renderer.spec.ts(x)`. Flat `src/<module>/*.spec.ts` files are not discovered by current `testMatch`; verify new specs with `yarn test --listTests --runTestsByPath <file>`.
- Uses `@kayahr/jest-electron-runner` for Electron environment simulation
- Tests run on Windows, macOS, AND Linux CI — always verify cross-platform
- **Screen-capture / WebRTC / portal behavior CANNOT be validated in software-rendered VMs** — Chromium gates the PipeWire capture path on hardware GL (gate moves between Electron versions). Validate on hardware GL (GPU passthrough or physical machine) and prefer dbus-level assertions (`org.freedesktop.portal.ScreenCast` requests) over dialog visibility, which is portal/boot-state flaky. Full story: `docs/postmortem-screen-picker-startup-enumeration.md`

## QA Flow Authoring

When creating or updating QA flows under `qa/`, read `qa/README.md`,
`qa/AGENTS.md`, and `qa/flow-template.md` first. QA steps must be
self-contained and visually findable for a tester or visual agent that knows
nothing about the feature.

- For Desktop PR, branch, or release-candidate QA passes, use
  `skills/desktop-qa-flows/SKILL.md` as the workflow entrypoint. The skill
  decides whether to update existing flows, add new flows, or create a new
  `qa/<feature-slug>/` pack based on changed user-visible risk. It is plain
  Markdown and can be used by any agent, including Codex, Claude, Hermes, Cursor,
  and GitHub agents, when explicitly pointed to it.
- Derive tester-facing steps from the implementation, not product intuition.
  Inspect changed React components, Fuselage icons, i18n labels, menu
  definitions, modal buttons, platform branches, tests, and helper pages.
- For branch-specific QA packs, lock the exact comparison range first: base
  branch, head branch or commit, and whether the whole requested range was
  reviewed. Do not claim complete QA coverage for a partial review.
- Convert risky Desktop changes into falsifiable user-visible hypotheses before
  writing flows. Use Desktop risk surfaces such as Electron main process,
  protocol handlers, OS default handlers, settings UI, menus, modals,
  packaging/installers, startup, shortcuts, workspace routing, i18n, and layout.
- Put the visible path directly in the `Action` cell. Do not create separate
  navigation sections or ask testers to open another file for basic UI
  discovery.
- Describe screen region, relative position, icon shape, nearby UI, visible
  text after interaction, and the confirmation state. If a tooltip or menu title
  appears only after hover/click, describe the visible anchor first.
- Prefer the smallest useful proof for the hypothesis: existing tests, targeted
  tests, local UI repro, OS-level repro, or code-path proof when runtime
  validation is not practical.
- For Qase compatibility, keep the flow table columns aligned with
  `qa/flow-template.md` and run `node qa/scripts/validate-flows.mjs qa/<pack>`
  plus `node qa/scripts/export-qase-csv.mjs qa/<pack>` after changes.

### Cross-Platform Compatibility

Use optional chaining with fallbacks for platform-specific APIs:

```typescript
// PREFERRED — works on all platforms without mocks
const uid = process.getuid?.() ?? 1000;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RocketChat/Rocket.Chat.Electron](https://github.com/RocketChat/Rocket.Chat.Electron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
