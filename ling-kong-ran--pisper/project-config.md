---
trigger: always_on
description: 本文档用于指导在 **Pisper** 仓库中工作的开发者和 AI 代理。
---

# AGENTS.md

本文档用于指导在 **Pisper** 仓库中工作的开发者和 AI 代理。

## 项目概览

Pisper 是一个覆盖桌面端、终端和移动端的多代理应用，基于 [Pi Coding Agent](https://github.com/earendil-works/pi/tree/main/packages/coding-agent) 构建。项目包含：

- React Web 界面（开发服务器和生产静态资源）
- Node.js 应用运行时与桌面伴随进程，承载会话、工具、MCP、技能、记忆、工作流、通道和计划任务
- Tauri 2 桌面壳，包含系统 WebView 和 Node SEA 伴随进程
- Rust Ratatui TUI 客户端

源码开发、验证与桌面 SEA 构建以 **Node.js 24** 为基线，补丁版本必须满足锁定依赖的要求。源码环境、npm 启动器所用宿主 Node 与移动端嵌入式 Node 的要求分别见 [`docs/development-workflow.md`](docs/development-workflow.md)，不得从旧的 `engines` 声明推断完整 Runtime 支持范围。运行时代理数据默认保存在 `~/.pisper/agent`，可通过 `PISPER_AGENT_DIR` 覆盖。不得提交该目录中的 API 密钥、机器人凭据或个人数据。

## 架构

| 路径 | 职责 |
| --- | --- |
| `src/` | React + TypeScript 前端（Vite、Tailwind 4、shadcn/ui、Zustand、i18next） |
| `src/app/` | 应用壳：路由、Provider、i18n 接线和导航 |
| `src/features/` | 功能页面（聊天、配置、技能、MCP/插件、工作流等） |
| `src/components/ui/` | shadcn/ui 基础组件；优先组合这些组件，避免编写一次性样式 |
| `src/components/app/` | 应用级通用组合组件的推荐归属；按需要创建，不承载 Feature 业务规则 |
| `src/locales/{zh-CN,en-US}/` | 翻译命名空间（`namespace:key`） |
| `runtime/` | Node 应用运行时（纯 ESM `.mjs`）：HTTP API、代理集成、服务和工具 |
| `runtime/runtime/` | Pi 代理运行时集成 |
| `runtime/services/` | 领域服务（会话、MCP、计划任务、工作流、Provider 等） |
| `runtime/tools/app/` | 应用级代理工具（每个工具一个模块） |
| `runtime/tests/` | 当前统一的 Node 测试入口，涵盖 Runtime、前端纯逻辑、协议、构建与源码守卫 |
| `packages/pisper/` | 私有源代码清单，以及发布为 `pisper` 的 npm 安装器/启动器 |
| `shared/` | Runtime 与客户端共享的小型 JS 模块（例如工作流图和发布说明） |
| `src-tauri/` | Tauri 2 壳（桌面桥接、宠物窗口、更新器、CLI 安装） |
| `src-tui/` | Rust TUI（`pisper` CLI） |
| `crates/` | 独立 Rust 库、原生插件和平台适配模块 |
| `scripts/` | SEA 打包、发布、冒烟测试和 i18n 检查脚本 |
| `docs/` | 产品文档、截图、赞助配置和打包说明 |
| `dist/` | Vite 生产构建输出 |
| `release/` | 已打包的 SEA/TUI/Tauri 产物（已被 Git 忽略） |

**桌面端运行形态：** Tauri 壳 → `pisper-sidecar`（Node SEA）→ `sidecar-runtime/`（Pisper Runtime、Pi 包、技能和原生模块）。开发 Web 流程中，`runtime/index.mjs` 嵌入 Vite 中间件，并默认在 `127.0.0.1:5173` 提供 SPA 和 API。

路径别名：`@/*` → `src/*`，`@shared/*` → `shared/*`。

## 规范执行方式

本文件同时服务于开发者和 AI 代理，规则分为三类：

- **强制边界：** 安全、协议兼容、发布流程，以及以“必须”“禁止”“不得”“只能”表述的要求。基础组件不依赖 Feature、跨 Feature 不访问内部实现、不得新增循环依赖均属于强制边界，不能仅凭变更说明豁免。
- **默认建议：** 以“默认”“优先”“建议”“可以”“应”表述的组织方式，例如子目录名称、组件共置和具名导出；其中同时含有明确禁止条件的部分仍属于强制边界。采用其他方案时，说明职责归属、依赖方向与取舍即可，不要求额外审批。
- **迁移债务：** 存量代码、工具与目标规范的已知偏差。新代码不得扩大偏差；本次涉及的区域优先治理，无法在当前范围完成的部分记录责任范围、迁移条件与验证方式，不要求顺带重写全仓库。

命令脚本、测试和 CI 是当前执行行为的事实来源，检查存在不代表其断言永远合理。发现与规范冲突时，先核实保护目标：在同一范围内修正过时文档，或由负责集成的人提供替代验证并更新检查；超出任务范围时记录迁移项及当前失败。不得通过跳过检查、删除行为保护或口头解释宣称通过。已知偏差与检查覆盖范围见 [`docs/development-workflow.md`](docs/development-workflow.md)。

开发环境、依赖升级和验证矩阵见 [`docs/development-workflow.md`](docs/development-workflow.md)；故障排查见 [`docs/troubleshooting/README.md`](docs/troubleshooting/README.md)；跨层架构决策见 [`docs/architecture-decisions.md`](docs/architecture-decisions.md)。

## 代码结构与模块边界

代码结构以业务边界、变更原因、依赖方向和可测试性为主要依据。现有目录不要求一次性迁移。允许以修复目录归属、职责混杂或依赖边界为目标的独立重构任务，不要求附带功能变更；任务应限定业务范围，避免无关搬迁：

- 每个模块应有清晰的主要责任。前端页面、运行时服务、HTTP 适配器、平台桥接和共享协议放在能表达其责任的层中；跨层实现应通过明确的适配器或契约连接。
- `src/features/<area>/` 采用功能内聚的组织方式。只有在子域有独立的变更原因、测试边界或依赖关系时才继续分组；不要仅因为文件数量增加就机械创建目录。
- `src/components/ui/` 放置无业务语义的基础组件；需要应用级通用组合组件时可以使用 `src/components/app/`。两者不得依赖任何 Feature 实现，包括其公共入口；业务数据和动作通过参数、回调或插槽传入。页面、路由和应用壳是组装公开 Feature 的组合边界。
- 跨 Feature 通过明确的公共接口协作。被多个 Feature 使用不自动构成提升到全局目录的理由：有业务归属的模型、事件和逻辑仍由原领域持有；只有无业务归属或确实跨客户端、跨运行时复用的内容才按下文“目录职责”选择共享位置。
- 页面文件负责页面编排和状态连接。模型、校验、表单、弹窗、列表和可视化逻辑可以与页面同域共置；当独立的职责或生命周期形成清晰接口时再提取模块，并按需要分组，不要求为每一种技术角色建立子目录。
- 一个文件可以声明多个共同变化的组件，例如 `Dialog` 与 `DialogContent`，或只服务于当前页面的局部展示组件。当组件具有独立业务职责、生命周期或实际复用需求，且拆分能形成清晰接口时，优先独立文件；是否共享状态、能否单独测试和导出数量都不单独作为拆分依据。
- 文件行数只作为评审信号，禁止新增仅凭固定行数判定架构正确性的硬门槛。优先处理多个变化原因、难以测试、依赖混乱或高冲突的模块；统计时区分生产代码、内联测试和生成内容。已有行数守卫按开发流程中的迁移项处理，不通过压缩代码或搬运方法凑行数。
- 拆分后应有明确的状态所有者、生命周期与窄接口。仅通过共享 `this`、原型注入、双向回调或传递整个 Runtime 对象搬移方法，不视为完成职责解耦。对共同变化、拆开后产生大量透传和跨文件跳转的片段，可以合并；结构评审同时允许拆分与合并。
- `runtime/http/routes/` 负责请求解析、权限边界、调用服务、流式编排和响应序列化；复杂业务逻辑放在服务层。只有当路由具有独立权限、生命周期或测试边界时才按 API 子域拆分，不要求每个端点都单独建文件。
- `runtime/services/` 按领域和用例组织。配置归一化、持久化、网络连接和归档处理等职责在拥有独立测试边界时拆成同域辅助模块，不为追求小文件而拆分。
- TUI 和 Tauri 也遵循按责任拆分的原则。入口文件保留必要的编排和公共状态；渲染、生命周期、代理/配对、设备操作、语音和资源处理在确有独立边界时拆分。
- Runtime、Web 和 TUI 共享的 HTTP/SSE 数据必须有单一契约来源，或至少有覆盖关键负载和错误情况的契约测试。契约变更应优先采用向后兼容的增量方式；需要分阶段迁移时，保留旧字段或版本并说明移除条件。
- 开始结构性改动前，先说明现有问题、文件移动映射、状态归属、依赖方向、受影响客户端与验证范围。验收检查调用方、懒加载、路由、导出、测试发现入口和源码守卫均已同步，且未增加隐式依赖；只改变文件位置不能作为职责治理完成的依据。记录与回滚要求见 [`docs/architecture-decisions.md`](docs/architecture-decisions.md)。

## 命令

```bash
# 安装
npm install                 # 根据 package-lock.json 安装依赖
npm ci                      # 为 CI/发布执行可复现的全新安装

# 日常 Web 服务器
npm run dev                 # 使用 Vite 中间件启动 Web 与 API
npm run build               # 构建生产前端并检查产物体积预算
npm run preview             # 通过生产服务器提供已构建资源
npm start                   # preview 的别名

# 质量检查
npm run typecheck           # 检查 src、Vite 配置及 tsconfig.jscheck.json 指定范围
npm run lint                # 运行 oxlint
npm run format              # 执行 prettier --write .
npm run format:check        # 检查 Prettier 格式
npm run i18n:check          # 检查 zh-CN 和 en-US 是否定义了所有字面量 src 键
npm run check               # 依次执行 typecheck、lint、i18n:check、format:check 和 test:startup
npm test                    # 运行 runtime/tests/*.test.mjs 中的全部测试
npx tsx --test runtime/tests/foo.test.mjs  # 运行一个或多个指定测试文件

# Node SEA 伴随进程
npm run sidecar:dev         # 在开发环境直接运行伴随进程
npm run sidecar:sea         # 构建 Node SEA 并准备运行时闭包
npm run sidecar:sea:smoke   # 对暂存的 SEA、运行时和 API 执行冒烟测试


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ling-kong-ran/pisper](https://github.com/ling-kong-ran/pisper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
