---
trigger: always_on
description: AI-native automation layer for **EasyEDA Pro (嘉立创EDA专业版)**. A skill drives a
---

# easyeda-agent

AI-native automation layer for **EasyEDA Pro (嘉立创EDA专业版)**. A skill drives a
Go daemon, which dispatches typed schematic actions to a connector extension
running inside EasyEDA, which calls the official `eda.*` API.

```
skill ──▶ Go CLI/daemon ──WebSocket──▶ connector .eext ──▶ eda.* API
          (typed actions)   49620-49629   (in EasyEDA Pro)
```

## 官方插件库调研参考
文章：docs/ecosystem-survey.md，遇到什么不确认的情况可以来这里参考分析，并更新认知到相应文档；

## 首要准则 — Skill 优先

> **本项目是「边开发、边更新 Agent Skill」的联合开发模式。**
>
> - **开发和测试的主要对象是 Skill**（唯一对外入口 `skills/easyeda-agent/`）。
> - Go CLI/daemon（`cmd/easyeda` + `internal/`）和连接器插件（`extension/`）是**为 Skill 服务的基础设施**，而非最终目的。
> - 每次改动首先问：「Skill 里的工作流、知识、或 guardrail 需要同步更新吗？」——如果需要，先改 Skill，再改底层实现。
> - 修改底层 action / daemon / 插件后，必须同步更新 Skill 里对应的工具描述、示例、或注意事项。

## 首要准则 — CLI 子命令设计

详见 [`docs/cli-design.md`](docs/cli-design.md)。核心约束：所有明确的功能模块必须以 **Cobra 子命令**方式暴露（`easyeda sch`、`easyeda pcb`、`easyeda bom` …），`--help` 自描述，新功能先设计命令接口再写实现，Skill 描述与子命令签名保持同步。开发闭环：`debug.exec_js` → typed action → Cobra 子命令。

## 首要准则 — 固定测试用例（端到端验收）

**每次做端到端测试，都必须把 [`esp32MiniRequire.md`](esp32MiniRequire.md)
（客户口吻的**原始需求**：4 层板 + 点灯 + 5V 供电端子 + 降压到 3V3 + CH340 USB 烧录 +
BOOT/RESET 按键 + 四角 M3 固定，**故意不含 BOM/UUID/网表**）当输入，让 agent 自己
选型 → 放置 → 编组 → 布线 → `sch layout-lint` → DRC → 转 PCB（4 层叠层 / GND 内电层 /
丝印极性 / 天线 keepout）→ save 完整跑一遍**——照 `skills/easyeda-agent/references/design-flow.md`
流程脊柱（S0–S6 + P0–P10），不是只测单点，**也绝不喂加工过的答案**（喂好 BOM/网表就不叫真实场景了）。
这是 agent 从需求到成品的回归基准：layout-lint / autosave / design-flow / 连接器 任何改动后都重跑此用例。
验收：需求条条落实（0 overlap、0 fatal、网络连通、丝印/极性正、4 层电源树、已落盘）。
测试工程用 `--project ceshi`，测完清理还原。

## Notes

reply as chiense! reply as chiense! reply as chiense!

**Commit directly on `main` — do NOT create feature branches.** Develop and commit
on `main` by default (user preference). Don't `git checkout -b`; just commit to
`main`. Push only when explicitly asked.

## Layout

| Path | What |
|---|---|
| `cmd/easyeda` + `internal/{app,daemon,protocol}` | Go CLI + daemon. `internal/protocol/actions.go` = the 20 typed actions. Daemon: `/health`, `/eda` (connector WS), `/action`. |
| `extension/` | TypeScript connector → esbuild → `.eext`. `src/transport.ts` (port-scan + auto-reconnect), `src/actions.ts` (eda.* handlers + `connect_pin`). |
| `skills/easyeda-agent/` | Merged public skill — short `SKILL.md` router plus `references/` for design flow, schematic, PCB, conventions, canonical data, and `scripts/` for lint/BOM/parts/calibration tools. |
| `docs/FEATURES.md` | Feature-status inventory (20 actions grouped by capability) + roadmap. |
| `skills/easyeda-agent/SKILL.md` | The user-facing skill. |

## Dev workflow

**Keep the daemon hot-reloading while you work** (rebuilds + restarts on any `.go`
change; the connector auto-reconnects because it port-scans 49620-49629 in the
background):

```bash
make dev          # air live-reload of `easyeda daemon` — leave running in a terminal
```

Requires [air](https://github.com/air-verse/air): `go install github.com/air-verse/air@latest`.
Config is `.air.toml`: on any `.go` change it runs `make dev-build` (version-stamped
build → `./bin/easyeda` **and** a best-effort copy to `$PREFIX/bin/easyeda`), then
runs the daemon from that same `./bin/easyeda`. **So the `easyeda` CLI on your PATH
is refreshed on every rebuild — daemon and CLI never drift.** (Before this, air only
rebuilt the daemon; the PATH CLI stayed frozen at the last `make install`, so a new
subcommand like `easyeda doc` was missing until you reinstalled.) If `$PREFIX/bin`
isn't writable, air prints a warning and you run `make install` once with sudo to fix
perms. The dev binary is git-describe-stamped (e.g. `v0.5.1-19-g…-dirty`); a
non-clean stamp is treated as "dev" by the `health` connector-version check, so it
never false-flags a connector as stale against a dev daemon.

Other targets:

```bash
make build        # bin/easyeda (version-stamped via git describe)
make install      # build + install to /usr/local/bin (PREFIX overridable; sudo only if needed)
make daemon       # one-shot daemon (no reload) — prefer `make dev`
make test         # go test ./...
make lint-test    # linter rule-trust harness (orientation consistency + fixtures)
make actions      # print the typed action catalog
make eext         # bump PATCH + build importable .eext, STABLE uuid (update in place: uninstall old → import)
make eext-fresh   # fallback: bump PATCH + FRESH uuid (imports as a new entry; delete the old one) — for when the installed one won't uninstall
make connector    # build .eext at the current version/uuid (no bump — same-version dev only)

skills/easyeda-agent/scripts/lint.sh <project>          # live lint (DIFF if a baseline exists)
skills/easyeda-agent/scripts/lint.sh <project> --save   # full lint + record baseline
```

## Release workflow

```bash
# 一条命令发版：自动把 connector + CLI 统一到同一版本，交叉编译 5 平台，
# 打包 skills.tar.gz，创建 GitHub Release 并上传所有 assets。
make release VERSION=v0.5.1

# 用户一行安装
curl -fsSL https://raw.githubusercontent.com/zhoushoujianwork/easyeda-agent/main/install.sh | sh
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhoushoujianwork/easyeda-agent](https://github.com/zhoushoujianwork/easyeda-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
