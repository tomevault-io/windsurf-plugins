---
trigger: always_on
description: * 新功能设计文档放在 `docs/specs/`，文件名 `YYYY-MM-DD-<功能名>.md`
---

# 项目指令 — epytor

## 语言

* **始终用简体中文回复**

***

## 需求

* 新功能设计文档放在 `docs/specs/`，文件名 `YYYY-MM-DD-<功能名>.md`
* 先写 spec 再开发——明确需求范围、交互边界、验收标准
* 配置项参考见[开发 → 配置参考](#配置参考)

***

## 开发

### 基本规则

* **包管理器**：必须用 `pnpm`，禁止 npm/yarn
* **构建**：修改代码后执行 `pnpm build` 验证编译无误
* **调试**：F5 启动扩展调试实例（`.vscode/launch.json`）
* **语言**：全部 TypeScript；Extension 端用 `tsconfig.json`，WebView 端用 `tsconfig.webview.json`
* **双目标构建**：`dist/extension.js`（Node.js）+ `dist/webview.js`（Browser），由 `esbuild.mjs` 完成
* **Git commit 规范**：commit 描述用**中文**，类型前缀保留英文（`feat:`、`fix:`、`refactor:`、`chore:`、`docs:` 等）。例：`feat: 新增XXXX功能`、`fix: 修复XXXX问题`
* **逐项提交**：todo list 中每完成一个独立任务**必须**单独 `git commit`，禁止多个 todo 混在一个 commit 中（方便出问题时精确回溯）
* **诚实原则**：不确定的事直接说"不确定"，禁止编造 URL、issue 编号、API 接口、文档引用或任何事实性信息
* **优雅原则**：禁止 hack 或补丁式写法，优先使用框架/库官方 API、CSS 变量、配置回调等正路方案
* **自检原则**：代码移动/提取后**必须**搜索确认旧位置已删除，不得留有死代码或同名遮蔽；标记 roadmap 条目完成前逐项列出实际完成项与未完成项，不得将部分完成标记为整体完成
* **查证原则**：引用文件位置、函数名、调用关系时，若不确定则先 grep 确认再写，禁止凭记忆编造

### 架构约束

* WebView ↔ Extension 通信**只通过** `webview/messaging.ts` 中封装的函数
* WebView 侧不直接 `import` VSCode API，通过 `acquireVsCodeApi()` 获取句柄
* CSS 必须使用 `--vscode-*` 变量以适配亮/暗主题
* 不在模块外部维护全局状态（单例除外，如 editor view）

### 关键文件速查

```
src/extension.ts                         — 扩展入口，注册 CustomEditorProvider
src/MarkdownEditorProvider.ts            — Provider 核心（消息路由、自动保存、revert）
src/utils/getNonce.ts                    — CSP nonce 生成
src/utils/imageService.ts               — 图片本地保存（MD5 去重）+ 服务器上传
src/i18n/webviewTranslations.ts         — WebView 翻译数据
webview/index.ts                         — WebView 入口（消息路由、DOM 事件委托、品牌标识注入）
webview/editor.ts                        — CrepeBuilder 入口（Milkdown 7.21.2 + Crepe 原生功能注册）
webview/messaging.ts                     — WebView ↔ Extension 消息协议（唯一通信层）
webview/style.css                        — VSCode 主题全覆盖（--vscode-* CSS 变量，覆盖 Crepe 组件）
webview/i18n/index.ts                    — t() / kbd() 翻译函数
webview/ui/icons.ts                      — SVG 图标
webview/ui/tooltip.ts                    — Tooltip 组件
webview/utils/themeBus.ts               — Mermaid/CodeMirror 深浅主题统一事件总线
webview/components/selectionToolbar/index.ts — 选区变更回调（驱动源码行号映射）
webview/components/toc/index.ts         — 目录（TOC）面板（吸底工具栏下方、可固定、可拖拽宽度）
webview/components/imageView/index.ts   — 图片 NodeView（选中/lightbox/工具栏/缩放 handle）
webview/components/findBar/index.ts     — 编辑器内查找栏（Cmd/Ctrl+F）
webview/components/pathLink/            — 路径链接自动补全
webview/headingIds.ts                    — 标题 id 管理（不操作 DOM，仅保留签名）
docs/specs/                              — 功能 spec 文档
docs/roadmap.md                          — 项目路线图（面向用户的功能规划）
docs/tech-debt.md                        — 技术债务清单（面向开发者的代码改进）
```

### 配置参考

| 设置项 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| `epytor.autoSave` | boolean | `true` | 编辑后自动写盘 |
| `epytor.autoSaveDelay` | number | `1000` | 防抖延迟（ms） |

***

## 测试

### 技术栈

| 层次 | 框架 | 适用范围 |
| :--- | :--- | :--- |
| Extension 单元测试 | **Vitest 2.x**（Node 环境） | `src/utils/`、`src/MarkdownDocument.ts` |
| WebView 单元测试 | **Vitest 2.x + jsdom 24.x** | `webview/utils/`、`webview/messaging.ts` |
| 集成测试（计划中） | **@vscode/test-electron + Mocha** | 需真实 VSCode Extension Host |

`vscode` 模块通过 `__mocks__/vscode.ts` 统一 mock，由 `vitest.config.ts` 的 `resolve.alias` 注入，禁止在单个测试文件中 `vi.mock("vscode")`。

### 命令

```bash
pnpm test              # 一次性运行全部单元测试
pnpm test:watch        # 监听模式（开发期间使用）
pnpm test:coverage     # 运行测试并生成覆盖率报告（coverage/）
```

### 目录与命名

```
src/__tests__/           — Extension 侧单元测试（Node 环境）
webview/__tests__/       — WebView 侧单元测试（jsdom 环境）
webview/__tests__/setup.ts  — jsdom 全局 setup（注入 acquireVsCodeApi）
shared/__tests__/        — 共享类型测试
__mocks__/vscode.ts      — vscode API 统一 mock
```

* 测试文件命名：`<模块名>.test.ts`，与被测文件同名
* 测试结构遵循 **AAA 原则**（Arrange / Act / Assert），`describe` → `it` 两层
* `it` 描述格式：`输入条件 应该 期望结果`（中文）

### 覆盖率要求

| 模块 | 行覆盖率下限 |
| :--- | :--- |
| `src/utils/imageService.ts` | ≥ 85% |
| `src/utils/getNonce.ts` | 100% |
| `src/MarkdownDocument.ts` | ≥ 80% |
| `src/utils/contentTransform.ts` | ≥ 90% |
| `src/utils/lineMap.ts` | ≥ 90% |
| `webview/utils/slug.ts` | ≥ 90% |
| **整体** | ≥ 70% |

### 强制流程

**每次代码改动**（bug 修复、新功能、重构还债）必须完整走完以下流程：

```
代码改动 → pnpm build → pnpm test → 输出手测清单 → vscode_askQuestions 逐项确认 → git commit
```

各阶段详细要求：

**阶段一：自动化验证**

1. 运行 `pnpm build` 确认编译无误
2. 运行 `pnpm test` 确认全部测试通过
3. 任一失败则先修复，不得跳过

**阶段二：人工验收**

1. **输出手测清单**：列出受影响的交互路径和验收点，每条一行，编号排序
2. **逐项确认**：通过 `vscode_askQuestions` 弹出确认框（每屏 ≤4 项，超过则分屏）
3. 开发者逐项选择「✅ 通过」或「🛑 有问题」
4. 全部通过 → 进入阶段三；有任一未通过 → 修复后重新从阶段一开始

**阶段三：提交**

方可 `git commit`（遵循[逐项提交](#基本规则)规则）

***

**功能开发后**附加要求：

* 编写对应单元测试（核心逻辑、边界值、异常路径各至少一个用例）

**Bug 修复后**附加要求：

* 先补充**能复现该 bug 的测试用例**（写在修复同一 commit 内）
* 确认该用例在修复前失败、修复后通过

**git push 前**：

* **必须**执行 `pnpm test`，全部通过才允许推送

### 测试失败处理

```
测试失败
  │
  ├─ 是新引入的失败？→ 定位代码变更，修复后重新运行
  │
  ├─ 是测试预期不符实现（实现已有意变更）？→ 同步更新测试
  │
  └─ 是环境/依赖问题？→ 检查 jsdom 版本、vscode mock 是否完整
```

**禁止行为**：

* 禁止跳过（`it.skip`）或注释失败的测试用例来让 CI 通过
* 禁止修改测试预期值来掩盖 bug（除非实现有意变更且经过评审）
* 禁止在未运行测试的情况下 push 到 `main` 或 `dev` 分支

### Mock 规范

* 每个 `describe` 块在 `beforeEach` 中调用 `vi.clearAllMocks()` 重置 mock 状态
* 文件系统操作统一 mock `vscode.workspace.fs`（禁止使用真实 fs 写磁盘）
* 依赖时间的逻辑使用 `vi.useFakeTimers()` / `vi.useRealTimers()`，禁止 `setTimeout` 真实等待
* 禁止测试 `private` 类方法，通过公共接口验证行为

### CI 自动化


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peiyucn/epytor](https://github.com/peiyucn/epytor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
