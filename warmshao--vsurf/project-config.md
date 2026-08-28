---
trigger: always_on
description: - No fluff or cheerful filler text
---

# Development Rules

## Conversational Style

- No fluff or cheerful filler text
- Keep answers short and concise
- No emojis in commits, issues, PR comments, or code
- Technical prose only, be kind but direct (e.g., "Thanks @user" not "Thanks so much @user!")

## Code Quality

- Read files in full before making wide-ranging changes, before editing files you have not already fully inspected, and when the user asks you to investigate or audit something. Do not rely only on search snippets for broad changes.
- Don't be too verbose with comments in the code. Only write comments when there is serious ambiguity
- No `any` types unless absolutely necessary
- Check node_modules for external API type definitions instead of guessing
- **NEVER use inline imports** - no `await import("./foo.js")`, no `import("pkg").Type` in type positions, no dynamic imports for types. Always use standard top-level imports.
- NEVER remove or downgrade code to fix type errors from outdated dependencies; upgrade the dependency instead
- Always ask before removing functionality or code that appears to be intentional
- Do not preserve backward compatibility unless the user explicitly asks for it
- Never hardcode key checks with, eg. `matchesKey(keyData, "ctrl+x")`. All keybindings must be configurable. Add default to matching object (`DEFAULT_EDITOR_KEYBINDINGS` or `DEFAULT_APP_KEYBINDINGS`)
- NEVER modify `packages/ai/src/models.generated.ts` directly. Update `packages/ai/scripts/generate-models.ts` instead.

## Commands

- After code changes (not documentation changes): `npm run check` (get full output, no tail). Fix all errors, warnings, and infos before committing.
- Note: `npm run check` does not run tests.
- NEVER run: `npm run dev`, `npm run build`, `npm test`
- Only run specific tests if user instructs: `npx tsx ../../node_modules/vitest/dist/cli.js --run test/specific.test.ts`
- Run tests from the package root, not the repo root.
- If you create or modify a test file, you MUST run that test file and iterate until it passes.
- When writing tests, run them, identify issues in either the test or implementation, and iterate until fixed.
- For `packages/coding-agent/test/suite/`, use `test/suite/harness.ts` plus the faux provider. Do not use real provider APIs, real API keys, or paid tokens.
- Put issue-specific regressions under `packages/coding-agent/test/suite/regressions/` and name them `<issue-number>-<short-slug>.test.ts`.

## Daemon Protocol Changes

- Classify every daemon command, event, and response-shape change as backward-compatible, capability-gated, or incompatible.
- Add optional features behind a negotiated server capability. Clients must check the capability before sending the command or depending on the event.
- Bump `DAEMON_PROTOCOL_VERSION` for incompatible changes or when startup begins requiring behavior an older daemon cannot provide.
- Update `DAEMON_SCHEMA_REVISION`, the command/event compatibility maps, and both new-client/old-daemon and old-client/new-daemon tests for every wire change.
- Optional daemon metadata and UI features must degrade locally. They must not prevent the agent, session attachment, or interactive startup from working.
- Never make a new daemon command part of startup without a protocol or capability gate.

## Dependencies

- A 7-day minimum release age applies to all dependency updates: `.npmrc` sets `min-release-age=7` and `.github/dependabot.yml` uses a matching `cooldown`. Never bypass it for routine updates.
- Enforcement requires npm >= 11.10; older npm silently ignores the setting, so use a current npm when updating dependencies.
- For an urgent security patch younger than 7 days, override explicitly: `npm install --min-release-age=0 <pkg>`.

## GitHub Workflow

When creating issues:

- Add `pkg:*` labels to indicate which package(s) the issue affects
  - Available labels: `pkg:agent`, `pkg:ai`, `pkg:coding-agent`, `pkg:tui`
- If an issue spans multiple packages, add all relevant labels

When posting issue/PR comments:

- Write the full comment to a temp file and use `gh issue comment --body-file` or `gh pr comment --body-file`
- Never pass multi-line markdown directly via `--body` in shell commands
- Preview the exact comment text before posting
- Post exactly one final comment unless the user explicitly asks for multiple comments
- If a comment is malformed, delete it immediately, then post one corrected comment
- Keep comments concise, technical, and in the user's tone

When closing issues via commit:

- Include `fixes #<number>` or `closes #<number>` in the commit message
- This automatically closes the issue when the commit is merged

## PR Workflow

- Analyze PRs without pulling locally first
- If the user approves: create a feature branch, pull PR, rebase on main, apply adjustments, commit, merge into main, push, close PR, and leave a comment in the user's tone
- We work in feature branches until everything is according to the user's requirements. Never merge PRs by yourself.

## Testing VSurf Interactive Mode with tmux

To test VSurf's TUI in a controlled terminal environment:

```bash
# Create tmux session with specific dimensions
tmux new-session -d -s vsurf-test -x 80 -y 24

# Start VSurf from source
tmux send-keys -t vsurf-test "cd /Users/kevin/vsurf/vsurf && ./vsurf.sh" Enter


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [warmshao/vsurf](https://github.com/warmshao/vsurf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
