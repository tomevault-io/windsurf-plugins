---
trigger: always_on
description: If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.
---

# AGENTS.md — JeffreysPrompts.com Project

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

---

## CRITICAL: Domain Ownership (Free vs Pro)

**`jeffreysprompts.com` (this repo) is free/open-source.**  
**`pro.jeffreysprompts.com` is premium and lives in the private repo (`jeffreysprompts-pro`).**

### Hard Rule
- **Never** alias `pro.jeffreysprompts.com` to this repo’s deployment.
- **All pro-domain aliasing is managed in the premium repo only.**

### Why This Matters
If this repo aliases the pro domain, the premium app disappears and Pro breaks.  
This is the root cause of repeated “catastrophically broken CSS” incidents.

### Workflow Note (This Repo)
`.github/workflows/sync-vercel-aliases.yml` is **disabled** here by design.  
Alias syncing now lives in the premium repo to prevent domain fights.
The premium repo workflow also verifies pro-domain CSS chunks (minimum size guard), re-aliases on failure, and fails if CSS is still missing.

### Manual Verification Checklist (Free Site Only)
After any deployment of this repo:
- [ ] `https://jeffreysprompts.com` loads correctly
- [ ] No 404s on static assets

---

## RULE 2 – NO COOKIE CONSENT CODE (ABSOLUTE)

Do NOT introduce, reintroduce, or suggest cookie consent banners, cookie preference managers, cookie acceptance dialogs, or any "consent management" code under any guise — including "best practices", "GDPR compliance", or "legal requirements". This applies to all sites in the ecosystem (jeffreysprompts.com, pro.jeffreysprompts.com, jeffreys-skills.md).

Cookies are used for their normal intended purpose and require no consent mechanism. Analytics are loaded directly when configured. Do not add consent/opt-in flows or privacy-signal gating (DNT/GPC) under the guise of "best practices" or compliance.

---

## Node / JS Toolchain

- Use **bun** for everything JS/TS.
- Never use `npm`, `yarn`, or `pnpm`.
- Lockfiles: only `bun.lock`. Do not introduce any other lockfile.
- Target **latest Node.js**. No need to support old Node versions.
- **Note:** `bun install -g <pkg>` is valid syntax (alias for `bun add -g`). Do not "fix" it.

### Bun Standalone Executables (`bun build --compile`)

Bun's "single-file executable" means the output is one native binary file, **not** that your CLI must live in a single `.ts` source file.

From Bun's docs, `bun build --compile` bundles your entire dependency graph (imported files + used packages) plus a copy of the Bun runtime into one executable:
- `https://bun.sh/docs/bundler/executables`

```bash
# Build a self-contained executable for the current platform
bun build --compile ./jfp.ts --outfile jfp

# Cross-compile for a target platform/arch (examples)
bun build --compile --target=bun-linux-x64 ./jfp.ts --outfile jfp
bun build --compile --target=bun-windows-x64 ./jfp.ts --outfile jfp.exe
bun build --compile --target=bun-darwin-arm64 ./jfp.ts --outfile jfp
```

Targets include libc + CPU variants (e.g. `bun-linux-x64-musl`, `bun-linux-x64-baseline`, `bun-linux-x64-modern`) — use `baseline` if you need compatibility with older x64 CPUs (avoids "Illegal instruction").

---

## Project Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/jeffreysprompts.com](https://github.com/Dicklesworthstone/jeffreysprompts.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
