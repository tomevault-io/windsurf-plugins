---
trigger: always_on
description: 本文件适用于整个仓库。修改前先阅读根 `README.md`、目标目录的 README 和相关设计文档。
---

# Univer Workspace 仓库指南

本文件适用于整个仓库。修改前先阅读根 `README.md`、目标目录的 README 和相关设计文档。
`apps/workspace/AGENTS.md` 对 `apps/workspace/**` 提供更具体的数据库与部署约束。
`apps/cli/AGENTS.md` 对 `apps/cli/**` 提供 CLI 与 Client Core 的职责边界，以及从用户 README
移出的 Skills、渲染副本、PDF 打印和运行时许可证约束。两者同时适用，冲突时以更具体且更严格的规则为准。

## 项目目标

本仓库拥有 Univer Workspace 产品及其配套 CLI。它将 Univer、Univer Pro、Univer Collaboration
SDK 和 Univer CLI SDK 组装成两个对外应用：

- Univer Workspace：可部署的 Browser、产品 HTTP API、协同入口和后台任务。
- Univer Workspace CLI：面向 Agent 的远程 Workspace 自动化应用。

本仓库还包含供 Browser 或 Node-hosted Workspace Agent Client 使用的 private packages。它们是内部
实现，不是额外的对外应用或跨仓库公共 SDK。

## 仓库结构与职责

```text
apps/workspace                 Workspace Browser、Server、HTTP contract 与部署应用
apps/cli                       Univer Workspace CLI
apps/agent                   Univer Workspace Agent（DSH 定制服务 + 两个预装插件）
packages/client-core           Node-hosted Workspace Agent Client 共享能力
packages/reference-provider   Browser 专用的 private referenced-Unit policy
packages/dsh-univer-workspace-plugin        能力插件（远程 Unit 工具集 + headless runtime）
packages/dsh-univer-workspace-skin-plugin   皮肤插件（workspace 品牌与 logo）
scripts                       仓库级 SDK 版本与 CLI 本地开发脚本
```

- `apps/workspace` 拥有 Workspace 产品模型，包括 Identity、Space、Node、Resource、ACL、Trash、
  Recent、Blob、Asset、Operation 和 Worktree 的产品级组织。
- `apps/cli` 拥有 Workspace origin、登录 Session、Commander composition 和面向 Agent 的 CLI 交付体验。
- `packages/client-core` 拥有 Node-hosted Workspace Agent Client 共享的 HTTP、错误、storage-neutral 认证协议、
  远程产品 workflow 与 worker-backed content runtime；Client Shell 注入 origin、凭据、license 与 packaged
  worker entry，package 不读取 CLI Session 或配置。
- `apps/agent` 拥有 Workspace Agent 服务：DSH（DeepSeek Harness）定制组装，以 Workspace OAuth
  授权用户身份提供 agent 操作远程 Workspace 文档（Unit）的能力。它预装两个仓库内开发的插件，
  本身不发布为公共 SDK。
- `packages/reference-provider` 只服务于 Workspace Browser。CLI 在自身 application 内维护独立
  Provider；两者共享 persisted identity 和行为语义，但不为消除代码重复而制造跨应用公共合同。
- `packages/dsh-univer-workspace-plugin` 提供 agent 操作远程 Workspace 文档的能力（工具集、空间对账、
  headless 协同 runtime、导入导出）。代码与维护独立，通过 harness profile 预装，不发布到 npm。
- `packages/dsh-univer-workspace-skin-plugin` 只提供 DSH 浏览器面的 workspace 外观（主题令牌与品牌）。
- `apps/*` 可以组合 SDK 能力；private packages 不得反向依赖 application。`packages/dsh-univer-workspace-plugin`
  依赖 `apps/agent` 暴露的 `workspaceAuth`/`workspaceSession` 服务，通过 cordis 服务组合而非 npm 依赖。

## SDK 与仓库边界

本仓库是 product/application composition root，不重新拥有上游 SDK 的合同：

- Univer / Univer Pro SDK 拥有 Unit 数据模型、Facade API、mutation、render、Office exchange 和内容能力。
- Univer Collaboration SDK 拥有 snapshot、changeset、revision、OT、协同 Service、Worktree、
  Database Adapter、Endpoint 和 Transport 合同。
- Univer CLI SDK 拥有 target-neutral 的 headless runtime、execution、inspection、render、daemon 和可选
  Commander preset。
- Workspace 产品模型、认证、资源目录、远程 workflow 和 deployment 留在本仓库。

只通过已发布 package 的公开 exports 使用其他 SDK。代码、构建、测试和生成流程不得依赖相邻仓库
checkout、其他仓库的绝对路径或未发布源码目录。

`apps/agent` 与两个 dsh 插件包通过公开 npm 的 `@deepseek-ai/*` 包使用 DSH（dsh 是外部产品，本仓库
不 fork、不修改其源码）。dsh CLI 运行时图由 `packages/dsh-runtime` 声明——它是独立嵌套 workspace
（`packages/*` glob 显式排除），自带 committed `pnpm-lock.yaml`，以精确 pin 锁定完整闭包；桌面产物
与 agent 镜像在构建时先 `pnpm install --frozen-lockfile` 再 `pnpm deploy --prod
--config.node-linker=hoisted` 把它物化为 workspace 安装之外的自包含目录，不在构建时浮动解析。
该闭包不得并入根 workspace 图：共享解析域会重解析消费者 optional peers，使同一 dsh 包产生多个
实例并分裂跨包品牌类型（SessionId、Context）。物化同时使 dsh client 的 react 18 树与 Univer SDK
的 react 19 图保持分离，不分裂 `@wendellhu/redi` 实例。

所有 version-coupled `@univer-cli/*`、`@univerjs/*` 和 `@univerjs-pro/*` 依赖使用同一个精确
SDK release。升级时运行：

```bash
pnpm update:univer-sdk --sdk_version <exact-sdk-version>
```

`@univerjs/icons`、`@univerjs-pro/cli-assets` 和 `@univerjs-pro/doc-typst-native-binding` 按自身
发布节奏独立发版：声明保持精确版本，不跟随 SDK baseline。原生绑定
`@univerjs-pro/engine-formula-rust-binding` 和 `@univerjs-pro/exchange-node-binding` 由 wrapper 包
`@univerjs-pro/engine-formula-rust`、`@univerjs-pro/exchange-node` 声明，manifest 只声明 wrapper；
CLI 打包脚本从 wrapper manifest 读取绑定版本写入 artifact 运行时依赖。`pnpm-workspace.yaml` 的
`overrides` 只承载 dev 版本探索与 insiders 修复回移：顶层 SDK 条目必须是 dev 或 insiders 版本，
`父包@版本>子包` 形式的 scoped 条目只约束一条边，可持任意版本形式。升级时清空其中的全部 SDK 条目。

必须同时提交所有受影响的 manifest 和 `pnpm-lock.yaml`，不得手工只更新其中一部分。

## 标准能力与临时代码

- 优先在应用边界直接组合上游公开能力；只有承担了新的职责或生命周期时才增加抽象，不为简单转发制造
  包装层，也不在本仓库复制上游合同。
- 历史数据兼容、迁移和补偿逻辑必须与常态业务路径隔离，集中在单一入口和明确的生命周期阶段执行；实现
  应幂等，并说明适用范围、失败语义和退出条件。
- Workaround 必须集中隔离，避免临时分支散落到正常代码中；注明原因、影响范围和删除条件，并在上游
  问题消失时及时移除。

## Workspace 数据与运行边界

- 产品数据与 Univer 协同数据分别存储。产品数据库不得保存 snapshot、changeset 或 revision；
  Collaboration Database Adapter 不拥有 Space、Node、Resource 或 ACL 产品模型。
- Blob 与内嵌 Univer Asset 的字节由 `BlobStore` 保存，产品数据库只保存身份、元数据和恢复状态。
- 产品数据库、Collaboration Service 与 BlobStore 之间不存在伪造的跨系统事务。跨边界写入使用
  持久化 Operation、idempotency 和 recovery 明确收敛。
- Browser 和 CLI 都不能信任客户端提供的 User、Role、Resource、Unit、Worktree 或 confirmed
  revision；服务端从认证 Session 与产品数据解析权威身份和权限。
- `apps/workspace/**` 的 schema、迁移、备份、升级和部署操作必须遵守
  `apps/workspace/AGENTS.md`。不得把 `db:reset` 用于正常启动、升级或生产恢复。

## HTTP contract 与生成物

- `apps/workspace/contracts/http` 是产品 HTTP contract 的源码。
- `apps/workspace/generated/http` 由 Redocly 和 `openapi-typescript` 生成，不得手工修改。
- Express 路由实现、OpenAPI 源文件、生成类型和调用方必须描述同一行为。
- 修改 HTTP contract 时运行 `pnpm --filter @univerjs/univer-workspace api:verify`，并同步更新受影响
  的 Server、Browser、CLI、测试和文档。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dream-num/univer-workspace](https://github.com/dream-num/univer-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
