---
trigger: always_on
description: Tauri 2 desktop shell for `omp` (oh-my-pi). React UI served from `src/` in `tauri dev` (**no bundler** — JSX is transpiled in-browser by `@babel/standalone`) and from the precompiled `dist/` in release builds (see *Release frontend*). Rust backend spawns `omp --mode rpc-ui` per tab.
---

# AGENTS.md

Tauri 2 desktop shell for `omp` (oh-my-pi). React UI served from `src/` in `tauri dev` (**no bundler** — JSX is transpiled in-browser by `@babel/standalone`) and from the precompiled `dist/` in release builds (see *Release frontend*). Rust backend spawns `omp --mode rpc-ui` per tab.

## Commands

| Task | Command |
|---|---|
| Install Tauri CLI | `npm install` |
| Dev | `npm run dev` |
| Prod build | `npm run build` (= `tauri build --config src-tauri/tauri.dist.conf.json`: embeds the precompiled `dist/`) |
| Build `dist/` only | `npm run build:frontend` (writes the gitignored `dist/`; also the last step of `npm test`) |
| Rust check (CI) | `cd src-tauri && cargo check --locked` |
| Rust fmt | `cd src-tauri && cargo fmt` |
| Rust lint (must stay clean) | `cd src-tauri && cargo +nightly clippy --all-targets --all-features -- -W clippy::pedantic -W clippy::nursery -D warnings` |
| Rust tests | `cd src-tauri && cargo test` |
| All JS regression scripts | `npm test` — add a new `tests/test-*.mjs` to its chain in `package.json` (not `tests/test-rpc.mjs`, which needs a live omp) |
| Probe omp RPC | `node tests/test-rpc.mjs` |
| Keymap chord regression | `node tests/test-keymap.mjs` (or `npm run test:keymap`) |
| Markdown XSS-escaping regression | `node tests/test-markdown.mjs` (or `npm run test:markdown`) |
| Chat scroll-pin regression | `node tests/test-scroll-pin.mjs` (or `npm run test:scroll-pin`) |
| Session-title auto-rename gates regression | `node tests/test-session-title.mjs` (or `npm run test:session-title`) |
| Slash-command palette regression | `node tests/test-slash-commands.mjs` (or `npm run test:slash-commands`) |
| Prompt history regression | `node tests/test-prompt-history.mjs` (or `npm run test:prompt-history`) |
| Subagent manager reducer regression | `node tests/test-subagents.mjs` (or `npm run test:subagents`) |
| Bash tool-card output/copy text regression | `node tests/test-tool-output.mjs` (or `npm run test:tool-output`) |
| Updater state regression | `node tests/test-updater.mjs` (or `npm run test:updater`) |
| Updater feed assembly regression | `node tests/test-updater-json.mjs` (or `npm run test:updater-json`) |
| Model picker recent group regression | `node tests/test-recent-models.mjs` (or `npm run test:recent-models`) |
| Image viewer geometry regression | `node tests/test-lightbox.mjs` (or `npm run test:lightbox`) |
| Project navigation + tab-layout regression | `node tests/test-project-nav.mjs` (or `npm run test:project-nav`) |
| Empty-workspace backdrop regression | `node tests/test-matrix-rain.mjs` (or `npm run test:matrix-rain`) |
| Per-tab composer/plan state + collapsed-paste regression | `node tests/test-session-ui.mjs` (or `npm run test:session-ui`) |
| Goal mode reducer (rows, snapshots, strip, budget input) regression | `node tests/test-goal.mjs` (or `npm run test:goal`) |
| Steer/follow-up queue strip regression | `node tests/test-message-queue.mjs` (or `npm run test:message-queue`) |
| Ask dialog answer rules regression | `node tests/test-ask-dialog.mjs` (or `npm run test:ask-dialog`) |
| Run state + failed-request + job row + retry row regression | `node tests/test-turn-status.mjs` (or `npm run test:turn-status`) |
| Tab-switch transcript merge regression | `node tests/test-transcript-merge.mjs` (or `npm run test:transcript-merge`) |
| Conversation-tree model regression | `node tests/test-conversation-tree.mjs` (or `npm run test:conversation-tree`) |

`omp` must be on PATH (`%LOCALAPPDATA%\omp\omp.exe` on Win). CI and every release run the same suite (`.github/workflows/tests.yml`): `cargo test --locked` on win/linux/mac, plus `npm test`.

**omp version floor.** `MIN_OMP_VERSION` in `src-tauri/src/agent/spawn.rs` is the only omp version knowledge in the app: `spawn_omp` runs `omp --version` and refuses an older omp with a message naming both versions, or unreadable output with one quoting it and the required version; either is cached in `last_errors` like any spawn failure, and the bridge's `openSession`/`openProject`/`resumeSession` file it as a note (`_openReported`). Only a pass is remembered, so an `omp update` takes effect without an app restart. A change that starts using an RPC command, event or `get_state` field from a newer omp release raises the constant in the same commit (plus a CHANGELOG line); no code branches on the omp version or keeps a fallback for an omp below the floor. Finding a command's first release in omp's repo: `git log --reverse --format=%h -S '"<command>"' -- packages/coding-agent/src/modes/rpc`, then `git tag --contains <sha> --sort=v:refname` (first line; plain `--contains` sorts `v18.4.10` before `v18.4.9`). State fields are unquoted (`-S 'isSettled:'`), and agent-core events live outside `modes/rpc`, so widen the path for those.

The floor is 18.4.11: the conversation tree's `fork` command (see *Conversation tree*) is the newest RPC the app sends. `branch` is older. The tree reads session files directly, so it needs no other omp change.

## Architecture

Three layers:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apoc/omp-desktop](https://github.com/apoc/omp-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
