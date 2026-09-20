---
trigger: always_on
description: llm-mock 是**脚本化的模型 API mock**（Bun 运行时，唯一依赖 hono）：把预设的响应序列写成 JSON
---

# CLAUDE.md

## 项目定位

llm-mock 是**脚本化的模型 API mock**（Bun 运行时，唯一依赖 hono）：把预设的响应序列写成 JSON
脚本，**第 i 次请求返回第 i 条**；客户端要流式时把同一条完整响应转换成对应协议的 SSE 序列
（Chat / Messages / Responses 按 body 的 `stream: true`，Gemini 那份写在路径的
`:streamGenerateContent` 上——见「关键约定与陷阱」）。

同一份脚本可以走四种线协议（各自独立计游标，互不干扰）：

| 端点 | 协议 | 谁在用 |
| --- | --- | --- |
| `POST /v1/chat/completions` | OpenAI Chat Completions | peri、opencode、opencode2、pi、dsh、MiniMax Code、Hermes Agent、Cline、脚本自测 |
| `POST /v1/messages` | Anthropic Messages | Claude Code |
| `POST /v1/responses` | OpenAI Responses | Codex |
| `POST /v1beta/models/{model}:generateContent` | Google Gemini API | Antigravity CLI |

两个用途：

- **性能压测**：以脚本控制的节奏驱动 harness（peri / Claude Code / Codex / pi / dsh / MiniMax Code /
  Antigravity CLI / opencode2 / Hermes Agent / Cline；opencode **v1** 已退出排名、不再跑，见
  「与 harness 集成」开头），测量 harness 进程自身的 CPU / 内存开销（不采 GPU）；
- **功能测试**：不调用真实模型，复现 agent 的多轮循环、工具调用与流式渲染。

## 压测工作流（已实现）

一条命令跑完「起 mock → 起 harness → 每 100ms 采样 → 出记录」：

```sh
bun run scripts/perf/run.ts --script data/scenarios/long-run.json --exhausted stop --timeout-ms 600000
bun run scripts/perf/run.ts --help                      # 全部选项（--script 必填，没有默认剧本）

cd playground/peri        && bun perf-demo.ts --timeout-ms 600000   # peri 沙盒（默认长剧本 + stop）
cd playground/claude-code && bun perf-demo.ts --timeout-ms 600000   # Claude Code 沙盒
cd playground/codex       && bun perf-demo.ts --timeout-ms 600000   # Codex 沙盒
cd playground/pi          && bun perf-demo.ts --timeout-ms 600000   # pi 沙盒
cd playground/deepseek    && bun perf-demo.ts --timeout-ms 600000   # dsh 沙盒
cd playground/minimax-code && bun perf-demo.ts --timeout-ms 600000  # MiniMax Code（mcode）沙盒
cd playground/antigravity && bun perf-demo.ts --timeout-ms 600000   # Antigravity CLI（agy）沙盒
cd playground/opencode2   && bun perf-demo.ts --timeout-ms 600000   # opencode v2（bin 名 opencode2）沙盒
cd playground/hermes      && bun perf-demo.ts --timeout-ms 600000   # Hermes Agent（hermes）沙盒
cd playground/cline       && bun perf-demo.ts --timeout-ms 600000   # Cline（cline）沙盒
# cd playground/opencode  && bun perf-demo.ts --timeout-ms 600000   # opencode v1 已退出排名：代码保留，常规批次不再跑
```

`perf-demo.ts` 都是复用同一套实现的薄入口（相对路径按仓库根解析），差别只在 harness 命令、
沙盒与配置注入方式（详见「与 harness 集成」）。**默认剧本是各家的长剧本**
（`data/scenarios/long-run*.json`，由 `gen-long-run.ts` 按自家工具形状生成），`--exhausted` 默认
`stop`；`run.ts` 的 `--script` 是必填（见「关键约定与陷阱」），那份默认值因此由各家 demo 自己带：

- `playground/peri`：默认注入 `--db-path`（沙盒会话库）与 `--settings`（运行时生成、指向本次端口的 JSON）；
- `playground/opencode`（v1）：`XDG_*` 隔离 + `{env:LLM_MOCK_BASE_URL}` 变量替换（换端口不用改配置）；
- `playground/opencode2`（v2）：同样是 `XDG_*` + `{env:…}`，外加 XDG_CONFIG_HOME、一条死代理，
  命令固定带 `--standalone`（**不加会留一个常驻后台服务，三次读数冷热不均**）；二进制按
  **`opencode2` 这个 bin 名**找（裸 `opencode` 已被 v2 顶掉，认名字才不会拿错代）；
- `playground/claude-code`：`HOME` + `CLAUDE_CONFIG_DIR` 都指到沙盒（**只改后者挡不住用户级 settings**）；
- `playground/codex`：`CODEX_HOME` 指向沙盒（用户全局配置里有 hooks 与别的 provider）；
- `playground/pi`：`PI_CODING_AGENT_DIR` 指向沙盒，`models.json` 每次启动按本次端口重写
  （pi 的 `baseUrl` 不吃 `$VAR` 插值，换端口只能改文件）；
- `playground/deepseek`：`DSH_HOME` 指向沙盒，provider 全走环境变量
  （`$DEEPSEEK_BASE_URL` / `$DEEPSEEK_API_KEY`），**不用生成配置文件**；
- `playground/minimax-code`：`MINIMAX_DATA_DIR` 指向沙盒，provider 按本次端口写进沙盒的
  `config.yaml`（mcode 的 `baseURL` 不吃环境变量插值，与 pi 同理）；
- `playground/antigravity`：`HOME` 指向沙盒（`$HOME/.gemini/antigravity-cli/settings.json` 里选
  `modelProvider: gemini`），端点靠 `GOOGLE_GEMINI_BASE_URL`、凭据靠 `GEMINI_API_KEY` 假值
  （agy 的 provider 配置与登录态都只按 HOME 找，与 Claude Code 同理）；
- `playground/hermes`：`HERMES_HOME` 指向沙盒（config.yaml / .env / sessions / state.db / skills
  全从这里找），provider（`model.provider: custom` + `base_url`）按本次端口重写那份 `config.yaml`
  （没有环境变量插值这一说，与 pi / mcode 同理）。**代码不在 HERMES_HOME 下**——官方安装脚本把仓库
  放在 `~/.hermes/hermes-agent`、`~/.local/bin/hermes` 是固定指向它的启动壳，所以换 HERMES_HOME
  只换数据、不动代码。
- `playground/cline`：`--config` / `--data-dir` / `--hooks-dir` 三个位置参数都指到沙盒，
  provider 按本次端口写进 `<沙盒>/data/settings/providers.json`（不吃环境变量插值，与 pi / mcode /
  hermes 同理）。**不用换 HOME**（实测用户级 `~/.cline` 抢不走配置）；**也不注入死代理**——实测那样
  反而把它拖慢 4~5 倍（见「与 harness 集成」的 Cline 一节）。

需要复核采样口径时跑 `bun run scripts/perf/verify.ts`（对 `yes` / `sleep` 这类已知负载回归，
并打印两个候选后端的开销与分辨率）。想把「CPU 与内存」混成一个可比的数（谁跑完同一部剧本烧的资源
更少）看**统一计分**——公式结构借自阿里云 FC，**系数是本项目定的 CPU 与内存 1:1**
（`CU = 1.0 × 核·秒 + 1.0 × GB·秒`），另给一列不折算的**峰值**（最坏一刻占多少）；**还在 Beta：
口径没定稿，先用来看趋势、别把名次当结论**，见下面的「统一计分（Beta）：CPU 与内存 1:1」。

### 场景：长剧本端到端（跑完整个剧本，测时长）

`--exhausted stop` 让 mock 在剧本走完后返回一条「任务结束」纯文本（`finish_reason=stop`），
harness 收到即自行收尾退出——于是能测**端到端时长**（含启动，`perf.log` 里的「端到端时长」）
与整段资源消耗，而不是某段固定时间窗内的资源写照。剧本由生成器现造（仓库里不放剧本文件），
生成器**固定带两条收尾条**（轮数之外）：一条同文的「任务结束」文本 + 一条空白响应——后者是给
peri 的「预测下一步输入」的，能消掉它固定 5.0s 的收尾等待（理由见「已知限制与坑」）：

```sh
# 各家各一份（工具名/参数形状按各家实测，见 gen-long-run.ts 的 ArgShape）
bun run scripts/perf/gen-long-run.ts --turns 100 --out data/scenarios/long-run.json           # peri / opencode / Claude Code（Bash + command）
bun run scripts/perf/gen-long-run.ts --turns 100 --args exec --out data/scenarios/long-run-codex.json
bun run scripts/perf/gen-long-run.ts --turns 133 --tool bash --out data/scenarios/long-run-pi.json  # pi 要 133：压缩请求每轮多吃一条
bun run scripts/perf/gen-long-run.ts --turns 100 --tool bash --args command+description \
  --out data/scenarios/long-run-dsh.json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KonghaYao/harness-perf-benchmark](https://github.com/KonghaYao/harness-perf-benchmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
