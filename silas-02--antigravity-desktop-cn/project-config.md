---
trigger: always_on
description: These instructions apply to every AI agent and contributor working in this repository. The goal is to deliver natural, complete, and maintainable Simplified Chinese localization without breaking Antigravity behavior, user content, technical identifiers, or renderer performance.
---

# Antigravity Chinese Localization — Project Instructions

These instructions apply to every AI agent and contributor working in this repository. The goal is to deliver natural, complete, and maintainable Simplified Chinese localization without breaking Antigravity behavior, user content, technical identifiers, or renderer performance.

## 1. Current target and sources of truth

- The current localization release is **v2.12.2**, targeting the official **Antigravity v2.12.2** client. The current version dictionary is `dicts/v2.12.2.json`; keep these values, the release date, and the support statement in `README.md` consistent.
- Treat the repository's actual files and executable behavior as authoritative. If these instructions, `README.md`, a dictionary filename, an engine comment, or the current client disagree, report the mismatch and update only what the requested task requires.
- The engine loads **every** `dicts/*.json` file. A version bump must leave exactly one active version dictionary, migrate the previous dictionary deliberately, update all current-version references, and be checked against official packaged source and metadata. If renderer source is unavailable, verify the package version and relevant packaged injection surfaces, use observed client UI as renderer evidence, and state that limitation instead of claiming a full source audit. Do not change tags or release metadata unless explicitly requested.
- Keep the user-facing installation, restoration, platform, path, dependency, and compatibility claims in `README.md` consistent with the actual engine and supported launchers.

## 2. Ownership and dictionary semantics

- `dicts/*.json` contains fixed renderer-facing text. Search existing dictionaries before editing, reuse established wording, and put each entry in the most appropriate module dictionary.
- `loadDictionary()` reads all dictionaries in lexicographic filename order, collapses whitespace, trims keys, and normalizes curly quotes. A later normalized key overrides an earlier one; the renderer also has a case-insensitive fallback. Invalid JSON is skipped by the client loader. The repository verifier is responsible for detecting these mechanical failures.
- Prefer exact entries for short labels and complete fixed sentences. Entries longer than 20 characters may also be used for substring replacement, so their source text must remain sufficiently specific. Use a fragment-only key only when the client is confirmed to render that fragment as an independent, unambiguous UI node.
- `localization_engine.js` both generates the renderer translation injection and patches packaged Electron surfaces while unpacking/repacking `app.asar`. Put fixed native menu, tray, loading, and updater text in the corresponding narrow engine injection block; use bounded engine rules for dynamic or structurally fragmented renderer text.
- Preserve `--brand-title`: English retains the official `Antigravity` brand, hidden removes its visual label, and translated permits the Chinese brand translation.
- `install.sh`, `双击安装中文汉化.command`, and `双击安装中文汉化.bat` are the supported installation entry points; `uninstall.sh`, `双击卸载还原官方英文.command`, and `双击卸载还原官方英文.bat` are the restoration entry points. `localization_engine.js` is side-effecting and is never a development test command.
- Treat shipped `.bat` files as GBK/CRLF release artifacts protected by `.gitattributes`. Change them only when explicitly requested, using `convert_to_gbk.ps1`, and verify the resulting encoding and line endings.

## 3. Translation quality and protected content

- Translate complete meaning in its real UI context rather than word by word. Keep buttons, menus, prompts, errors, and status text concise and natural, using standard Simplified Chinese punctuation.
- Use the following terminology unless context requires a documented exception:
  - `agent` → “智能体”
  - `conversation`, or a persisted chat/thread/history item → “会话”
  - `chat` used as a visible action, capability, or button → “聊天”
  - a conversational exchange or agent dialogue → “对话” when natural in context
  - `project` → “项目”; `workspace` → “工作区”; `worktree` → “工作树”
  - `goal` → “目标”; `task` → “任务”
  - `file` → “文件”; `folder` → “文件夹”
  - `page` → “页面” or counted “个页面”; `search` → “搜索” or counted “次搜索”
  - `tool` → “工具”
  - UI `artifact` or a generated deliverable → “交付件”; a technical artifact may be “构件”
- Keep project and workspace distinct even when the UI visually groups them.
- Translate only confirmed product-owned wrapper text such as dialog or toast titles, buttons, labels, and fixed explanatory copy. Never translate user prompts or chat bodies, third-party web content, generated model responses, editor or file content, terminal or subprocess output, CLI/Git diagnostics, stack traces, URLs, paths, commands, code, shortcuts, secrets, credentials, model or product names, MCP/API/configuration identifiers, environment variables, version numbers, Git refs or hashes, exit codes, or error IDs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Silas-02/antigravity-desktop-cn](https://github.com/Silas-02/antigravity-desktop-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
