---
trigger: always_on
description: This repository contains a local Destiny 2 MCP server with Bungie OAuth authentication.
---

# Agent Instructions

This repository contains a local Destiny 2 MCP server with Bungie OAuth authentication.

For any agent, not only Codex:

- Read `README.md` first for the platform-neutral installation and verification flow.
- Read `skills/destiny2-mcp/SKILL.md` for tool routing and evidence boundaries.
- Read `skills/destiny-mcp-setup/SKILL.md` only for installation, OAuth, registration, or setup troubleshooting.
- Run `scripts/verify_mcp.py` for the real MCP handshake; do not treat registration alone as proof of readiness.

## 遇到 Bungie API 问题怎么查

官方文档一页全包：<https://bungie-net.github.io/> —— OAuth scope 表、每个端点的请求/响应、AWA 三段流程、DestinyComponentType 组件枚举，都在这一页。

要深查（离线读原文、确认字段名或端点路径）就把它拉到仓库外：

    python scripts/fetch_bungie_api_docs.py            # 抓到 ~/.destiny_mcp/reference/，带日期与 manifest.json（版本号/字节数/sha256）
    python scripts/fetch_bungie_api_docs.py --check    # 只比在线文档版本号与上次记录，看文档换没换版本（--exit-code 时不一致退出 1）

那份快照**不进 git**：它是上游产物、不是我们的口径；我们依赖的是自己的实测事实。

**实测优先于文档**：`docs/reference/bungie_api.md` 记着我们踩过的坑与真机结论（scope、AWA、free vs 付费插槽接口、组件 305 必须带清单类、写入后的同步窗口、神器三个 hash 家族…）。与官方文档冲突时，以那里的「实测」为准，并去更新那一条。

## Skill maintenance

`skills/destiny2-mcp/` is the single source for the agent-facing guide. Nothing in it may assume one host: the same folder is loaded by hosts with a skills directory, and hosts without one get a pointer block plus the public URL advertised in the MCP handshake.

- After editing anything under `skills/destiny2-mcp/`, run `scripts/install_skill.py` so the installed copy keeps up. It installs into **the host that is running you** by default (DSH: `~/.dsh/skills/`, Claude Code: `~/.claude/skills/`, Codex: `~/.codex/skills/`, detected from the environment); `--list` shows the current host, `--host <name>` picks one, `--all` installs into every detected host, `--pointer` refreshes the instruction-file pointers, and `--mcp` registers the MCP server (writes the DSH profile patch; prints the paste-ready command for the others).
- A new user installs this for the agent they are talking to, not for every agent on the machine: do not spread copies into Codex and Claude "just in case".
- Never write into a directory the host manages itself (for example Cursor's `skills-cursor`); use `--target` or `--pointer <file>` for unlisted hosts.
- `tests/test_skill_contracts.py` checks `references/routing.md` against the code: intent coverage, parameter ownership, write intents, community categories. A red test there means the document is stale, not that the check is too strict.
- `tests/test_skill_install.py` checks the delivery layer: pointer idempotence, mirror sync, and that `config.ROUTING_GUIDE_URL` matches the installer's URL.
- `destiny_mcp/error_codes.py` is the only place `error.code` may come from: literal codes live in `ErrorCode`, exception classes are turned into codes by `code_for_exception()` (the class name *is* the contract), and write failures use `write_failed(intent)`. `tests/test_error_codes.py` rejects bare string codes and pins the exception→code mapping.
- `destiny_mcp/vocabulary.py` is the only place Chinese labels live (six stats, classes, locations, elements, legacy stat names). Labels are the Manifest's official Chinese strings; input aliases may be many but must resolve to one key. `tests/test_vocabulary.py` pins the official names, checks aliases round-trip (legacy names must point at *existing* labels), and rejects re-copied tables.
- `tests/test_conclusion_paths.py` guards diagnosis/conclusion output: a conclusion module may fall back to weaker wording, but never silently — the `except` must re-raise or leave a trace (append / reason / log). Add new conclusion modules to `_CONCLUSION_MODULES` there.
- `tests/test_architecture_layers.py` guards the module graph: imports may only go downwards (`_LAYERS`), no two modules may import each other, and every `svc["…_svc"]` key used in `tools/` must be declared in `service_context.ServiceContext`. A new top-level module must be registered there, otherwise the test fails on purpose — decide its layer instead of letting the rule silently skip it.

## Setup-related tasks

For installation, reinstallation, OAuth login, Codex MCP registration, or setup troubleshooting:

1. Read `skills/destiny-mcp-setup/SKILL.md` completely before taking action.
2. Follow that Skill through a real MCP handshake and verification; registration alone is not sufficient.
3. Run `skills/destiny-mcp-setup/scripts/verify_mcp.py` and require `BUNGIE_PROFILE_CHECK=ok` plus all eight tools in the normal profile.
4. Preserve unrelated worktree changes and inspect existing `.env`, OAuth tokens, and Codex MCP entries before changing them.

Never print or request secrets in chat. This includes `.env` contents, Bungie API keys, OAuth client secrets, authorization codes, access tokens, refresh tokens, and callback URLs containing authorization codes. Ask users to enter credentials locally in `.env`.

After registering or changing the MCP server, tell the user to restart Codex or open a new task so the new server is discovered.

## 写代码的习惯

这些不是风格偏好，是踩过的坑换来的。能自动化的都配了守门测试——改代码前**先看它守什么**。

### 单一出处：一个事实只写一次

| 事实 | 唯一出处 | 守门 |
| --- | --- | --- |
| 错误码 | `destiny_mcp/error_codes.py` | `tests/test_error_codes.py`（禁止裸字符串；异常类名即契约） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [8z2wz746tv-png/destiny2-mcp](https://github.com/8z2wz746tv-png/destiny2-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
