---
trigger: always_on
description: 产品说明见 [LOONGPORT.md](LOONGPORT.md)（那份给人看：它替用户做什么、六条硬约束、怎么打包）。
---

# LoongPort 代码仓规则

产品说明见 [LOONGPORT.md](LOONGPORT.md)（那份给人看：它替用户做什么、六条硬约束、怎么打包）。
本文件只讲**写这个仓的代码时怎么决策**。

## 设计档案在另一个仓

设计文档、进度、spec 不在本仓，在同级的档案仓里（需要时用 `/add-dir` 单次挂载，别常驻）。
维护者本机的具体布局见工作区那份 `CLAUDE.md`（不入任何仓）。

**上一代实现是「参考不复用」**：它的 relay 层比现在这版复杂一个数量级（云同步边界、
多 app 展开、更多分支裁决），照搬会把这版简化的成果丢掉。查它的**结论**（实测记录、
某个设计为什么那样分）是对的，照抄它的**实现**是错的；那边带行号的引用基于旧的子模块
指针，引用前先 `grep -n` 复核。

## 一、最高优先级：能复用 cc-switch 的就复用

**这个仓是 cc-switch 的 fork，底层要跟着上游升级。** 所以「复用上游」不是风格偏好，
而是**决定未来升级成本的架构约束** —— 每一处自己另写的东西，都是将来 merge 上游时要手工
处理的冲突；每一处复用上游的东西，上游改进了我们免费拿到。

### 判定顺序（自上而下，命中即停）

1. **上游已有的组件 / hook / 工具函数 / 类型** → 直接用。
   例：折叠用 `src/components/ui/collapsible.tsx`（Radix 封装，已在仓里），
   不要引第三方折叠库、也不要自己写展开动画。
2. **上游已有的视觉 token**（间距、圆角、选中态、hover 效果）→ 抄它的值。
   判据：新页面和旧页面放一起，看不出是两个人写的。
3. **上游已有的模式**（数据流、命令命名、错误处理形状）→ 照它的形状写。
4. 以上都没有 → 才新建，且**新建的东西尽量收在自己的目录里**
   （`src/components/relay/`、`src-tauri/src/relay/`），别散进上游文件。

### 改上游文件时：改动面越小越好

不得不动上游文件时（如 `App.tsx` 的视图分流、`ProviderList` 加一层过滤），
**只改必须改的那几行**，把逻辑放进自己的新文件里让它调用。

反例：为了实现一个功能把上游某个 600 行组件重构一遍 —— 那等于放弃了那个文件的上游升级。

### 什么时候可以不复用

- 上游那套**语义上不适用**：例 `ProviderCard` 服务的是「用户手工配置的 provider」
  （可编辑、可删除、可拖拽排序），而 LoongPort 的托管项没有这些操作 —— 硬塞进去会让
  两种形态互相污染。这时另建组件是对的，但**视觉 token 仍要抄**。
- 上游的默认行为**对 LoongPort 有害**：例 updater 端点指向 cc-switch 自己的发布源，
  留着会把用户升级成 cc-switch（见 `lib.rs` 里那段说明）。这类要明确禁用并写清理由。

判据一句话：**「不复用」要能说出上游那套具体哪里不适用，说不出就是复用**。

## 二、技术栈事实（别套错工具）

| 项 | 实际 | 常见误判 |
|---|---|---|
| UI 库 | **Radix UI + Tailwind v3**（shadcn/ui 那套） | 不是 Semi Design、不是 antd、不是 MUI |
| Tailwind | **v3.4.x**，配置在 `tailwind.config.cjs`，CSS 用 `@tailwind base` | 不是 v4 —— 别套 `@theme` / `@import "tailwindcss"`，v3 不认，样式会当场崩 |
| 图标 | `lucide-react` | 别引第二个图标库 |
| 样式合并 | `clsx` + `tailwind-merge`（`cn()`） | 别手拼 className 字符串 |
| 后端 | Tauri 2 + Rust，SQLite 走 `rusqlite` | — |

`src/components/ui/` 下是标准 shadcn 封装（`collapsible` / `accordion` / `dialog` /
`select` / `tabs` …，共 23 个），**先翻那个目录再考虑新建**。

### ⚠️ 许可证界线：sub2api 是 LGPL-3.0，本仓是 MIT

| 项目 | 许可证 | 我们能做什么 |
|---|---|---|
| **cc-switch**（本仓 fork 源） | **MIT** | **可自由复用代码** —— §一「能复用就复用」讲的就是它 |
| **sub2api**（对接的中转站后端） | **LGPL-3.0 或更高** | **只能读，不能抄代码进本仓** |

**为什么读它没问题**：我们与 sub2api 的关系是**HTTP 客户端**，不链接、不包含它的代码 ——
跟浏览器访问一个 LGPL 网站一样，不构成衍生作品。

**可以从它源码拿的（接口事实，不受版权保护）**：
端点路径、HTTP 方法、鉴权方式、请求/响应的字段名与类型、状态码语义、
业务规则的**结论**（如「高峰倍率只对订阅型分组生效」）。

**不能拿的（表达形式，受版权保护）**：
整片函数实现、成套的 struct 定义照搬、算法代码逐行翻译。
⚠️ 具体踩点：它前端有个 `platform → app` 映射函数（`KeysView` 的 `ns()`）与我们的
`platform_map` 做同一件事 —— **参考它的取值域可以，照抄那个函数不行**。
我们的 `platform_map` 是独立实现（穷尽 match + 编译期基数闸），有意不同构。

**一句话判据**：**写下来的是「那边的接口长什么样」就没问题，是「那边的代码怎么写的」就不行。**

（顺带：sub2api 的 README_CN 声明「从未授权任何个人或组织基于本项目开展商业化运营」——
那是针对「拿它的代码搭站运营」，不针对「写一个客户端连它」。但商业化前值得再确认。）

### 查 sub2api 的行为：先找它的 Go 源码，别逆推线上 JS

对接 sub2api（端点、字段、鉴权、计费规则）时，**优先看 sub2api 后端的 Go 源码**
（开源，`Wei-Shaw/sub2api`；源码在本机 design 仓的 `upstream/sub2api` 子模块，
路径见工作区那份 CLAUDE.md —— 维护者本机布局唯一源）。
它是契约本身，比读线上 SPA 的 minified bundle、比历史实测记录都权威一个量级 ——
能给到「哪个 handler 第几行填了哪个字段」级别的证据。

三条纪律（都踩过）：

1. **只认 `routes/*.go` 里的路由注册，handler 上方的注释不可信**：
   `user_handler.go` 注释写 `GET /api/v1/users/me` 而真实路由是 `/user/profile`；
   `api_key_handler.go` 写 `/api/v1/api-keys/:id` 而真实是 `/keys/:id`。
2. **注意版本差**：本地 clone 的 commit 与线上跑的版本常常不同
   （实测：本地 `0.1.165` vs 线上 `0.1.169`）。凭源码下的结论，落地前对线上响应复核一次。
3. **别用 HTTP 状态码探测 SPA 路由**：sub2api 前端是 Vue SPA，
   `/topup` `/recharge` `/wallet` 全返回 200（同一个 `index.html`），
   但路由表里根本没这些路径。要查路由得读打包后的 JS 或源码。

### 读上游源码之前：先查代码地图

档案仓里有一份 cc-switch 的 zread wiki（30 页，覆盖 Tauri 2 架构、AppState、SQLite schema
与迁移、Provider 数据模型、Live Config 写入、路由与故障转移、React 组件架构、i18n、测试
体系等）。**读上游源码前先查它，能省掉一轮 grep。**

三条硬约束（都踩过或实测过）：

1. **别 `@` 导入、别软链进本仓** —— 全量 376k 字符，`@` 进来当场炸上下文；且它 untracked
   在子模块工作树里、不入任何 git，软链 commit 后在新 clone 的机器上是悬空链接。按需读单页。
2. **它正文写的「当前版本 3.18.0」是错的**（抄了旧 README），别引它当事实。
   但**行号是准的** —— 六处抽样全部对齐。
3. **子模块指针一 bump 行号就集体失准**（纯注释 commit 也能让整片下移）。指针不自动 bump
   所以当前稳；bump 那天要么重新生成，要么降级成「只读结构、不引行号」。

### 改 codex 模型目录 / 上下文窗口：三条外部事实（对着 codex-rs 源码核实过）

1. **本地 catalog 是该 provider 的唯一权威**。config.toml 配了 `model_catalog_json` 后，
   codex 给这个 provider 建 `StaticModelsManager`（model-provider/src/provider.rs），
   官方 `models_cache.json` 的元数据**完全不参与** —— catalog 里写 262144，codex 就真跑
   256k（比官方默认 272k 还低）。别拿官方缓存推断生成档位上的行为。
2. **`model_context_window` 会被 `max_context_window` 钳制**。config.toml 顶层的
   `model_context_window`（「1M 上下文窗口」开关写的值）在 codex 侧先
   `min(该模型 entry 的 max_context_window)` 再生效（models-manager `with_config_overrides`，
   官方测试名就叫 `model_context_window_override_clamps_to_max_context_window`）。
   曾把映射表值同时钉进两个键，导致 1M 开关对任何填了窗口的行静默失效 —— PR #221 修根：
   不再编造上限；厂商镜像路径保留厂商 max、用户值更高时抬升。
3. **官方 catalog 的形状是两个不同的值**：`context_window`=默认窗口（官方模型当前全是
   272k，2026-08 快照、会漂移），`max_context_window`=上限（gpt-5.4=1M、5.6 系=872k、
   5.5/5.4-mini=272k）。`max_context_window` 是 serde 全可选字段，缺省=不钳制；
   `auto_compact_token_limit` 缺省时按窗口 90% 推导。
4. **官方同名模型默认取官方高窗口**：生成 catalog 时按 slug 匹配本机
   `models_cache.json`（codex 官方目录缓存），命中则默认窗口=官方上限、并如实声明
   该上限（钳制语义因此是正确的）；未命中的第三方模型不声明上限，显式行值永远赢。
   旧的全局兜底 262144 在官方 slug 上是残留，自动剥离让位；Kimi 系列的 262144 是
   真实窗口，未命中官方目录所以保留。

修后语义：映射表「上下文窗口」= 该模型默认窗口（官方模型缺省=官方上限，逐模型不同）；
1M 开关 = 全局覆盖（codex 官方对 `model_context_window` 的定义）；catalog 启动时加载，
改完要重启 codex。生成逻辑全在 `codex_config.rs`：`codex_model_catalog_from_settings`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SailingLoong/LoongPort](https://github.com/SailingLoong/LoongPort) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
