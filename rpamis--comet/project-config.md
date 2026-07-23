---
trigger: always_on
description: 验证范围必须与改动风险相匹配，不要在每次编辑后默认运行全量测试。
---

## 回复语言

必须采用中文回答用户

## 测试

验证范围必须与改动风险相匹配，不要在每次编辑后默认运行全量测试。

- 每轮先运行覆盖当前改动的最小相关测试。
- 纯文档或 Skill 内容修改：运行相关契约测试和受影响文件的 Prettier 检查。
- 单一 `app/`、`domains/` 或 `platform/` 模块修改：运行对应测试；涉及编译、Runtime 或生成物时再运行 build。
- 跨模块、Runtime、安装/路由、发布准备或其他高风险修改：最终交付前运行一次全量测试。
- 全量测试失败或超时时，先定位原因；只有修正了明确原因后才重跑，不盲目重复。
- CI 已覆盖全量检查时，可以在本地只运行相关验证，但交付时必须明确说明未在本地运行的检查。

```bash
npx vitest run <相关测试文件>                     # 默认：最小相关测试
npx vitest run                                   # 高风险修改或最终交付前的全量测试
```

## 提交前检查

仓库已配置 Git pre-commit 钩子（husky + lint-staged），每次 `git commit` 会自动对 `app/`、`domains/`、`platform/`、`scripts/`、`test/`、`.github/`、`config/` 下的暂存源文件运行 `prettier --write`（与 CI `format:check` 范围一致；冻结的 `test/fixtures/` 除外），编辑器无关，所有贡献者生效。

根据改动范围选择提交前检查；不要求每个提交机械地运行全部命令（CI 会执行完整检查）：

```bash
pnpm format:check   # 大范围格式检查；小范围可只检查受影响文件
pnpm lint           # 修改源码、测试、脚本或配置时
pnpm build          # 涉及编译、Runtime、生成物或发布资产时
pnpm test           # 高风险修改或最终交付前需要本地全量验证时
```

注：本地 Windows 若 `core.autocrlf=true`，未改动的旧文件可能因 CRLF 被 `prettier --check` 误报；钩子只处理暂存文件，不受影响，旧文件下次编辑时会自动转为 LF。

## Commit 规范

提交信息必须使用类型前缀，格式为 `<type>: <summary>`，scope 可选：`<type>(<scope>): <summary>`。

常用类型包括：`feat`、`fix`、`docs`、`chore`、`refactor`、`test`、`build`、`ci`、`perf`。

示例：

- `feat: add eval report language switch`
- `fix(eval): prevent chart labels from overlapping`
- `docs: update contributor commit rules`

## 项目结构规范

当前源码目录按责任分层：

- `app/`：CLI 入口、命令编排和用户交互层。只能组合 domain/platform 能力，不承载领域规则。
- `domains/`：业务领域模块。每个子目录是一个可独立维护的领域模块，例如 `domains/bundle/`、`domains/comet-classic/`、`domains/comet-native/`、`domains/comet-entry/`、`domains/dashboard/`、`domains/skill/`、`domains/workflow-contract/`。
- `platform/`：文件系统、进程、安装平台、版本、路径等平台适配能力。domain 不应直接散落平台差异逻辑。
- `scripts/`：构建、发布、benchmark、lint 等仓库自动化脚本。可调用源码模块，但不要成为运行时业务入口。
- `assets/`：发布资产和内置 Skill 内容。修改 runtime 源码后必须通过构建同步生成资产，不要把业务逻辑只写在生成物里。

测试目录必须跟随被测对象归属：

- `test/app/` 覆盖 `app/` 命令和 CLI 行为。
- `test/domains/<domain>/` 覆盖对应 `domains/<domain>/` 模块；新增 domain 时同步新增同名测试目录。
- `test/platform/` 覆盖 `platform/` 适配层。
- `test/scripts/` 覆盖 `scripts/` 自动化脚本。
- `test/repository/` 覆盖 README、CI、仓库布局等跨层约束。
- `test/fixtures/` 和 `test/helpers/` 只放测试数据与测试工具。
- 禁止新增或恢复 `test/ts/` 这种横向桶；旧文件应迁移到上面对应目录。

架构约束由 `pnpm run lint:architecture` 校验，并已接入 `pnpm lint`。它会检查顶层目录白名单、活跃源码根、app/domain/platform 子模块、脚本模块、Classic/Native/Entry runtime 入口与生成物、内置 Skill 根目录、测试归属和禁止旧目录回归。如果确实需要新增顶层目录、源码模块、测试根目录或例外，必须先更新 `config/repository-layout.json`、架构 linter 和本节说明。

## Workflow runtime 与 Hook 路由规范

脚本是 **Node.js 启动器或生成 bundle（`.mjs`）**，只依赖 Node.js，**不依赖 bash / Git Bash / WSL**。

- Classic 真实逻辑位于 `domains/comet-classic/`，由 `pnpm build:classic-runtime` 生成 `assets/skills/comet/scripts/comet-runtime.mjs`；同目录其他 Classic launcher 保持薄封装。
- Native 真实逻辑位于 `domains/comet-native/`，由 `pnpm build:native-runtime` 生成 `assets/skills/comet-native/scripts/comet-native-runtime.mjs`；Native 主流程和 Guard 不得依赖外部 Skill。
- 共享入口、selection 与 Hook Router 位于 `domains/comet-entry/`，由 `pnpm build:entry-runtime` 生成 `comet-entry-runtime.mjs` 与 `comet-hook-router.mjs`。
- 每个平台只安装一份 `comet-workflow-guard` Rule；支持 Hook 的平台只安装一个 `comet-hook-router.mjs`。Router 根据 `.comet/current-change.json` 一次只调用当前 Native 或 Classic Guard，两边的阶段、目录、schema 与 Guard 逻辑保持独立。
- `comet-hook-guard.mjs` 与 `comet-native-hook-guard.mjs` 是各自 runtime 的薄 Guard launcher，不直接作为平台 Hook 安装。
- `comet-env.mjs` 打印自身所在目录（scripts dir），供 skill 样板代码解析同级启动器路径。
- 跨平台由 Node 保证：hash 用 `node:crypto`，YAML 用 `yaml` 包，子进程用 `child_process`（构建/校验命令走 `spawnSync(cmd, { shell: true })`）。不再有 `sed -i` / `sha256sum` vs `shasum` / `pipefail` 等 shell 可移植性问题。
- 新增/重命名 runtime 入口或生成物必须同步 `assets/manifest.json`、`config/repository-layout.json` 的对应 runtime 映射与 `test/repository/*-runtime-assets.test.ts`；Classic launcher 还需同步 `test/domains/comet-classic/comet-scripts.test.ts` 的 fixture 列表。
- skill 样板（boilerplate，当前版本 `v3`）在所有 SKILL.md / reference 中重复，改动需全量同步；样板通过 `find` 定位 `comet-env.mjs`，再用 `node "$COMET_ENV"` 解析路径，命令统一为 `node "$COMET_STATE" ...` 形式。

## 脚本依赖关系

```
comet-runtime.mjs        ← domains/comet-classic/*
comet-native-runtime.mjs ← domains/comet-native/*
comet-entry-runtime.mjs  ← domains/comet-entry/*
comet-hook-router.mjs    ← 平台唯一 Hook 入口 → 当前 selection 的一个 workflow Guard
```

Classic 打包入口 `domains/comet-classic/classic-cli.ts` 导出 `main` / `runClassicCli` / `CLASSIC_COMMANDS`；Native 与 Entry 的入口由 `config/repository-layout.json` 分别声明。esbuild ESM bundle 保留所需 export，启动器直接 import 调用，单进程、无 bash、无二次 node 派生。跨 workflow 的稳定契约放在 `domains/workflow-contract/`，入口归属与路由放在 `domains/comet-entry/`；不要为了复用而合并 Native 与 Classic 的状态机或 Guard。

## .comet.yaml 状态机

每个 change 的状态文件，字段变更需要同步三处（全在 TypeScript 中）：

1. `domains/comet-classic/classic-state-command.ts` — `set` 白名单 + enum 验证（`SETTABLE_FIELDS` / `MACHINE_OWNED_FIELDS`）
2. `domains/comet-classic/classic-validate-command.ts` — schema 校验 + 已知字段
3. `test/domains/comet-classic/comet-scripts.test.ts` — 测试中的 yaml 字符串

改完 1/2 后 `pnpm build` 重新生成 `comet-runtime.mjs`，否则 `classic-runtime.test.ts` 的新鲜度检查会失败。

## 双语言 Skill

skill 优化时先写中文版本（`assets/skills-zh/`），用户确认后再修改英文版本（`assets/skills/`）。

## 中文术语翻译规范

中文文档不得把英文 “gate” 直译为“门”（如“压缩门”“调试门”“确认门”），这种译法在中文语境下不自然。应按实际含义翻译：

- `gate`（阶段性检查/阻塞点）→ 根据语境用“协议”“阶段”“检查”“阻塞点”等，如 `debug gate` → “异常调试协议”
- 修饰词性质的 `proactive/active` → “主动式”，如 `proactive context compression` → “主动式上下文压缩”，不写作“主动压缩门”

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rpamis/comet](https://github.com/rpamis/comet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
