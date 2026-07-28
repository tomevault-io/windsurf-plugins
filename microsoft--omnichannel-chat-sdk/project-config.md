---
trigger: always_on
description: Local development, packaging, and end-to-end test workflow across the four LCW repos (omnichannel-sdk, omnichannel-chat-sdk, omnichannel-chat-widget, CRM.OmniChannel.LiveChatWidget). Paths and commands below assume the typical Windows layout with all four repos cloned side-by-side under `c:\lcw` and a Git Bash shell; adjust for WSL (e.g. `/mnt/c/lcw/...`) or macOS/Linux (`~/src/lcw/...`) as needed. Use when changing code in any of these repos and validating the change in the locally served live 
---


# LCW multi-repo local development skill

This workspace contains four interlinked repos that together produce the Live Chat Widget. When making a change in any one of them, follow the rules below to (a) propagate the change to the running widget for local testing and (b) prepare a clean PR per repo.

## Repo topology and dependency direction

```
omnichannel-sdk (low-level OC REST client)
        │  consumed by
        ▼
omnichannel-chat-sdk  (high-level chat SDK; owns ACS adapter dep)
        │  consumed by (via yarn link)
        ▼
omnichannel-chat-widget  (React widget components)
        │  consumed by (via yarn link)
        ▼
CRM.OmniChannel.LiveChatWidget  (LCW host app, webpack bundle, serves localhost:9000)
```

Local paths:

| Repo | Path | Build command | Output |
|---|---|---|---|
| omnichannel-sdk | `c:\lcw\omnichannel-sdk` | `yarn build` | `lib/` |
| omnichannel-chat-sdk | `c:\lcw\omnichannel-chat-sdk` | `npm run build:tsc` | `lib/` |
| omnichannel-chat-widget | `c:\lcw\omnichannel-chat-widget\chat-widget` | `yarn build:esm && yarn build:cjs` | `lib/esm`, `lib/cjs` |
| CRM.OmniChannel.LiveChatWidget | `c:\lcw\CRM.OmniChannel.LiveChatWidget\src` | `SKIP_TS_CHECK=1 yarn build:webpack:dev` | `dist/v2scripts/ocw.js` |

`omnichannel-chat-widget` also has a sibling package `chat-components` under the same repo; build it the same way if you touched it.

## Linkage facts (verified)

- `omnichannel-chat-sdk` is symlinked into chat-widget and LCW via `yarn link`. Check with (Git Bash on Windows shown; WSL/macOS users substitute paths):
  ```bash
  ls -la /c/lcw/omnichannel-chat-widget/chat-widget/node_modules/@microsoft/omnichannel-chat-sdk
  ls -la /c/lcw/CRM.OmniChannel.LiveChatWidget/src/node_modules/@microsoft/omnichannel-chat-sdk
  ```
  The expected symlink target depends on which shell ran `yarn link` originally:
  - **Git Bash / PowerShell / cmd on Windows** (yarn installed for the Windows user): `C:\Users\<user>\AppData\Local\Yarn\Data\link\@microsoft\omnichannel-chat-sdk\` (Git Bash also surfaces this as `/c/Users/<user>/AppData/Local/Yarn/Data/link/...`; PowerShell: `$env:LOCALAPPDATA\Yarn\Data\link\...`).
  - **WSL** (yarn installed inside the WSL distro): `~/.config/yarn/link/@microsoft/omnichannel-chat-sdk` inside WSL — note this is the WSL user's home, not `%USERPROFILE%`. WSL accessing files under `/mnt/c/...` will see the Windows link store at `/mnt/c/Users/<user>/AppData/Local/Yarn/Data/link/...` instead.
  Mixing a Windows-installed yarn with a WSL-installed yarn is the most common reason `yarn link` appears to silently no-op — each yarn has its own link registry.
- `omnichannel-chat-widget` (the `chat-widget` and `chat-components` sub-packages) are likewise yarn-linked into LCW when working locally.
- The ACS adapter (`@microsoft/botframework-webchat-adapter-azure-communication-chat`) is a transitive dep installed independently in each of the three downstream node_modules trees. Lockfiles in each downstream repo pin it independently. Do **not** assume changing chat-sdk's `package.json` will alter the version installed in chat-widget or LCW node_modules — those come from their own lockfiles.

## The standard local-test loop (update → build → refresh)

When a change is made anywhere in the stack, rebuild every consumer below it, **wipe LCW's webpack cache**, then refresh the browser.

```bash
# 1. (if omnichannel-sdk changed)
cd /c/lcw/omnichannel-sdk && yarn build

# 2. (if omnichannel-chat-sdk changed)
cd /c/lcw/omnichannel-chat-sdk && npm run build:tsc

# 3. (if omnichannel-chat-widget changed)
cd /c/lcw/omnichannel-chat-widget/chat-widget && yarn build:esm && yarn build:cjs

# 4. ALWAYS for LCW (webpack cache MUST be wiped between linked-dep edits or
#    stale modules will be served even after rebuild)
cd /c/lcw/CRM.OmniChannel.LiveChatWidget/src
rm -rf node_modules/.cache/webpack
SKIP_TS_CHECK=1 yarn build:webpack:dev

# 5. Start (or keep running) the dev server, then hard-reload the browser
#    Test page: http://localhost:9000/lcw-test.html
```

If the LCW dev server is already running with file-watching, you still must wipe `node_modules/.cache/webpack` after editing a yarn-linked package, because webpack 5's persistent cache key does not invalidate on symlinked-source mtime changes.

## Common gotchas

1. **ACS adapter resolves to `0.0.1-beta-1` instead of `beta.N`** — caret ranges (`^0.0.1-beta.6` etc.) can resolve to the rogue prerelease `0.0.1-beta-1` even though intuitively it looks like `beta.6` should win. Why this happens per SemVer 2.0.0 §11.4:
   - Pre-release identifiers are split on `.`. `beta-1` is **one** identifier (the `-` is a literal character inside an alphanumeric identifier, not a separator). `beta.6` is **two** identifiers: `beta` and `6`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/omnichannel-chat-sdk](https://github.com/microsoft/omnichannel-chat-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
