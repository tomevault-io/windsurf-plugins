---
trigger: always_on
description: Spatiotemporal 是论文 [*A Programming Paradigm for Spatiotemporal Composability*](https://github.com/cordiverse/paper) 的 Rust 演算实现；`spatiotemporal-agent/` 是其上的插件化 agent harness，形态对齐 [DeepSeek Harness (dsh)](https://github.com/deepseek-ai/deepseek-harness)：**一切都是插件**。
---

# AGENTS.md

Spatiotemporal 是论文 [*A Programming Paradigm for Spatiotemporal Composability*](https://github.com/cordiverse/paper) 的 Rust 演算实现；`spatiotemporal-agent/` 是其上的插件化 agent harness，形态对齐 [DeepSeek Harness (dsh)](https://github.com/deepseek-ai/deepseek-harness)：**一切都是插件**。

改内核前读根 [README.md](README.md) 的「与论文的对应」和「Rust 里的七个设计决定」；改 agent 前读 [spatiotemporal-agent/README.md](spatiotemporal-agent/README.md)。

---

## 本文件的两种用途

1. **给编码 agent（Cursor 等）**：改这个仓库时遵守下面的布局、命令与约定。
2. **给运行中的 spatiotemporal-agent**：`system-prompt` 插件会把工作区根目录的 `AGENTS.md`（默认文件名，见 `cordis.yml`）注入 model 的 system prompt。在仓库根启动 agent 时，本节与下文「运行中 agent 的行为」会一并生效。

---

## 仓库布局

```
src/                      演算内核（Context、Loader、fiber、Steps、compose）
tests/                    内核与配置层测试（对着论文定理写）
examples/                 swap_provider、watch_config
crates/
  spatiotemporal-wasm/    wasm 基质适配器 + WIT + guest 测试
  spatiotemporal-script/  QuickJS 脚本基质适配器
  spatiotemporal-process/ 子进程（NDJSON stdio）基质适配器
spatiotemporal-agent/     agent harness（宿主 + cordis.yml + 浏览器 UI）
  src/
    host.rs               Toolbox、Llm、Surface、AgentLoop、SystemPrompt
    registry.rs           插件注册表（name → 工厂）
    runtime.rs            AgentRuntime（热对账；会话 patch 栈）
    session.rs            JSONL 会话 + `.patch.json` 持久化
    workspace.rs          工作区目录列表
    workspace_store.rs    工作区切换与 `.agent/workspaces.json`
    plugins/              native 插件实现
    keys.rs               coeffect 键（llm、fs、shell、agent-loop…）
  plugins/                script 叶子（cite.js、echo.js）
  guests/outline/         wasm 叶子（outline）
  cordis.yml              基础组合
  cordis.smoke.yml        CI：echo LLM + probe 界面
  cordis.creation.yml     创造模式 patch
  cordis.coding.yml       编码模式 patch
  assets/                 Web UI、CODING.prompt.md
scripts/                  build-guests.sh（各子项目各一份）
.github/workflows/ci.yml  kernel / msrv / wasm / script / agent 五条 job
```

**边界**

| 层 | 拥有什么 | 不拥有什么 |
|---|---|---|
| 内核 `spatiotemporal` | effect/coeffect、Loader 对账、fiber 生命周期 | HTTP、bash、浏览器、LLM |
| 适配器 `spatiotemporal-wasm/script` | guest 生命周期、WIT/`host.*`、燃料/中断 | 新的 coeffect 种类 |
| Agent `spatiotemporal-agent` | cordis 组合、工具登记、Web、session | 不应「偷偷补齐」内核缺口（见 README 330–342 行） |

---

## 命令

在仓库根目录：

```bash
# 内核（default-members 只含根 crate，日常最快）
cargo fmt --all --check
cargo clippy --all-targets -- -D warnings
cargo test
cargo run --example swap_provider
cargo run --example watch_config

# MSRV 1.94（与 workspace 一致）
cargo +1.94.0 build -p spatiotemporal

# wasm 适配器（需 wasm32-wasip2 + guest 产物）
rustup target add wasm32-wasip2
./scripts/build-guests.sh          # 在 crates/spatiotemporal-wasm 下执行
cargo clippy -p spatiotemporal-wasm --all-targets -- -D warnings
cargo test -p spatiotemporal-wasm

# script 适配器
cargo clippy -p spatiotemporal-script --all-targets -- -D warnings
cargo test -p spatiotemporal-script

# 子进程适配器
./crates/spatiotemporal-process/scripts/build-guests.sh
cargo clippy -p spatiotemporal-process --all-targets -- -D warnings
cargo test -p spatiotemporal-process

# agent（需编 outline.wasm）
./spatiotemporal-agent/scripts/build-guests.sh
cargo clippy -p spatiotemporal-agent --all-targets -- -D warnings
cargo run -p spatiotemporal-agent -- --smoke    # 无 API key、不听端口
export DEEPSEEK_API_KEY=sk-...
cargo run -p spatiotemporal-agent                 # Web http://127.0.0.1:8787
cargo run -p spatiotemporal-agent -- --creation   # 创造模式
cargo run -p spatiotemporal-agent -- --coding     # 编码模式（更多 tool 轮次）
```

**改什么跑什么**：只动内核不必编 wasm/agent；动 agent 插件至少 `--smoke`；动 wasm guest 要 rebuild guests + `cargo test -p spatiotemporal-wasm`。

---

## 密钥与环境变量

| 变量 | 默认 | 用途 |
|---|---|---|
| `DEEPSEEK_API_KEY` | （对话必填） | DeepSeek Bearer token |
| `DEEPSEEK_BASE_URL` | `https://api.deepseek.com` | API 网关 |
| `DEEPSEEK_MODEL` | `deepseek-chat` | 模型名 |
| `PORT` | `8787` | Web 端口（或 `cordis.yml` `ui.config.port`） |
| `WORKSPACE` | 当前工作目录 | fs/bash 沙箱根（`root: .` 时） |

**永远不要**把 key 写进 `cordis.yml`、README 或提交 `.env`。`--smoke` 用脚本 `echo` 代替 DeepSeek，CI 不需要 key。

---

## 架构约定（对齐 dsh，用 Rust 表述）

### 组合，不是 monolith

- 能力来自 `cordis.yml` 每一行；`name` 是**断言**不是赋值——换实现 = `disabled: true` + `insert` 新行（见 `cordis.smoke.yml`）。
- 插件通过 `ctx.set::<Key>`（native）或 `host.registerTool` / `host.register-llm`（guest）贡献能力；卸载时宿主持有登记的逆。
- Agent 侧热对账走 `AgentRuntime::push_layer`，仅创造模式/审批通过后使用，且**只写入当前会话**的 `.patch.json`；不要绕过 Loader 直接改 fiber 树。
- **不要**把会话级 script/wasm 工具写进 `cordis.yml`（全局，所有新会话可见）；用 `define_script` + 审批，或 `save_patch` 导出到 `cordis.patch.yml`。

### 三种基质

| 基质 | 适合 | 不适合 |
|---|---|---|
| **native** | LLM HTTP、Web 界面、fs/bash 沙箱、agent-loop | — |
| **wasm** | 小 payload 叶子工具（outline） | 整段对话进 guest、新 coeffect |
| **script** | 快速试验叶子（cite、echo） | 不可信代码无审批热装 |
| **process** | MCP 桥、已有 CLI guest（NDJSON stdio） | 新 coeffect、大 payload |

**guest 与 IO 的分工（对齐 dsh Code Mode 思路）**

- script/wasm **默认只 grant `markdown`**（只读快照），**不要** grant `fs` / `shell`。
- 需要读文件、跑命令：**你（LLM）直接调** native 的 `read` / `write` / `edit` / `bash` / `web_fetch`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [curtiseng/spatiotemporal](https://github.com/curtiseng/spatiotemporal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
