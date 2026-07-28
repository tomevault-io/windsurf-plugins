---
trigger: always_on
description: > **AGENT.md 是导航，不是真实来源；与代码冲突时以代码为准。**
---

# AGENT.md — RSSH 仓库导航

> **AGENT.md 是导航，不是真实来源；与代码冲突时以代码为准。**
> 行号、数量、签名都会变；不要把这里的字面量复制进 PR。每条声明都给了**核对命令**，自己跑一遍再用。
> 所有方案，必须是行业规范，不能走偏门，歪门邪道必须跟用户沟通后。

---

## 置顶规则（其他 AI 必读，违反就回滚）

这些是反复踩出来的硬约束。不是建议。

### R1. 后端事件命名：`<domain>:<event>:<instanceId>`

不带实例 ID 后缀的全局事件会在多 tab / 多传输并存时互相串话。第三段按事件实际作用域使用 session、prompt 或 transfer ID；不要一律假定它是 tab ID。

```bash
rg 'emit\(|format!\("[a-z]+:' src-tauri/src   # 核对最终 channel 的构造位置
```

### R2. CLI ↔ GUI 走 OSC 7337，不要再造 IPC

GUI 内嵌终端跑 `rssh-cli` 时，CLI 通过 OSC 7337 转义序列与 GUI 通信，xterm parser 解码后调 store。要扩通信，加个 OSC kind，**不要**起 socket / pipe / tauri event。

```bash
rg 'OSC_RSSH_ID|7337' src src-tauri/src/bin
```

### R3. 新增前端 command 必须检查两个后端适配器

Tauri 路径需要在 `commands/*.rs` 实现并加入 `src-tauri/src/lib.rs` 的 `generate_handler!`。同一前端功能若要在 JetBrains / browser headless 模式可用，还必须在 `src-tauri/src/server.rs` 的 dispatcher 中暴露同名 command，并尽量复用同一 domain helper。漏任一目标适配器，那个目标运行时就是 "command not found"。

```bash
rg 'generate_handler!' src-tauri/src/lib.rs
```

### R4. Tab 根容器必须明确尺寸与滚动所有权

`.pane.visible` 是 `position: absolute; inset: 0; display: flex; flex-direction: column`。直接子树必须填满它，并明确由哪一层滚动。普通滚动页面沿用 HomeScreen 的三件套：

```css
flex: 1;
overflow-y: auto;
min-height: 0;   /* 缺这条 flex 子元素不收缩，overflow 失效 */
```

终端、编辑器、Forward 这类固定画布可以用 `height: 100%`，把 overflow 留给内部 xterm / editor / list；不要盲目给根节点加 `overflow-y: auto`。共同不变量是：flex 子项需要收缩时必须有 `min-height: 0`，而且滚动只能有一个清晰 owner。

### R5. Secret 不进 DB 明文，只走统一的加密存储

调用方只能走 `SecretStore` (`src-tauri/src/secret/`)。`HybridStore` 用 ChaCha20-Poly1305 加密后把密文写入 DB `secrets` 表。首次选择 backend 时，系统 keychain 可用就把 master key 放 keychain，否则（包括 Android 和部分 headless 环境）放 data dir 下的 `master.key`；选择结果是 sticky 的，已选 keyring 后 keychain 失效必须硬失败，不能静默换新 file key。不要直接读写 `secrets` 表，也不要把原 secret 当成 keychain value 的当前架构。

```bash
rg 'secret_store|SecretStore' src-tauri/src
```

### R6. 不要建"分析文档" / "实施计划归档"

工作流要求临时 `IMPLEMENTATION_PLAN.md` 时，用完即删。不要把过程记录沉淀成新的 `NOTES.md` / `ARCHITECTURE.md` / `DESIGN.md`；除非用户明确要长期文档，仓库导航就维护这一份。

### R7. Svelte 5 runes only

`$state` / `$derived` / `$effect` / `$props`，事件 `onclick={fn}`。看到 `$:` / `export let` / `on:click` ——拒绝合并，让作者升级。

### R8. 跨页 state 放进对应 store，不放组件单例

Tab、导航和连接会话协调在 `app.svelte.ts`；AI、主题、快捷键、传输等分别在既有 domain store。沿用私有 `$state` + 导出 getter / action 的边界，**不要**导出裸 `$state` 对象，**不要**在组件里新建跨页全局状态，也不要把所有领域硬塞回 `app.svelte.ts`。

```bash
rg '^let _.*\$state|^export function' src/lib/stores src/lib/ai src/lib/themes
```

### R9. 平台条件统一走 `cfg` / `app.isMobile`

OS / 设备形态分支：Rust 端用 `#[cfg(...)]`，前端用 `app.isMobile`（UA 嗅探，顶层 const），不要在各组件重复造一套判断。Docker CLI、kubectl、keychain 等外部能力是否可用，仍应在运行时真实探测。

```bash
rg 'cfg\(target_os|isMobile' src src-tauri/src
```

### R10. 新增功能必须显式考虑四条入口

每个新 feature / UI 改动，PR 描述里至少写清以下入口各自怎么处理：

- **桌面 GUI**：默认目标，必须可用
- **移动 GUI**：`app.isMobile` 路径。没右键、没快捷键、没多窗口、屏幕窄。要么适配（`MobileKeybar` 加按钮、长按代替右键），要么显式声明"移动端不提供"
- **CLI**：`src-tauri/src/bin/rssh/`。CRUD 类操作大概率要补；纯 UI/可视化类可声明 N/A
- **Headless / JetBrains**：`src-tauri/src/server.rs` + `src/lib/ipc-shim.ts`。复用同一前端，但 command / event 需要 server adapter 支持

允许的结论是“全部支持”或“只在 X 入口，因为 Y”。**不允许**的是没想过——上线后才发现移动端按钮够不着、CLI 改了 schema 但读不出新字段、JetBrains 页面只能报 unknown command。

```bash
rg 'isMobile' src/lib/components       # 看现有移动端分支怎么写
rg '#\[cfg\(' src-tauri/src/commands   # 看 command 层平台分支
rg '"[a-z_]+" =>' src-tauri/src/server.rs  # 看 headless dispatcher
```

### R11. Transport session 必须经过 lifecycle registry

SSH / PTY / serial / Telnet / SFTP / forward 的 open 路径必须先在 `commands/lifecycle.rs` 预留规范 UUID、绑定 `SessionOwner` 与 nonce，再把 Ready handle 激活进 typed map。取消、关闭、窗口销毁和 reload reconcile 都依赖这份 registry；不要绕过它直接向 `sessions` / `pty_sessions` 等 map 插 handle。

```bash
rg 'reserve_resource|\.activate\(|reconcile_owner|close_owner' src-tauri/src/commands src-tauri/src/server.rs
```

### R12. 动态发现只持久化 source，不持久化结果

`dynamic_discovery_sources` 是用户配置，进入 DB 与 sync；Docker container / K8s pod 的发现结果是瞬时 launch target，只能转成 connector-backed PTY tab。不要偷偷写成 Profile，也不要让已消失的容器在 Home 留陈旧数据。

```bash
rg 'dynamic_discovery_sources|DynamicDiscoveredTarget|connectDynamicTarget' src src-tauri/src
```

---

## 事实（Facts）— 文件 + 概念 + 核对命令

### 二进制与 crate 布局

| 概念 | 在哪 | 怎么验证 |
|---|---|---|
| GUI 二进制 `rssh` | `src-tauri/src/main.rs` | `cat src-tauri/Cargo.toml` 看 `[[bin]]` |
| CLI 二进制 `rssh-cli` | `src-tauri/src/bin/rssh/main.rs` + `commands/`，gated by feature `cli` | 同上 |
| Headless 二进制 `rssh-server` | `src-tauri/src/server_main.rs`，gated by feature `server` | 同上；给 browser / JetBrains 提供 embedded HTTP + WebSocket IPC |
| 共享 lib `rssh_lib` | `src-tauri/src/lib.rs`，`[lib] name="rssh_lib"` | `rg 'rssh_lib::' src-tauri/src/bin/rssh` |

### 前端

| 概念 | 在哪 | 怎么验证 |
|---|---|---|
| Tab / 导航 / 会话协调 store | `src/lib/stores/app.svelte.ts` | 搜 `export function` |
| 领域 store | `src/lib/{ai,themes}/` + `src/lib/stores/` | AI、主题、快捷键、传输等各自维护 |
| Tab 渲染分发 | `src/lib/components/AppShell.svelte` | 搜 `tab.type === ` |
| 终端层 | `src/lib/components/TerminalPane.svelte` | 单文件，xterm + 高亮 + auth 全在内 |
| OSC 解码 | `src/lib/osc/handler.ts` | `registerRsshOscHandlers` |
| Headless IPC shim | `src/lib/ipc-shim.ts` | Tauri 外安装兼容 `invoke` / `listen` 的 WebSocket adapter |
| 键盘注册表 | `src/lib/keyboard/registry.ts` | `attachShortcuts` |
| i18n | `src/lib/i18n/index.svelte.ts` + `locales/{en,zh}.ts` | `t('key')` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shihuili1218/rssh](https://github.com/shihuili1218/rssh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
