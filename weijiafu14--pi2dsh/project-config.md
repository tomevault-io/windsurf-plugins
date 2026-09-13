---
trigger: always_on
description: generateSummary——Pi 官方 streamFn 注入口接 DSH llm 桥）。
---

# CLAUDE.md — pi2dsh 工作准则

pi2dsh：通用 Pi Host ABI 兼容层，让 Pi 生态插件原样跑在 DeepSeek Harness
(DSH) 上。本文件是本仓库一切工作的标准，**任何新 session 开工前先读完**。
违反任何一条 = 返工。每条标准背后的真实事故记录在
[docs/STANDARDS.md](docs/STANDARDS.md)——改标准前先读事故。

**总纲一句话：对用户，一切是 DSH；对插件，一切是 Pi；中间层是唯一的
翻译官，且能借 DSH 官方的力就绝不自己造。**

## 一、用户安装使用标准

用户只用 DSH 官方命令，装一次引擎，之后装谁用谁，全程没有我们发明的步骤。

- 姿势：`dsh plugin add pi2dsh` 装一次引擎；之后 `dsh plugin add
  <Pi插件原包>` 直装 npm 原包。零转换、零生成产物；装了才挂、卸了就没。
  **代码里没有第二条路**：convert/host 命令、generateBundle/generateHostBundle
  导出、src/generator.ts 已整体删除（2026-08-16）。留着它们的真实代价不是
  "多一个特例"，而是**验证会架在错的路上** —— 主集成测试和真机端到端都曾
  在装转换产物，跑得再绿也证明不了用户那条路。开发和测试必须同一条路。
- **一份引擎实例挂所有插件**：一个模型目录、一个 /login、一个凭证存储、
  一个升级单元。禁止多份桥拷贝各自为政（事故：/login-2 自撞）。host 级
  资源（provider 目录/catalog/伴生映射/登录/凭证存储）经 SharedHostState
  跨包单份共享；包级资源（tools/commands/events）各归各。**零个社区 Pi 包
  也必须挂 host 级运行时**：内建 OAuth provider、`/login`、凭证恢复和伴生路由
  属于引擎，不得因插件发现结果为空而跳过。零包 profile 还是**最快挂载路径**，
  对组合服务的挂载期立即探测在这里必挂（2026-08-30 事故：凭证恢复用
  `optionalService` 立即探测 credentials，零包时服务未组合、存量登录路由全丢
  MISSING_CREDENTIAL，装任意包就"碰巧"好——对组合服务一律 `ctx.inject` 等到位，
  且零包 profile 是必测回归形状，契约测试在 tests/dsh-runtime.spec.ts）。
- 升级解耦：升引擎不动插件、升插件不动引擎；lockfile 锁死，只有显式
  `add <pkg>@latest` 才动；`pi2dsh inspect <pkg>@<版本>` 是升级预检门。
- 发现机制 = 读 profile 依赖清单（每项都是用户显式 add 的）+ Pi 官方
  `pi` 字段/目录约定判定包身份；**绝不扫 node_modules**（Prettier 3 弃用
  目录扫描的公开教训）。config `packages`/`exclude` 显式收窄兜底。
- **引擎自身依赖必须干净**：不带任何安装脚本（pnpm 对传递依赖的安装脚本
  报错性拦截，用户第一条命令就会炸——事故：pi-ai→genai→protobufjs）、
  不拖 CLI-only 大件（事故：typescript 23MB 白下载；现为 optional peer +
  懒加载分包，改依赖后必须验证引擎 chunk 的加载路径）。
- 撞上宿主安全门（pnpm 构建脚本审批）**不绕**：那是用户拍板的权利。文档
  写清应对即可（allowBuilds 设 true / approve-builds）。
- 加/卸插件后要重启 dsh（挂载在启动时）；先卸插件再卸引擎；伴生路由等
  引擎配置是 per-profile 的，每个用到的 profile 配一份。

## 二、Pi 插件处理标准（用户面界线，铁律）

**插件说 Pi 话，用户说 DSH 话，中间层负责翻译——用户面前永远没有 Pi。**

- 用户接触面——要动手写的配置、要看的文档教程、要敲的命令、报错里的
  指引——**一律 DSH 形状、DSH 官方机制**：配模型 = DSH settings 的
  `llm-pi-ai:` 段；配伴生路由 = 引擎的 cordis 插件 config
  （cordis.patch.yml）；凭证 = DSH credentials 引用（apiKeyEnv）。
- Pi 形状只允许活在两处：**插件视野**（shim/投影/事件）与**中间层内部
  实现**（vendored 源码、内部存储如 auth.json）。
- 判据：**用户需要亲手读写的东西里出现 Pi 词汇/格式 = 泄漏 = 返工**
  （事故：models.json 作为"Pi 标准配置入口"被搬进 DSH 用户世界，教 DSH
  用户写 Pi 格式文件，最终全链删除）。
- 我们兼容的对象是**插件代码**，不是把 Pi 生态的用户习惯搬给 DSH 用户。
  "Pi 教程照搬可用"不是目标，是泄漏。
- **插件自身配置的标准**（用户怎么配好一个 Pi 插件）分三层：
  1. **环境变量**（主路径）：Pi 插件生态的主流配置面是 env
     （VISION_BRIDGE_*/PI_VISION_*），env 是宿主中立的——DSH 用户设
     env 是纯 DSH 动作，零泄漏。examples 教这条。
  2. **插件自带斜杠命令**：插件用命令管理自己的配置（/vision），命令经
     中间层进 DSH 命令面板——用户敲的是 DSH 面板里的命令。
  3. **插件内部落盘**：插件以为在写 Pi config 目录，实际被重定向到
     `$DSH_HOME/pi2dsh/` 内部目录——文件在，但**不是用户接触面**，任何
     文档都不教用户碰它（auth.json 同理）。
  判据：用户给插件配置的动作只有"设 env、敲插件命令"两种；**任何"教用户
  手工编辑 Pi 格式文件"的路径都不存在**。若未来出现只认手工配置文件的
  插件（top50 无此形态），标准处置=引擎 config 加 per-package 的 DSH
  形状配置槽由中间层翻译落盘——出现第一个消费者时按此补，不预制。
- **Pi 扩展工厂没有参数位**（`ExtensionFactory = (pi) => void`，
  ../pi types.ts 实锤）：Pi 官方不存在"装插件给插件传参数"的通道，插件
  配置一律由插件自己定义来源（环境变量是事实标准，如 VISION_BRIDGE_*）。
  applyPiPackage 的 options.config 只喂中间层自己（visionCompanions），
  永远进不了插件视野。**别为这个不存在的通道发明 per-package 透传**；
  若上游 Pi 某天给工厂加了 config 参数，再按 DSH 惯例（管理者插件
  config 按名嵌套，llm-pi-ai providers 同款）一步接上。

## 三、中间层开发标准

**三层零跨层。跨层 = 不一致性 = 返工。**

```text
第 1 层  Pi 插件（原样源码，零修改）
         它需要的一切只来自中间层：三包 import 被 jiti alias 截获到 compat
         shim；registerX/事件/ctx 面全是中间层投影；插件视野里的数据面
         与 API 形状 100% Pi 词汇，永远不出现 DSH 概念（连字符串都不行，
         宿主托管路由的 api 用 Pi 官方词 'faux'）。唯一豁免：
         PiCapabilityError 等能力缺口报错文案——第三节要求它讲清 DSH 侧
         动作（如 dsh plugin remove X），文案是给用户看的指引，不算数据
         面泄漏。
第 2 层  pi2dsh 中间层
         compat 三 shim / ExtensionAPI 收单 / Pi 元数据账本 / registry
         投影 / 事件桥 / 凭证 / 会话与子代理桥 / 伴生路由。以"普通
         cordis 插件"身份接 DSH。
第 3 层  DSH（不知道 Pi 存在）
         看到的只是普通插件与普通 llm adapter。
```

- **DSH 已有官方能力，一律"配置翻译 + 官方实现"，禁止自建平行运行时/
  传输/第二套配置入口——动手前先查 DSH 官方有什么**（事故：官方
  llm-pi-ai 就在默认组合里、任意 OpenAI 兼容网关本是纯配置，却先背
  pi-ai 全家桶再自写 wire client，两版全是重复建设，全部删除）。已知的
  官方件：llm-pi-ai（模型网关）、dsh-mcp-client（配置型 MCP server）、
  dsh-skill-filesystem（skills）、settings/credentials seam。
- **“配置”与“能力包”分开判。** 只有 MCP server 定义时翻译给官方
  dsh-mcp-client；用户显式安装的 Pi MCP 能力包若自己拥有管理面、lazy
  proxy、OAuth、resources/prompts、transport/cache，则保留原包运行时，
  中间层只投影它使用的公开 Host ABI，绝不复制 transport。宿主同名命令
  优先，外来命令用有来源含义的别名（dsh-TUI 原生 `/mcp` 不动，Pi 包
  入口为 `/pi-mcp`）。这条是通用能力包边界，不许写包名业务特判。
- **能力包完整性按归属拆证据。** 会被 Host ABI 改变的生命周期、命令、TUI、
  动态工具、问答、附件、模型回调与取消必须用真实包穿过真实 DSH runtime 做
  E2E；包内部 transport/OAuth/cache/protocol 用同版本上游完整套件与 conformance
  证明。两边版本、结果、已知降级必须落一张矩阵，禁止再用一条 echo 宣称“完整
  支持”。范本：`docs/mcp-compatibility.md`。
- **Pi 终端面只桥公开服务。** `ui.custom`/`setStatus` 只在组合存在 dsh-TUI
  `tuiScenes`/`tuiStatus` 时投影；否则保持 headless。实现按能力工作，不按包名工作；
  不复制插件自己的管理状态或业务逻辑。
- 单一目录、单一调用路径：运行时模型目录只有 DSH llm 目录，Pi registry
  是其精确投影（包注册路由出口 restore 完整 Pi 形状——账本是中间层
  本职）；插件一切标准模型调用（registry.complete、getProvider().stream、
  pi-ai 顶层 complete/stream、createAgentSession）必经中间层转给 DSH llm
  路由；插件面永远拿不到直连传输；wire 层只属于路由供应商内部。
- **Pi 运行时挂载唯一路径：每个 root Agent 一份，全 surface 无条件一致
  （2026-08-21 拍板）**。作用域与时序拆开各用官方机制拿：挂载由
  `agent/created`（所有发布路径必触发、loop 启动前 emit）驱动进公开契约
  `agent.ctx`（agent-local、dispose 自动 unwind；DSH 自家 schedule 插件
  就是这个模式）；首轮正确性由官方 awaited waterfall 收口——
  `system-prompt/assemble` gate 等本 agent 挂载完成并用官方
  `tools.schemas(agent)` 补齐 waterfall 前已快照的 `assembly.tools`，

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weijiafu14/pi2dsh](https://github.com/weijiafu14/pi2dsh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
