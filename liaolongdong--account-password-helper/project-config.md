---
trigger: always_on
description: - 这是一个本地优先的密码管理器浏览器扩展。处理改动时按以下顺序权衡：安全与隐私、功能正确性、数据与行为兼容性、可维护性、性能、代码风格。
---

# Account Password Helper 项目规则

## 适用范围与优先级

- 本文件适用于整个仓库。
- 这是一个本地优先的密码管理器浏览器扩展。处理改动时按以下顺序权衡：安全与隐私、功能正确性、数据与行为兼容性、可维护性、性能、代码风格。
- 开始修改前先阅读相关实现、测试及 `docs/ARCHITECTURE.md`；不要仅凭文件名或猜测改变行为。
- 只做满足当前需求所需的最小完整改动。保留工作区中的用户改动，不覆盖、不回退、不顺手格式化或重构无关文件。
- 未经用户明确要求，不改变既有功能、交互、默认值、存储结构、加密格式或浏览器权限；确有必要时，先说明影响并询问确认。
- 未经用户明确要求，不提交、推送、发布代码，也不升级依赖或改写 lockfile。

## 快速参考

### 常用命令

| 用途     | 命令                                                 |
| -------- | ---------------------------------------------------- |
| 开发     | `pnpm dev`（端口 8899）                              |
| 构建     | `pnpm build`（Firefox: `pnpm build:firefox`）        |
| 类型检查 | `pnpm typecheck`                                     |
| Lint     | `pnpm lint`（修复: `pnpm lint:fix`）                 |
| 样式检查 | `pnpm lint:style`（修复: `pnpm lint:style:fix`）     |
| 格式检查 | `pnpm format:check`                                  |
| 测试     | `pnpm test:run`（单文件: `pnpm test:run -- <file>`） |
| 图标构建 | `pnpm icons:build`                                   |
| 包分析   | `pnpm analyze`                                       |

### 关键文件速查

| 职责             | 文件                                                                     |
| ---------------- | ------------------------------------------------------------------------ |
| 公共类型定义     | `utils/types.ts`                                                         |
| Storage 键名     | `utils/storageKeys.ts`（`STORAGE_KEYS` + `SESSION_MEMORY_KEYS`）         |
| 存储门面         | `utils/storage.ts` → `utils/storage/` 子模块                             |
| 加密核心         | `utils/encryption.ts`（PBKDF2 + AES-256-GCM）                            |
| 会话管理         | `utils/sessionManager.ts` + `utils/sessionManager-storage.ts`            |
| 消息路由         | `entrypoints/background/messageRouter.ts`                                |
| 侧边栏管理       | `entrypoints/background/sidePanelManager.ts`                             |
| SW 保活与闹钟    | `entrypoints/background/backgroundServices.ts`                           |
| Vue i18n（完整） | `utils/i18n/`（Options/SidePanel/Popup）                                 |
| 轻量 i18n        | `utils/i18n-lite.ts`（Content/Background，`tl()` 函数）                  |
| 主题令牌         | `assets/theme/tokens.css`（`--aph-*` 变量，6 套色彩方案）                |
| WXT 配置         | `wxt.config.ts`（路径别名、Element Plus 按需引入、非阻塞 CSS）           |
| 测试配置         | `vitest.config.ts`（`WxtVitest()` 插件，Node 环境，Web Crypto 原生支持） |
| 文档地图         | `docs/README.md`（目录职责、入库与本地产物的判定口径）                   |

## 代码改动与优化边界

### 修改功能或修复问题时

- 修改既有代码时，同时检查本次触及范围内是否存在明显的大文件职责混杂、重复逻辑、超大组件、散落类型、重复常量或可独立测试的公共方法；在能证明行为等价且风险可控时，可以随当前改动一并做局部优化。
- 顺带优化必须服务于当前任务，只限直接相关模块，不扩展成全仓库重构，不借机改名、换风格、替换技术方案或调整无关交互。
- 功能改动与结构优化应尽量分成清晰、可审查的步骤：先建立或确认当前行为，再完成结构调整，最后实现需求；避免在一个大改动中同时混合行为变化和大规模搬迁。
- 修复 bug 时先用测试或可复现步骤固定预期行为。若在修改中发现当前任务之外的 bug、体验问题或产品改进点，先记录并说明，不顺手改变行为。
- 对触及代码做必要清理，但不要为了形式上的“抽象”增加层级。只有当抽取后职责更清晰、能减少真实重复或能独立测试时才创建新组件、composable、类型或工具方法。

### 明确进行代码优化或重构时

- 优化代码的默认目标是内部结构改善，必须保持外部可观察行为不变，包括功能结果、异常与降级路径、UI 布局和文案、操作步骤、焦点与键盘行为、动画与滚动、事件时序、消息协议、存储数据、默认值、加密格式、权限和网络行为。
- 优化前先阅读调用方和现有测试，明确被优化模块的职责、入口、输出、副作用和兼容边界；必要时先补充特征测试或回归测试，再移动或重写代码。
- 大文件拆分应按稳定职责和数据流划分，保持原入口为薄的编排层。拆分后避免循环依赖、双向状态同步和跨层访问，不以文件行数作为唯一拆分依据。
- 重复代码抽离前确认各处语义、错误处理、生命周期和未来变化方向一致；看起来相似但业务约束不同的代码不得强行合并。
- Vue 组件抽离应有明确的 UI 职责和类型化 props/emits 契约。不得因为抽离改变 DOM 结构、样式作用域、焦点顺序、事件传播、挂载时机或响应式更新时序。
- composable 抽离负责响应式状态与生命周期副作用；纯函数和无状态算法放入 `utils/`。抽离后应保持状态所有权单一，并确保 listener、observer、timer 和 watcher 的注册及清理次数不变。
- TypeScript 类型放在最接近其所有者的位置；只有被多个模块稳定共享时才上移到公共类型文件。抽离类型不得弱化约束、扩大联合类型或用可选字段掩盖不合法状态。
- 公共方法应围绕单一业务能力设计，参数和返回值明确，优先纯函数。不要创建笼统的 `helpers`、`common` 或万能工具类，也不要为了减少少量行数隐藏关键业务语义。
- 变量或常量提取应表达业务含义、消除魔法值或避免重复计算；一次性且语义清晰的表达式不机械提取。共享常量放在所属领域附近，真正跨领域时才放入 `utils/constants.ts`。
- 迁移、重命名或抽取后及时删除已被替代的死代码并更新所有引用、测试和必要文档，但不改动无关文件。
- 优化完成后使用与优化前相同的场景和测试验证行为等价，并针对新边界补充测试。性能优化还需提供前后使用同一方法获得的数据，不以主观感受判断。

### 必须暂停并询问确认的情况

- 如果优化需要或可能改变任何功能、业务规则、用户交互、视觉呈现、默认值、数据格式、兼容性、权限、隐私边界或性能取舍，立即停止实现，先向用户说明当前行为、拟议变化、原因、影响范围和可选方案，获得明确确认后再继续。
- 如果无法可靠判断重构是否行为等价，视为可能影响功能或交互，必须先询问确认，不能以“应该没有影响”作为继续依据。
- 如果保持兼容需要明显增加复杂度，或者发现原行为存在缺陷但修正会改变用户体验，也必须让用户在“保持原行为”和“接受行为调整”之间做决定。
- 纯内部、范围明确、已有验证覆盖且可证明行为等价的重构可以直接执行；交付时仍须列出优化内容和验证证据。

## 技术栈与包管理

- 使用 WXT、Manifest V3、Vue 3、TypeScript、Element Plus、Vite、Vitest。
- 使用 `pnpm`，以 `package.json` 的 `packageManager` 和 `pnpm-lock.yaml` 为准；不要混用 npm 或 yarn。
- 优先复用现有依赖与 Web/Chrome 原生 API。新增或升级依赖前，先证明必要性并评估包体积、安全、权限和兼容性影响。
- Element Plus 继续通过现有 resolver 按需引入，禁止整包导入。
- 保持 TypeScript strict 模式，不降低 `tsconfig.json`、ESLint、Stylelint 或测试规则来绕过问题。

## 仓库架构约定

- `entrypoints/`：WXT 扩展入口。入口文件只负责初始化、依赖装配和生命周期连接，复杂逻辑下沉到对应模块。
- `entrypoints/background/`：后台 Service Worker、消息路由和扩展生命周期逻辑。
- `entrypoints/content/`：页面识别、填充和注入式 UI；避免阻塞页面主线程，不污染宿主页面全局样式或变量。
- `components/`：Vue 展示与交互组件；`composables/`：可复用的响应式状态和副作用；`utils/`：与 Vue 生命周期无关的领域逻辑和纯工具。
- `utils/storage.ts` 与 `utils/storage/` 是存储访问边界；`utils/encryption.ts`、`utils/crypto-light.ts` 和既有会话模块是密码学与会话边界。不要在 UI 或入口中复制这些逻辑。
- 跨入口消息应由后台路由统一处理，并使用 `utils/types.ts` 或就近的类型文件定义判别联合和响应类型。新增消息必须处理未知类型、失败响应和异步通道生命周期。
- Background Service Worker 随时可能被回收。不得把全局内存当作持久事实来源；短期缓存必须可重建、可失效，并以合适的 `chrome.storage` 数据为准。长期任务使用 `chrome.alarms` 等 MV3 机制。
- 注册事件、Port、observer、timer 或 DOM 监听器时，保证幂等初始化并在生命周期结束时清理，防止重复监听和泄漏。

## 安全与隐私基线

- 不记录、上传或暴露主密码、账号、密码、TOTP secret、解密后的条目、恢复数据、剪贴板内容、加密密钥或可推导这些信息的完整对象。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liaolongdong/account-password-helper](https://github.com/liaolongdong/account-password-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
