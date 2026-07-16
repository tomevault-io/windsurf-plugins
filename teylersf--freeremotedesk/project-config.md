---
trigger: always_on
description: This file is written for you (Claude Code, Cursor, Aider, Codex, Continue, etc.) — the AI coding agent your user has asked to set up FreeRemoteDesk for them.
---

# Instructions for AI coding agents

This file is written for you (Claude Code, Cursor, Aider, Codex, Continue, etc.) — the AI coding agent your user has asked to set up FreeRemoteDesk for them.

**Your job:** deploy the user's own instance of FreeRemoteDesk to their own free-tier Cloudflare + Vercel accounts, then install the host agent on their machine. Total user interaction should be **three browser sign-ins** (one per CLI on first auth) and **one installer double-click**. Everything else is you.

**Do not deviate from these steps.** They've been tested end-to-end.

---

## Prerequisites — check + install these first

Run these check commands. If a command errors, install per the notes and re-check before moving on.

| Tool | Check command | If missing |
|---|---|---|
| Node.js ≥ 20 | `node --version` | Install from https://nodejs.org (LTS). On Windows: `winget install OpenJS.NodeJS.LTS` |
| pnpm ≥ 9 | `pnpm --version` | `corepack enable && corepack prepare pnpm@9.12.0 --activate` |
| GitHub CLI | `gh --version` | `winget install GitHub.cli` (Windows) / `brew install gh` (macOS) / see cli.github.com |
| Wrangler (Cloudflare) | `npx wrangler --version` | Comes with `pnpm install` — no separate step |
| Vercel CLI | `npx vercel --version` | Comes with `pnpm install` — no separate step |

Also verify the three CLIs are authenticated:

```bash
gh auth status                         # if not: gh auth login
npx wrangler whoami                    # if not: npx wrangler login  (opens browser)
npx vercel whoami                      # if not: npx vercel login    (opens browser)
```

For any CLI that isn't logged in: run its login command, then wait for the user to complete the browser flow. **Do not skip this** — every subsequent step depends on all three auths being live.

---

## Step 1 — Clone the repo

```bash
git clone https://github.com/Teylersf/freeremotedesk
cd freeremotedesk
pnpm install --frozen-lockfile
```

---

## Step 2 — Deploy signaling to Cloudflare Workers

```bash
cd signaling
npx wrangler deploy
```

**Extract the deployed URL** from the wrangler output. It looks like:
`https://freeremotedesk-signaling.<username>.workers.dev`

The URL is on a line like `Published freeremotedesk-signaling (X sec) https://…workers.dev`.

**Save it as `$SIGNALING_URL`** for later steps.

**Verify:** `curl "$SIGNALING_URL/health"` should return `{"ok":true,"service":"freeremotedesk-signaling"}`.

If verification fails, do NOT proceed — the wrangler deploy silently succeeded but the routing didn't work. Retry with `npx wrangler deploy --dispatch-namespace freeremotedesk` or investigate.

---

## Step 3 — Deploy the PWA to Vercel

```bash
cd ../pwa
# Set the signaling URL as a build-time env var, then deploy production
echo "$SIGNALING_URL" | npx vercel env add VITE_SIGNALING_URL production
npx vercel deploy --prod --yes
```

The first `vercel deploy` in a fresh repo will prompt you to link a project. Accept defaults (create new project, name `freeremotedesk`, root directory `./` — you're already in `pwa/`).

**Extract the deployed URL** from the vercel output. It looks like:
`https://freeremotedesk-<hash>.vercel.app`

**Save it as `$PWA_URL`** for later steps.

**Verify:** `curl -s "$PWA_URL" | grep -c FreeRemoteDesk` should return `1` or higher.

---

## Step 4 — Download + install the host agent

Detect the user's OS + arch:

| OS + arch | Asset name pattern |
|---|---|
| Windows x64 | `*_x64_en-US.msi` (preferred) or `*_x64-setup.exe` |
| macOS Apple Silicon | `*_aarch64.dmg` |
| macOS Intel | `*_x64.dmg` |
| Linux (Debian/Ubuntu) | `*_amd64.deb` |
| Linux (other) | `*_amd64.AppImage` |

```bash
# Windows PowerShell example
gh release download --repo Teylersf/freeremotedesk --pattern "*_x64_en-US.msi" --dir "$env:USERPROFILE\Downloads"
Start-Process "$env:USERPROFILE\Downloads\FreeRemoteDesk_0.1.0_x64_en-US.msi"
```

```bash
# macOS ARM example
gh release download --repo Teylersf/freeremotedesk --pattern "*_aarch64.dmg" --dir "$HOME/Downloads"
open "$HOME/Downloads/FreeRemoteDesk_0.1.0_aarch64.dmg"
# Then instruct user to drag FreeRemoteDesk.app into /Applications
```

```bash
# Linux Debian/Ubuntu example
gh release download --repo Teylersf/freeremotedesk --pattern "*_amd64.deb" --dir /tmp
sudo dpkg -i /tmp/FreeRemoteDesk_*_amd64.deb
```

---

## Step 5 — Configure the agent

The user must launch the agent from their app launcher / Start menu. Its first-run wizard will prompt for:

- **Signaling URL** → paste `$SIGNALING_URL` from step 2
- **PWA URL** → paste `$PWA_URL` from step 3
- **Start on boot** → user's choice, recommend yes

You cannot fill this in for them — it's a native window. Give them the two values in a copy-friendly format and instruct them clearly.

Example message to user:

> Setup complete! Launch FreeRemoteDesk from your Start menu. It'll ask two questions — copy/paste these:
>
> - Signaling URL: `https://freeremotedesk-signaling.foo.workers.dev`
> - PWA URL: `https://freeremotedesk-abc123.vercel.app`
>
> Then click "Start session" → pick a screen → open the PWA URL on your phone → enter the 6-char code.

---

## Failure modes + recovery


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Teylersf/freeremotedesk](https://github.com/Teylersf/freeremotedesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
