---
trigger: always_on
description: Tauri v2 + Vue 3 + TypeScript 桌面应用。
---

# Yuumi

Tauri v2 + Vue 3 + TypeScript 桌面应用。
原版 Python (Seraphine) 项目的 Rust 重构。

## 编码指导原则 (Coding Guidelines)

**权衡：** 这些原则倾向于谨慎而非速度。对于微不足道的任务，请自行判断。

### 1. 编码前思考 (Think Before Coding)

**不要假设。不要隐瞒困惑。展现权衡。**

- **明确陈述你的假设**：如果不确定，请进行询问。
- **展现多种解释**：如果存在多种解释，请展示它们 —— 不要默默选择其中一种。
- **提倡更简单的方法**：如果有更简单的方法，请指出。在必要时进行反驳/建议。
- **停止并提问**：如果有不清楚的地方，请停下来，指出令人困惑的点，并进行询问。

### 2. 简洁第一 (Simplicity First)

**用最少的代码解决问题。不做任何投机性的编写。**

- **不要添加超出需求的功能**。
- **不要为单次使用的代码进行抽象**。
- **不要添加未要求的“灵活性”或“可配置性”**。
- **不要为不可能发生的情景编写错误处理**。
- **如果写了 200 行代码，而 50 行就能搞定，请重写**。
- **思考**：问问自己：“资深工程师会觉得这太复杂了吗？”如果是，请简化。

### 3. 外科手术式修改 (Surgical Changes)

**只改必须改动的。只清理你自己的烂摊子。**

- **不要“改进”相邻的代码、注释或格式**。
- **不要重构没有损坏/正常工作的代码**。
- **匹配现有的代码风格**，即使你有不同的习惯。
- **对于不相关的死代码，请提及 —— 不要直接删除它**。
- **清理引入的无用代码**：移除因**你的**修改而不再使用的 imports、变量或函数。
- **不要移除预先存在的死代码**，除非被明确要求。
- **检测标准**：修改的每一行都应该能直接追溯到用户的需求。

### 4. 目标驱动执行 (Goal-Driven Execution)

**定义成功标准。循环直到验证通过。**

将任务转化为可验证的目标：

- “添加校验” → “为无效输入编写测试，然后使其通过”
- “修复 Bug” → “编写复现该 Bug 的测试，然后使其通过”
- “重构 X” → “确保重构前后测试均能通过”
- **多步骤任务需陈述简短计划**：
  ```
  1. [步骤] → 验证: [检查项]
  2. [步骤] → 验证: [检查项]
  3. [步骤] → 验证: [检查项]
  ```
- **成功标准**：强大的成功标准能让你独立循环。弱标准（“使其工作”）需要不断澄清。

## Tauri v2 & Vue 3 编码规范 (Tauri & Vue Guidelines)

### Rust 后端 (Tauri v2 / Rust)

- **类型安全边界**：所有与前端交互的 Struct/Enum 必须实现 `serde::Serialize` 和 `serde::Deserialize`。
- **错误传播与序列化**：
  - `#[tauri::command]` 如果可能失败，必须返回 `Result<T, String>`。
  - 严禁随意使用 `unwrap()` 或 `panic!`，应使用 `map_err(|e| e.to_string())` 或 `thiserror` 将 Error 转化为前端友好的 String，并使用系统 `logging.rs` 的 logger 记录完整堆栈。
- **LCU 静态资源与自定义协议规范**：
  - LCU 静态资源（英雄、技能、物品、符文、战利品等图标）由 Rust 端的 `yuumi-asset://` 自定义协议流式返回原始字节，彻底绕开 IPC 与 Base64 传输。
  - WebView2 仅拦截 http/https 协议，前端图片 URL 使用 wry workaround 前缀 `http://yuumi-asset.localhost/`（host 含协议名），由 wry 自动还原为 `yuumi-asset://localhost/` 后再交给 Rust handler。
  - `get_lcu_asset` / `get_lcu_assets` 仅作兼容层保留，新功能**严禁**通过 IPC 命令读取图片 Base64。
- **共享状态管理与死锁防护**：
  - 只能通过 `tauri::State<'_, AppState>` 访问全局状态，不得使用不安全的全局静态变量。
  - 在异步 Command 或后台 Task 中获取状态锁时，**严禁跨 `await` 点持有同步锁 (`std::sync::MutexGuard`)**，必须先释放锁或在作用域块分离后再 `await`，防止 Tokio 线程池死锁。
- **异步与非阻塞**：
  - 严禁在 Command 的主线程中执行耗时的 CPU 计算或 I/O 操作。
  - 使用 `tokio::spawn` 投递后台任务，并在执行完毕后通过 `tauri::Emitter::emit`（Tauri v2 API，禁止使用 v1 的 `emit_all`）异步通知前端。
- **命令注册 Checklist**：
  - 新增 `#[tauri::command]` 时，**必须同步在 `lib.rs` 的 `invoke_handler!` 宏列表中注册**，否则前端调用会提示 command not found。

### Vue 3 前端 (Vue 3 / TypeScript)

- **类型约束与强类型收敛**：
  - 必须对所有 `invoke` 的入参及返回结果定义明确的 TypeScript Interface，绝对禁止使用 `any`。
  - 组件 `defineProps`、Composable 函数参数与返回值、Pinia Store 状态严禁随意使用 `any`；对于具备多形态或阶段性差异的数据结构（如选人与游戏内玩家对象、组队结构），必须在 `src/types/` 下统一定义结构明确的 Interface 或联合类型。
  - 严格处理可选属性（`undefined` / `null`）的空值守卫：在进行函数参数传递、数学运算、数组查找或对象动态 key 索引（如 `playerData[key]`）前，必须通过可选链、空值合并操作符（`??`）或类型守卫完成安全收敛，防止类型逃逸与运行时错误。
  - Rust 返回的 Result 应该在前端有合理的错误捕获（`try-catch` 或 `.catch()`），并通过 `useToast` 或 `message` 呈现给用户。
- **事件监听生命周期管理**：
  - 使用 `@tauri-apps/api/event` 的 `listen` 订阅 Rust 事件时，必须在组件销毁时（`onUnmounted`）调用返回的 `unlisten()` 函数，以防闭包内存泄漏。
- **LCU API 隔离原则**：
  - 前端绝不应直接建立与 LCU 端口的 HTTP/WebSocket 连接。
  - 所有 LCU 接口的调用，必须经由 Rust 端的 `call_lcu_api` 转发，以规避 Token 泄漏并统一错误捕获。
- **静态图片资源渲染规范**：
  - 所有 LCU 静态资源（英雄、技能、装备、符文、战利品图标等）统一使用 `<LcuImage :src="path" />` 组件或 `useLcuAsset` composable。
  - 资源加载底层基于 `yuumi-asset://` 自定义协议（前端使用 `http://yuumi-asset.localhost/` workaround URL）与 Chromium 原生网络层，自带 HTTP 强缓存与原生解码，**严禁使用 IPC `invoke` 批量传递图片 Base64 字符串**。
- **组件及路由状态保留**：
  - 页面路由切换基于 Vue 的 `currentPage` 控制。
  - Search 页和 GameInfo 页由于数据量较大且需要保留搜索/对比状态，必须使用 `v-show` 保持组件挂载，避免重新渲染销毁状态。
- **主题与样式**：
  - 遵循 "纯白水晶极光" 风格，背景使用毛玻璃模糊（`backdrop-filter: blur`），配色统一采用动态 CSS 变量，不可随意硬编码色值。

## 技术栈

- **前端**: Vue 3 + TypeScript + Vite + Pinia + Naive UI
- **后端**: Tauri v2 (Rust)
- **包管理**: pnpm

## 常用命令

```bash
pnpm tauri dev      # 开发（Vite + Tauri 窗口）
pnpm tauri build    # 构建生产包
pnpm dev            # 仅前端开发
pnpm build          # 仅前端构建

# 代码质量校验与测试（修改代码后进行验证）
pnpm type-check      # Vue / TS 类型检查 (vue-tsc --noEmit)
pnpm clippy          # Rust 代码静态检查 (cargo clippy)
pnpm format          # Rust 代码格式化 (cargo fmt)
pnpm test:rust       # 运行 Rust 单元测试 (cargo test)
pnpm check-all       # 一键检查全部 (Format + Type-Check + Clippy)
```

## 项目结构

```
Yuumi/
├── src/                            # Vue 前端
│   ├── App.vue                     # 根组件（自定义标题栏 + 导航栏 + 路由切换）
│   ├── main.ts                     # 主窗口入口
│   ├── opgg.ts                     # OP.GG 独立窗口入口
│   ├── i18n.ts                     # 多语言配置
│   ├── api/
│   │   └── lcu.ts                  # LCU API 封装 + Rust 命令调用
│   ├── store/
│   │   └── lcuStore.ts             # Pinia 全局状态（LCU 事件映射）
│   ├── utils/
│   │   └── theme.ts                # 主题色动态更新
│   ├── composables/
│   │   ├── useLcuAsset.ts          # LCU 资源路径 → data URL（缓存 + 去重）
│   │   ├── useToast.ts             # Naive UI 消息提示 Hook (多窗口安全降级)
│   │   ├── usePlayerSearch.ts      # 召唤师名称点击跳转搜索
│   │   ├── useTftData.ts           # 云顶之弈 API & 数据处理
│   │   ├── useTftMetaDecks.ts      # 云顶 OP.GG 热门阵容解析
│   │   ├── useLoot.ts              # 战利品智能开箱/分解/重铸
│   │   ├── useMatchHistory.ts      # 战绩历史 Hook
│   │   ├── usePremadeGroup.ts      # 组队分析 Hook
│   │   ├── useGamePlayerData.ts   # 对局玩家数据集中管理
│   │   └── useAutoSaveConfig.ts    # 配置项自动保存 Hook
│   ├── assets/                     # 静态资源（图片等）
│   ├── views/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ISuuuu/Yuumi](https://github.com/ISuuuu/Yuumi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
