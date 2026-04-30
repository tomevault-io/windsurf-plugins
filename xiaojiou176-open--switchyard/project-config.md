---
trigger: always_on
description: 这份文件是 `Switchyard` 仓的 repo 宪法。
---

# Switchyard AGENTS.md

## 文档目的

这份文件是 `Switchyard` 仓的 repo 宪法。  
它不是重复全局 `AGENTS.md`，而是把 **Switchyard 自己已经拍板的产品边界、上游关系、参考仓纪律、交付顺序** 固化下来，防止未来 Agent 再把项目做偏。

如果根目录的其他说明、口头讨论、实现尝试与本文件冲突，以本文件和 `.agents/internal-docs/adr/`、`.agents/internal-docs/contracts/` 为准。

---

## 一句话定义

> `Switchyard` 是一个面向 AI 产品开发者的共享 Provider Runtime。  
> 它的目标不是做另一个聊天产品或 personal assistant，而是把终端用户已有的 AI 访问资格统一转成可被 AI 产品消费的共享内核。

---

## 真理源顺序

后续任何实现、设计、修文，遵守以下真理源顺序：

1. `.agents/internal-docs/adr/*.md`
2. `.agents/internal-docs/contracts/*.md`
3. `.agents/internal-docs/blueprints/*.md`
4. `.agents/internal-docs/product/*.md`
5. `docs/runbooks/*.md`
6. `README.md`
7. 本地参考仓与历史对话记录

解释：

- `ADR` 决定边界和裁决
- `contracts` 决定语义和公开语言
- `blueprints` 决定阶段和执行顺序
- `product` 决定为什么存在、为谁服务
- 参考仓和对话记录只能提供**依据**，不能直接覆盖 `Switchyard` 自己的合同

---

## [已确认] V1 边界

### V1 只做两条供给 lane

- `BYOK`
- `Web/Login`

### V1 当前不做

- `Agent Input Lane`
- `Codex` 作为输入来源
- `Claude Code` 作为输入来源
- `Gemini CLI`
- 同 provider 多账户网页登录池化
- 自动轮转
- 自动换号
- 隐式 failover
- control-plane 主线
- raw fork 产品

### V1 固定网页登录 provider

1. `ChatGPT`
2. `Gemini`
3. `Claude`
4. `Grok`
5. `Qwen`

其中高稳定目标为：

- `ChatGPT`
- `Gemini`
- `Claude`

### 凭证边界

> **永远只接受终端用户自己的凭证。**

这意味着：

- 不做平台共享账号池
- 不做开发者公共凭证
- 不做平台代持型“公共 AI 资格”

---

## [已确认] 产品身份

### 长期身份

`Switchyard` 长期可以是 `Hybrid`：

- `Provider Runtime`
- `Agent Runtime`

### V1 实际身份

`Switchyard V1` 实际只聚焦：

> **共享 Provider Runtime**

### consumer compat 后置

以下对象保留为后续 consumer compat 目标，但不属于 V1 核心施工面：

- `Codex`
- `Claude Code`
- `OpenClaw`

顺序固定为：

1. 先做 `Switchyard` 自己的 kernel
2. 再做你的 3 个 repo 接入
3. 再做 `Codex / Claude Code / OpenClaw`

---

## [已确认] 架构纪律

### 必须分开的四个维度

后续任何设计与实现都必须把这 4 个维度拆开：

- `lane`
- `provider`
- `consumer`
- `surface`

不允许把：

- provider 写成 consumer
- consumer 写成 provider
- surface 写成另一套 runtime 语言
- future compat 提前写进当前 V1 核心

### 当前正式骨架

当前正式骨架是：

- `apps/`
- `packages/contracts`
- `packages/kernel`
- `packages/sdk`
- `packages/credentials`
- `packages/diagnostics`
- `packages/lanes/*`
- `packages/providers/*`
- `packages/consumers/*`
- `packages/surfaces/*`

具体目录与 Day 1 范围，以：

- `.agents/internal-docs/adr/0004-architecture-skeleton-monorepo.md`

为准。

---

## 上游关系纪律

### 总原则

> **技术上深借，产品上独立。**

### `openclaw-zero-token`

它的定位已经正式拍板：

> **技术母本，不是产品母本。**

这意味着：

- 可以深度研究
- 可以运行 sidecar
- 可以 selective transplant
- 可以迁移 Web/Login runtime、auth/session/refresh、gateway 逻辑

但不允许：

- 公开 raw fork
- 继承它的产品身份
- 继承 personal assistant worldview
- 继承 channels / mobile companion / operator-first product baggage

当前工作区内，它的本地研究根路径按占位写法记为：

`<local-reference-root>/openclaw-zero-token`

其中 `<local-reference-root>` 表示你在本机存放第三方参考仓的根目录；公开文档里不要写个人绝对路径。

在 `Web/Login` 这条 lane 上，后续任何实现、修补、Reality Gate 收口、acquisition 主线推进，都默认优先深读这个本地路径，而不是先去看那些更小、更旧的局部样本仓。

### `Vercel AI SDK`

定位：

- SDK/contracts/byok abstraction 骨架

形式：

- `pnpm` 直接依赖优先

### `LiteLLM`

定位：

- BYOK gateway / sidecar / routing 样本

形式：

- `lab sidecar`
- 不作为 TS 内核骨架直接吃进去

### `ChatALL`

定位：

- 产品层样板、能力矩阵样板

形式：

- `reference only`

### 当前非主线参考

以下仓当前不是 V1 内核主线：

- `codex`
- `claude-code`
- `openclaw`
- `gemini-cli`
- `aider`
- `opencode`
- `oh-my-openagent`

### 明确排除

当前已经被正式移出主线的历史小样本仓，不再进入文档叙事，也不作为当前主线参考。

如果后续有人想重新把这些历史小样本仓拉回主线，必须先改 ADR，不能静默复活。

---

## 实现纪律

### 先看什么

开始任何代码前，必须先读：

1. `.agents/internal-docs/product/v1-brief.md`
2. `.agents/internal-docs/product/scope-and-nongoals.md`
3. `.agents/internal-docs/adr/0001-v1-boundary-and-lane-model.md`
4. `.agents/internal-docs/adr/0002-external-repo-adoption-matrix.md`
5. `.agents/internal-docs/adr/0003-upstream-relationship-openclaw-zero-token.md`
6. `.agents/internal-docs/adr/0004-architecture-skeleton-monorepo.md`
7. `.agents/internal-docs/contracts/provider-runtime-contract.md`
8. `.agents/internal-docs/contracts/auth-accounts-and-credentials.md`
9. `.agents/internal-docs/contracts/service-and-sdk-surfaces.md`
10. `.agents/internal-docs/blueprints/v1-delivery-plan.md`
11. `.agents/Tasks/TASK_BOARD-*.md`

### 先做什么

后续实现顺序必须遵守：

1. `Kernel Alpha`
2. `Kernel Beta`
3. `First-party Integration`
4. `Consumer Compat`

### 当前最核心的 done signal

在没有 fresh verification 的前提下，不允许说：

- 已完成
- 已打通
- 已稳定

所有完成性结论必须绑定当前轮实际验证结果。

### 产品性格

失败策略必须优先：

1. 明确报错
2. 明确诊断
3. 把选择权交给用户

而不是：

- 偷偷切 provider
- 偷偷换账号
- 平台暗箱兜底

---

## 浏览器与运行时卫生纪律

### 资源卫生是正式约束，不是“顺手注意”

凡是触发 `Chrome / Chromium`、profile、browser bootstrap、live proof、CDP attach、support bundle、runtime cache、Docker sidecar 的任务，都必须把**资源卫生**当成正式 DoD 的一部分。

### 浏览器实例预算（硬上限）

1. 当前机器浏览器实例总数超过 **6** 时，当前 repo **禁止**再新开实例。
   - 先回收当前 repo 自己的实例。
   - 若明显属于别的 repo 或归属不明，标记 `other-repo-owned` / `owner-unknown`，不要越权清理。

2. 当前 repo 自己默认只允许：
   - **1 个主浏览器实例**
   - 必要时额外 **1 个短时诊断实例**

3. 当前 repo 自己的标签页上限是 **3 个**。
   - 超过就先关，不准靠堆 tab 排障。

4. 当前 repo 新克隆 / 新建的临时 profile，默认预算是 **0**。
   - 只有明确必要时才允许临时建。
   - 同轮任务结束前必须清理或明确留下理由。

5. 对同一 provider 的登录态检查 / 恢复预算是 **1-2 次定点尝试**。
   - 两次都证明当前 repo 自己的 canonical browser/profile 没有有效登录态后，就应收成 blocker。
   - 不准继续靠重复 clone profile 或多开浏览器碰运气。

### 必须遵守的规则

`worker-safe` 是当前 repo 的默认模式。禁止 `killall`、`pkill`、`killpg(...)`、非正 PID 信号、`loginwindow` / Force Quit API，以及 AppleScript / `System Events` 应用控制路径；清理只允许命中当前 repo 自己记录的正 PID、browser root、profile、CDP 记录或直接持有的 child handle，归属不明就 fail-closed。

1. **先盘点，再开新实例**
   - 开任何 `Chrome / Chromium` 之前，先检查当前机器上：
     - 当前 repo 自己的 browser 进程

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiaojiou176-open/Switchyard](https://github.com/xiaojiou176-open/Switchyard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
