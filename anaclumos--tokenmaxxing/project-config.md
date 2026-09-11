---
trigger: always_on
description: Repo-specific rules only. The owner's global rules load alongside this file in every session, so nothing here repeats them; where this file is silent, the global rule applies. Long-form detail lives in `DESIGN.md`, `docs/content/docs/`, and `.memory/`. The source carries no comments (owner ruling 2026-08-30), so rationale that could regress goes to `.memory/`, not the code site. Link to it, do not inline it.
---

# Agent rules

Repo-specific rules only. The owner's global rules load alongside this file in every session, so nothing here repeats them; where this file is silent, the global rule applies. Long-form detail lives in `DESIGN.md`, `docs/content/docs/`, and `.memory/`. The source carries no comments (owner ruling 2026-08-30), so rationale that could regress goes to `.memory/`, not the code site. Link to it, do not inline it.

## The project

tokenmaxxing pools the owner's own Claude Code and Codex logins, swaps to a fresher account as one nears its 5-hour or weekly limit, and keeps sessions continuable across swaps. Design: `DESIGN.md`. User docs: `docs/content/docs/`.

No external installed user base, so this is pre-production code: delete old-state compatibility rather than carry it forward.

## Safeguards and machine gotchas

- The owner's Mac runs tokenmaxxing straight from this working tree, so a bad git operation breaks the live install.
- This machine runs live supervisors, hooks, the periodic check, and the owner's real claude sessions. Stop processes by PID, and never kill a running session or supervisor to free a resource without asking.
- The owner's hosts are managed environments: never run `init`, `add`, `auth`, `uninstall`, or anything that writes settings.json, launchd/systemd units, shell rc, or the global package on a live host (owner, 2026-08-30). Ship code; activation is the owner's step.
- This working tree is a shared checkout (the owner and other agents work in it live). Stage commits by explicit path, never `git add -A`/`-u` (hook correction, 2026-08-30).
- Never print credential material: keychain blobs, `.credentials.json`, `auth.json`, OAuth access or refresh tokens. Report account labels and status only. A ky error carries its request, Authorization header included.
- Ask before any run that meters real quota or opens a session window (`status --ping`, live-pool runs). Free `/usage` reads are fine.
- This repo is PUBLIC. The no-Slack-info and no-device-info rule covers PR bodies, commit messages, review replies, release notes, and docs, not just `.memory`.
- Pool account labels are personal data, the same as emails and organization names: the owner names seats after people. Never paste `status`, `ls`, `doctor`, or log output that carries labels, emails, or org names into a PR body, commit, review reply, doc, memory, or subagent prompt. Mask or count them ("six team seats", "15 accounts") the way emails are masked (owner correction 2026-09-06).
- `rm` is aliased to `rm -i` on the owner's Mac. In a non-TTY shell the prompt gets EOF, nothing is deleted, and it still exits 0, so `rm f && echo ok` lies. Deletion is the owner's call except for artifacts this session created; when you must, pass `-f` and verify the path is gone.
- macOS has no `/bin/true`. Use `/usr/bin/true` in tests.
- Env overrides parse through zod at the read site rather than a central `env.ts`, because the CLI's knobs are all optional. Unset parses to undefined and the feature degrades there.
- State files that exist but fail to parse THROW. A truncated `accounts.json` read as an empty pool once let `init` overwrite it.
- A configured-but-missing path (`claudeBin`, `codexBin`, credential locations) fails fast. Never fall through to a PATH scan: seeding `/bin/true` as claudeBin made the scan resolve the real installed wrapper and wedge an E2E for 15 minutes, the same shape that fed the runaway-recursion incident.
- The Mac runs the working tree, the Linux boxes run an npm global that nothing auto-updates, so version skew is chronic. For any works-on-Mac-not-Linux report, compare the box's installed version against the repo before anything else.
- Core deps are zod, es-toolkit, ky, and `@modelcontextprotocol/sdk` (stdio MCP for the Agent Plugin). The global default stack does not apply (there is no date-fns here, date math goes through `Intl` in `parseResetClock`).
- Keep `node:fs`, which is Bun-native. `Bun.file`/`Bun.write` are async-only, non-atomic, and have no create-mode, so they cannot serve the 0600 credential store or the flock fd. When asked to simplify this, that is the answer.

## Credentials and identity

- Identity is the `accountUuid` that `fetchTokenIdentity` reports for a token (`GET /api/oauth/profile`), never the organization, never a stored label, and never a blob comparison. Team seats share one `organizationUuid`, so an org-keyed lookup collapses every seat onto the first one in the array: all seats read as active, share the tee, and the harvest lands in the wrong slot (2026-09-06). Two rotations of one account's token differ byte-for-byte. Park a credential under its token's real owner, and commit the active label inside the same critical section as the credential writes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anaclumos/tokenmaxxing](https://github.com/anaclumos/tokenmaxxing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
