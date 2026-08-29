---
trigger: always_on
description: 本文件是 /home/leo/code/dsh-pager-grok 的项目级执行约定。
---

# dsh-pager-grok Agent Instructions

本文件是 /home/leo/code/dsh-pager-grok 的项目级执行约定。

## 开始任何工作前

先阅读：

1. docs/GROK_BUILD_TUI_DEEPSEEK_ADAPTATION_PLAN.md；
2. docs/ARCHITECTURE.md；
3. docs/SOURCE_POLICY.md；
4. docs/TESTING.md；
5. docs/开发进度记录/README.md。

如果任务涉及 Grok 源码，先核对：

- 上游目录：/home/leo/code/grok-build；
- mirror commit：19d42e35c07a9c9244f03f6df0c4c353f970d4f9；
- SOURCE_REV：7d67deacbeb1c1093fdb4f9bcbfab2630e18a6aa；
- crates/dsh-pager-grok-ui/SOURCE_MANIFEST.md、许可证和对应上游测试。

## 绝对目标

本项目的终极目标是：保持 Grok Build TUI 原有的前端样式、布局、交互状态机、
快捷键、鼠标行为、滚动/选择、主题、动效和终端生命周期，把它适配到
DeepSeek Harness。

所有实现必须遵守以下分工：

- Grok 决定用户如何看到和操作：AppView、AgentView、组件树、Theme/Appearance、
  focus、Esc ladder、scrollback、selection、picker、queue、modal、status、
  timeline、viewer 和终端呈现优先复用 Grok；
- DSH 决定系统真实发生了什么：protocol、transport、session、control plane、
  replay/live、generation、queue authority、interaction、RPC effect 和
  DeepSeek 产品数据由 dsh-pager 保持；
- adapter 负责两者接线：SessionState → DSH-neutral DTO，以及
  UiIntent → UiEffect → receipt/notification。

数据不同不是重写 UI 的理由。Grok 有同职责实现时，必须先复制、移植或结构性
适配；不得因为适配超过 100 行、现有 DSH 代码已经能运行或名称不同，就新建
第二套同职责 UI。

## 修改仓库前的强制记录

任何会改变仓库状态的操作都适用本规则：代码、配置、文档、脚本、vendor、
Cargo.lock、测试 fixture、格式化结果和生成文件都包括在内。

只读搜索、分析、编译和测试可以先做。第一次目标文件修改前，必须在
docs/开发进度记录 创建新的时间戳记录：

~~~text
YYYY-MM-DD_HH-mm-ss_<简短主题>.md
~~~

时间使用 Asia/Shanghai（+0800），精确到秒。记录必须先列出目标、背景、设计
契约、计划文件、风险、回滚方式和预期验证。之后只能修改记录列出的范围；
范围扩大前先创建新记录。

完成或失败后，在同一份记录中写入实际文件、摘要、验证结果、阻塞和下一步。
回写同一份记录是该批次的收尾动作，不需要递归创建记录。历史记录不删除、
不静默改写。

## Git 提交规则

一份主进度记录对应一个 Git commit，一个 commit 只能对应一个主进度记录。
commit message 必须包含：

~~~text
Progress-Record: docs/开发进度记录/YYYY-MM-DD_HH-mm-ss_<简短主题>.md
~~~

提交前必须检查：

- 暂存区只有该记录和记录中列出的目标文件；
- 没有第二份主记录、无关格式化或生成物；
- trailer 路径与实际文件名完全一致；
- 需要拆分的工作先创建新的记录；
- 需要 squash 时先合并记录范围，再生成最终提交。

记录和目标文件应在同一个 commit 中提交。不要为了事后写 commit hash 产生
无意义的额外提交；记录路径、trailer 和提交文件列表是绑定依据。

当前 successor 尚没有 Git 元数据。Git 根确认/初始化、历史记录审计和
trailer/范围自动校验属于 M0.1；在此之前不能声称已有 commit 对应验收。

## 架构所有权

| 区域 | 负责内容 | 规则 |
|---|---|---|
| crates/dsh-pager-protocol | 线协议、版本、wire DTO | 保持向后兼容和 fixture；不放 UI |
| crates/dsh-pager | transport、loader、SessionState、control plane、presentation | 只拥有 DSH 真源；不得 import Grok view 或 ratatui Frame |
| crates/dsh-pager-grok-ui | Grok-derived view/input/layout、host adapter、effect reducer | view 不调用 RPC；适配逻辑集中在 adapter |
| crates/dsh-grok-inline / dsh-grok-textarea | Grok 低耦合终端和编辑器 | 保留来源、许可证和上游测试 |
| crates/dsh-pager-primitives / render | 中性渲染 helper、terminal lifecycle glue | 已有 Grok 实现时不要再维护平行算法 |
| crates/dsh-pager-bin | 参数、backend shell、非交互 smoke、UI 入口 | 不实现布局和业务 view |
| crates/dsh-pager-test-support | fixture、mock Harness、PTY screen、semantic snapshot | 新行为必须在合适层级留下证据 |
| docs/开发进度记录 | 每批变更的审计记录 | 先记录、后修改；一记录一 commit |

旧 app.rs、block_viewer.rs、picker.rs、queue_view.rs、selection.rs 只能作为
行为 oracle 或迁移 fallback，不能继续堆新的 Grok parity 功能，也不是默认 UI
入口。

## Grok 复用规则

复用等级如下：

- A0：固定 snapshot 原样复制；
- A1：路径/import/别名/少量 DTO 接线；
- B：结构性适配，但保留 Grok 组件、状态机、不变量、测试和视觉规则；
- C：只对 DSH 真源、协议、身份、DeepSeek 字段自建；
- D：排除 Grok agent、shell、tools、ACP、auth、persistence、telemetry 和
  foreign session runtime。

每个新 UI 模块进入前，必须回答：

~~~text
Grok 对应源码和测试：
复用等级：
DSH-neutral seam：
新增 DTO/Intent/Effect：
稳定 ID 和 generation：
将替换的旧模块：
失败测试、PTY/golden 和回滚：
~~~

vendor 修改必须更新 SOURCE_MANIFEST.md、hash、许可证/NOTICE 和本地修改说明。
不得把 Grok runtime crate 引入 DSH 生产依赖。

## 异步、身份和效果

- session、request、generation、sequence、render entry、queue item 和
  interaction 都要有稳定 identity；
- replay/live、attach/detach、reconnect、partial replacement 和 queue revision
  必须由 host authoritative state 收敛；
- UI local draft、focus、hover、pending indicator 不能冒充最终真源；
- 任何 effect 都要经过 Intent → Effect → receipt/notification，不能因重绘重复
  发送 RPC；
- stale/gone/duplicate/unsupported/error 必须显式进入诊断或 Grok status/modal；
- view 不得直接持有 RpcTransport、解析 RPC JSON 或修改 session log。

## 验证门禁

按变更风险运行最小相关集合，并在进度记录中写出实际命令：

~~~bash
cargo fmt --all -- --check
cargo check --workspace
cargo test --workspace --locked
cargo clippy --workspace --all-targets --all-features -- -D warnings
python3 scripts/check-protocol-fixtures.py
python3 scripts/pty-smoke.py --binary target/debug/dsh-pager
~~~

UI 变更还应覆盖 semantic buffer、geometry、focus、wrap、scroll、mouse、
paste、resize、terminal restore 和必要的 mock/real Harness 场景。不要只用
最终文字或 ANSI 字节声称视觉 parity。

### 浏览器终端截图对比（用户可见 TUI 变更强制）

完成任何用户可见的 TUI 功能后，必须使用 `ttyd + xterm.js + Playwright`
在真实浏览器渲染链路中，把 DSH TUI 与本机 Grok TUI 驱动到相同语义状态并
进行截图对比。布局、颜色、字体属性、间距、边框、glyph、wrap、scroll、
focus、hover、selection、overlay、modal、picker、queue、prompt、status、
timeline、mouse、paste 或 resize 有变化时都适用；未完成此步骤不能声明功能
完成或视觉 parity。

标准链路如下：

1. 分别通过 `ttyd` 启动 `target/debug/dsh-pager` 和 Grok 基准
   `/home/leo/.grok/bin/grok`。`ttyd` 负责 PTY/WebSocket，浏览器使用其内置
   xterm.js 前端；禁止使用应用内 `/cut`、重绘 terminal buffer 或桌面截图
   代替浏览器最终像素。
2. 双方固定相同的 viewport、`deviceScaleFactor`、终端行列、字体、字号、
   theme、`TERM=xterm-256color`、locale 和相关工具版本。推荐基线为
   `1200x800`、DPR 1、`DejaVu Sans Mono` 16px；偏离时必须记录原因。
3. DSH 优先使用确定性的 mock backend/fixture；Grok 使用能够表达同一 UI
   状态的命令和输入序列。禁止拿无关业务数据做逐像素结论。无法构造相同数据
   时，仍须比较 geometry 和视觉角色，并把仅由数据造成的差异单独列出。
4. Playwright 必须聚焦 `.xterm-helper-textarea`，通过真实 keyboard、mouse、
   paste 和 viewport resize 驱动交互，并截取 `.xterm` 元素，而不是应用内部
   的抽象视图。可启用 `screenReaderMode=true` 并读取 accessibility tree 等待
   状态，但语义文本不能替代像素证据。
5. 截图前必须等待稳定帧。光标闪烁、动画或增量 overlay 存在时，使用

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gitofxiongxiong/dsh-tui-grok](https://github.com/Gitofxiongxiong/dsh-tui-grok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
