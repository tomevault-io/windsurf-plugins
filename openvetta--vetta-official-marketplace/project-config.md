---
trigger: always_on
description: 本仓库是 Vetta 桌面端「开放能力市场」的官方源。源码在 `main` 开发，由 CI 向 gh-pages 发布 schema v3 索引、向 GitHub Releases 发布固定 .vettapkg。Desktop 读取 `gh-pages`。
---

# 能力编写手册

本仓库是 Vetta 桌面端「开放能力市场」的官方源。源码在 `main` 开发，由 CI 向 gh-pages 发布 schema v3 索引、向 GitHub Releases 发布固定 .vettapkg。Desktop 读取 `gh-pages`。

这份手册面向在本仓库中添加/修改能力的人与 AI。源码检查、发布检查与客户端校验分别保护不同边界；来源同步失败时查看主进程中的 open-marketplace 日志。

## 静态市场发布规则

本仓库采用 Helm 静态包仓库模式。`main` 是源码分支，事实源是 `.vetta/marketplace.source.json`；CI 在 `gh-pages` 生成 `.vetta/marketplace.json`。源码不保存制品索引，也不手工维护 marketplaceVersion。

- 修改前运行 git status，保留已有改动。没有用户授权不得提交或推送。
- 通过以 `main` 为基准分支的普通源码 PR 审核代码及能力版本。合入后 CI 发布新版本，不能自动合并源码 PR。
- 插件源码条目（含仅 Bundle 引用的成员）声明 minAppVersion；API、权限、命令与摘要由构建结果派生。
- 同版本运行内容继续使用已发布制品；准备发布时提升能力版本并同步相关身份文件。
- CI 先校验、上传并复核制品，再推进 gh-pages。每个插件使用固定的 `plugin-<slug>` Release，版本包只追加；已有版本不可覆盖。
- 首次联调尚未正式发布的 Desktop 版本时，`candidateAppCommits` 只能把该版本钉到 OpenVetta 的 40 位不可变 commit；稳定 Release 存在后门禁自动优先校验 Release。
- 文档和源码开发提交不增加市场版本。只有分发内容变化时 CI 分配新 marketplaceVersion。
- 不执行向 `main` 回写生成索引、先发包后提目录 PR 或逐提交版本递增的旧流程。
- 完成时依次运行 node scripts/marketplace.mjs check、node scripts/marketplace.mjs build 与 node --test tests/*.test.mjs；内容测试同时检查生成后的插件资源，不能放在构建前。Windows Python shim 环境可将 VETTA_PYTHON 指向真实解释器。
- 本地构建使用 node scripts/marketplace.mjs build；正式发布由 publish-marketplace.yml 执行。
- Desktop 使用分支 `gh-pages`。生成索引只写在 `gh-pages`。

## 添加一个能力的流程

1. 选类型：`skill` / `mcp` / `plugin` / `bundle`（没有别的类型，`scene` 不被支持）
2. 建包目录：`abilities/skills/<slug>/`、`abilities/mcp/<slug>/`、`abilities/plugins/<slug>/`、`abilities/bundles/<slug>/`（注意 mcp 目录没有复数 s）
3. 写包内文件（见「各类型包规范」）
4. 写展示层 `ability.json`（可选 `detail.json`、`assets/`）
5. 需要独立展示时在 `.vetta/marketplace.source.json` 的 `abilities[]` 注册；仅 bundle 成员则在 bundle 中写包路径引用
6. 准备发布运行内容时提升能力版本，普通源码 PR 审核通过后由 CI 自动发布
7. 执行源码与发布工具测试；用户要求提交时再核对暂存内容

插件项目的目录、职责拆分、Tailwind 接入和用户流程测试遵循
[`docs/plugin-project-structure.md`](docs/plugin-project-structure.md)。文件名必须表达职责；不要使用 `ui.ts`、`ui.tsx`、`utils.ts` 或 `primitives.tsx` 作为多个职责的容器。所有 UI `.tsx` 生产文件放在 feature/shared 的 `components/` 下（根目录 `index.tsx` 仅作为插件装配入口），基础组件原则上一个文件只放一个组件；`tools/` 仅用于 `ctx.agent.registerTool()` 的 Agent 工具。

## 开发插件：工具与手册

插件类能力的**开发单位是它自己的目录**（`abilities/plugins/<slug>/`），每个目录里有一份
`AGENTS.md` 交代该怎么开工。开发时先 `cd` 进去——所有工具命令都作用于「最近的那个 `plugin.json`」。

```bash
cd abilities/plugins/<slug> && npm install
npx vetta-plugin-cli docs          # 手册目录绝对路径 + 对应的 SDK 版本
npm run build                      # v3 本地预检可打包；正式 .vettapkg 由受保护 CI 构建，dist/ 不提交
```

SDK 手册随 `@vetta-org/plugin-sdk` 装进各插件自己的 `node_modules`，因此读到的合同与该插件
实际编译的版本一致。**不要硬编码 node_modules 路径**，用上面的命令解析。

> 现状：四个插件钉的都是 `^0.1.1` / `^0.2.0`，**早于手册随包发布的 0.3.1**，所以 `docs` 现在
> 都报找不到。升到 `^0.3.1` 才能用上，但那是跨 0.3.0 破坏性变更的升级，需要逐个插件评估。

新建插件工程（仓库根没有 `node_modules`，用全名）：

```bash
npx @vetta-org/plugin-cli init --id <slug> --name "<Display Name>" abilities/plugins/<slug>
```

它只创建目录，**不动索引**——什么时候上架是人的决定，按上面「添加一个能力的流程」登记。

### 索引生成与验证

源码声明不包含 marketplaceVersion 或 releases。通过 node scripts/marketplace.mjs check 校验；完整构建后的 .marketplace-build/site 由 CI 使用固定提交中的 Plugin CLI 源码对账（公开的 0.1.6 尚不支持此分发目录）。不要用旧 sync 改写源码配置。详见 [静态发布说明](docs/marketplace-v3.md)。

## 目录结构

```text
.vetta/marketplace.source.json
abilities/skills/<slug>/SKILL.md
abilities/mcp/<slug>/mcp.json
abilities/plugins/<slug>/plugin.json
abilities/bundles/<slug>/
abilities/<type>/<slug>/ability.json
abilities/<type>/<slug>/detail.json
abilities/<type>/<slug>/README.md
abilities/<type>/<slug>/assets/
```

## 源码配置

.vetta/marketplace.source.json 包含 schemaVersion: 3、name、displayName、repository、minAppVersion 与 abilities。
每个插件条目额外声明 minAppVersion，不填写 releases。普通条目字段和各类型身份合同如下。
源码版本可领先于 gh-pages 中已发布版本；每个能力版本独立发布。configVersion 只标识配置结构。

`abilities[]` 每一项：

```json
{
  "type": "skill",
  "slug": "hello-vetta",
  "name": "Hello Vetta",
  "description": "一句话说明这个能力做什么。",
  "version": "1.0.0",
  "configVersion": 1,
  "license": "MIT",
  "author": "Vetta",
  "category": "Examples",
  "categoryI18n": { "zh": "示例", "en": "Examples" },
  "tags": ["example"],
  "detail": { "i18n": { "zh": { "name": "…", "description": "…" } } },
  "source": { "path": "abilities/skills/hello-vetta" }
}
```

- `slug` 在整个解析后目录内**全局唯一**，不分类型；多个 bundle 可引用同一个类型和路径的成员
- 展示名称描述用途，不包含 `MCP` / `Skill` / `Plugin` / `Bundle` 或「技能 / 插件 / 套装」类型后缀；
  类型由客户端标签展示。改名只调整默认名称、语言覆盖及详情标题，不改变已有 slug、安装身份或配置版本。
- `source.path` 必须是仓库内相对路径，不能逃出市场根目录
- `configVersion` 在能力的配置契约变化时 +1
- `detail.i18n.<locale>` 用来放多语言的 `name` / `description`，目录页直接用
- `category` 是稳定分组标识，`categoryI18n` 是可选的语言键到显示名的字符串映射。官方能力必须补齐 `zh` / `en`，
  同一分类的译名保持一致；不要把 `category` 改成当前语言的译名。旧客户端忽略该可选字段，无须提高 `minAppVersion`。
- **`type: "mcp"` 的条目禁止出现 `config` 键**（安装配置只能放在包里的 `mcp.json`）；写了会直接报 `MCP configuration must be stored in source.path/mcp.json`

## 各类型包规范

### skill

包内必须有 `SKILL.md`，frontmatter 三个字段与 manifest 条目严格对齐：

```markdown
---
name: hello-vetta        # 必须 === 条目的 slug
description: …           # 必填，非空
version: 1.0.0           # 必须 === 条目的 version
---

正文就是发给 Agent 的指令。
```

整个包目录内不允许符号链接。

### mcp

包内必须有 `mcp.json`，schema 是 **strict** 的（多写任何键都会失败）：

```json
{
  "schemaVersion": 1,
  "slug": "context7",
  "version": "1.1.0",
  "server": { "type": "http", "url": "https://mcp.context7.com/mcp" },
  "parameters": [
    {
      "key": "CONTEXT7_API_KEY",
      "label": "Context7 API Key",
      "required": false,
      "secret": true,
      "placeholder": "sk-…",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openvetta/vetta-official-marketplace](https://github.com/openvetta/vetta-official-marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
