---
trigger: always_on
description: `dsh-diagram` 是独立发布的 DeepSeek Harness Web 插件。它把已经进入 DSH Session 的文章内容转换为可编辑的 Excalidraw 画布，并通过插件自有 sidecar 持久化。插件仓库、npm 包和 DSH 源码必须保持分离。
---

# AGENTS.md

`dsh-diagram` 是独立发布的 DeepSeek Harness Web 插件。它把已经进入 DSH Session 的文章内容转换为可编辑的 Excalidraw 画布，并通过插件自有 sidecar 持久化。插件仓库、npm 包和 DSH 源码必须保持分离。

本文件是开发代理的维护入口。`CLAUDE.md` 必须保持为指向本文件的软链。产品现状见 [README.md](README.md) 和 [README.zh-CN.md](README.zh-CN.md)，设计决策见 [DESIGN.md](DESIGN.md)，sidecar 的长期理由见 [.agents/notes/implemented/architecture/2026-08-14-diagram-canvas-sidecar.md](.agents/notes/implemented/architecture/2026-08-14-diagram-canvas-sidecar.md)。

## 发布身份与仓库

- GitHub 仓库：`https://github.com/hanzhangzzz/dsh-diagram`。
- npm 包：`dsh-diagram`。
- GitHub commit 的 author 和 committer 必须是 `huajuan404 <258429709+huajuan404@users.noreply.github.com>`。
- 推送或创建 Release 前运行 `gh api user --jq '{login,id}'`，只有返回 `huajuan404` 和 `258429709` 才能继续。
- npm 发布前运行 `npm whoami`；当前公开包 maintainer 是 `hanzhangz`。账号不匹配时停止，不要尝试换账号或改变 package ownership。
- 不得 force-push 已发布分支或移动已发布 tag。版本有问题时发布新 patch 版本。

## 开始开发

新环境只需要本仓库和公开 npm 依赖，不依赖相邻的 DSH 源码 checkout：

```sh
git clone https://github.com/hanzhangzzz/dsh-diagram.git
cd dsh-diagram
pnpm --version
pnpm install --frozen-lockfile
pnpm run typecheck
pnpm run test
```

要求 Node.js `^22.19.0 || >=24.0.0`，并确保 pnpm `>=10` 是 `PATH` 中可直接执行的二进制。不要假设 Corepack 在所有 Node 安装中都可用；先验证 `pnpm --version`。开发依赖使用公开的 DSH 版本，不要把 `package.json` 改回本机 `link:` 路径。

开始修改前：

1. 读取本文件、README、DESIGN 和与目标模块对应的测试。
2. 运行 `git status --short --branch`，保留不属于当前任务的改动。
3. 如果需要参考 DSH 源码，先记录其工作树状态；把它当只读依赖，任务结束时复核状态未变化。
4. 先写或更新能复现问题的 focused test，再改实现。

## 仓库分层

- `src/core/`：唯一的 `DiagramSpec`、scene、record 和 RPC 数据定义，以及确定性布局。
- `src/host/`：Session 归属、storage-domain、CAS、工具、HTTP RPC 和 editor 静态资源。
- `src/client/`：轻量 DSH Client 插件，只注册会话“画布”标签并挂载 iframe。
- `src/editor/`：Vite 构建的 Excalidraw 编辑器、RPC adapter、自动保存、草稿恢复和导出。
- `build/`：bundle、字体、第三方许可和真实安装 smoke 的生成器。
- `cordis.patch.yml`：保留在已安装包内，由 DSH 根据 `package.json.dsh.bundle.patch` 在组合 profile 时读取。`dsh plugin add` 只更新 profile 的 package dependency 和 `dsh.profile.bundles`；不要把它与 profile 自己的 patch 文件混为一谈。
- `lib/`：构建产物，始终由生成器重建，不直接编辑，也不提交。
- `tests/`：按上述分层覆盖协议、安全、生命周期、编辑器和发布安装。

跨 Host 与 editor 使用的数据只在 `src/core/` 定义。不要在两端复制 endpoint、错误联合、scene 字段或限制常量。

## 产品与数据不变量

- Agent 只生成紧凑的 `DiagramSpec`；确定性布局把它转换成初始 Excalidraw scene。不要让模型直接生成完整 Excalidraw JSON。
- `DiagramSpec.edges` 是 `from -> to` 的有向边。节点、分组 id 必须唯一，边端点和 group 引用必须存在。
- scene 一旦产生就是用户当前文档；`sourceSpec` 只记录生成来源。重新布局或读取不得覆盖用户编辑后的 scene。
- 每条记录同时保存 `sessionId` 和 `{createdAt, cwd}`。只比较 Session id 会在 id 重用后泄漏旧生命周期的数据。
- 未归属当前 Session lifecycle 的 diagram 必须与不存在相同地返回 `diagram-not-found`。
- 保存使用整 scene compare-and-set：必须先校验 `expectedRevision`。只有 scene 实质变化才生成新的不透明 UUID revision；相同 scene 返回 `unchanged: true` 并保留 revision，过期 revision 即使内容相同也必须冲突。冲突返回 Host 当前记录，绝不静默覆盖。
- 自动保存不写 Session log。只有显式调用 `diagram_read` 后，当前 scene 的受限摘要才通过普通 Tool Result 进入模型上下文。
- Session fork/export 不复制 sidecar；卸载插件不删除 sidecar。改变这些行为属于新的产品和数据迁移设计。
- 所有部署相关限制必须由 `DiagramConfig` 和 `cordis.patch.yml` 显式给出，不能在调用路径中藏默认值。
- 总记录数、单 Session 数和 canonical UTF-8 总字节预算必须共同生效。不要只限制每个 scene 而忽略持久化总量。

## Host、生命周期与 DSH 集成

- 普通插件工作不得修改 DeepSeek Harness 源码。发现 DSH API 缺口时，先记录上游需求；插件仍以当前公开 DSH 版本为发布目标。
- 当前支持 DSH `0.1.0-rc.6`。升级时同时更新 peerDependencies、devDependencies、README 徽章和兼容表、`build/smoke-dsh-install.mjs` 中的版本及真实安装测试。
- 不要对 DSH Service class 使用跨包 `instanceof`。DSH 的 source launch 和已构建 npm artifact 可能加载同一 class 的两个模块实例，导致合法 service 被误判。依赖 Cordis `static inject` 等待服务，再通过 `ctx.get("serviceKey")` 取得结构化接口。
- 注册必须跟随 Cordis 生命周期。WebServer route 用 `ctx.effect()` 包装 disposer；`Tools.register()` 自己已经注册 effect，不要重复包装成嵌套 effect。
- 初始化顺序是：验证物理 bind、取得依赖、打开 storage-domain、注册关闭 effect、注册静态资源和 RPC、注册工具。失败时不得留下半注册入口。
- 插件只允许 `webServer.host === "127.0.0.1"`。不要把 `authority: loopback` 或 Host header 检查误当作 socket 级本机认证，也不要在没有完整鉴权设计时开放 `0.0.0.0`。
- Session 查询不得创建、恢复或改变 Session。当前 DSH persistence 缺少 typed absent lookup，因此 cold lookup 先读 snapshot catalog 再 inspect；不要通过匹配异常文本推断 not-found。
- catalog/inspect 是异步的，前后都要复核 live Session，防止查询期间 lifecycle 被替换。`diagram_create`/`diagram_read` 的 Session 只来自 `exec.agent.session.header`，不能接受模型提供的 session id。
- repository 对同一 diagram 串行化保存，对创建和全局字节预算分别串行化；dispose 先关闭新写入，再等待已接纳写入完成。

## RPC 与静态资源安全

- `/diagram` 必须继续使用 `src/host/http-rpc.ts` 的专用有界 HTTP carrier。不要换回 `ctx.connection.rpc.handle()`：通用 bridge 会在业务 schema 之前缓冲远大于 1 MiB 的请求。
- HTTP body 上限是 `maxSceneBytes + 16 KiB` envelope allowance。`Content-Length` 和实际流入字节都要在 `JSON.parse` 前检查，超限返回 `413` 并关闭请求。
- 保持 Host、Origin、`Sec-Fetch-Site`、method、media type、URL endpoint 和 message method 的一致性检查。重复 Host、userinfo、path、跨站 Origin 和超长 `rpcId` 必须拒绝。
- Wire 外层使用 DSH 的标准 `client-request` / `server-response` envelope；list/get/save 的业务结果使用 `src/core/rpc.ts` 的严格联合。预期业务失败是返回值，不要 throw 成不透明 prose。
- Host 既校验请求，也校验业务实现返回值；Client 校验 envelope、响应 `rpcId` 和严格业务结果。Client 必须先通过 `list` 取得 Host 的 validation policy，再用同一 policy 解析 get/save。transport failure 与业务失败不得折叠成同一状态。
- 意外异常只写 Host logger；响应不能反射内部异常、文件路径或 storage 细节。
- `/diagram-assets` 只允许 `GET`/`HEAD`、MIME 白名单和 canonical realpath 内的文件。入口 `no-store`，带内容哈希的资源 immutable；CSP、same-origin、no-sniff 和 frame-ancestors 不能被弱化。
- iframe 只是加载边界，不是安全隔离边界。其安全性依赖同源、CSP、静态路径限制、RPC admission、scene 校验和 Session ownership 的组合。

## Scene 校验


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hanzhangzzz/dsh-diagram](https://github.com/hanzhangzzz/dsh-diagram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
