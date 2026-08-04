---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目定位

open-claude-router 是一个**路由和会话无状态**的 Anthropic Messages API ↔ OpenAI 协议（Chat Completions / Responses）转换服务。所有上游信息（URL、Authorization、模型名）由请求方逐请求传过来，服务端不读 provider 配置、不存任何凭证。客户端通过 HTTP header `X-Upstream-Format` 选择上游协议变体（不传或 `chat-completions` = 默认；`responses` = OpenAI o-series / gpt-5 原生协议）。服务默认把模型侧请求/响应写入保留 7 天的运维审计日志，但不写 Authorization；详见 README 的“模型交互日志”。

## 常用命令

- `npm run dev` — tsx watch 启动，默认监听 `:3457`
- `npm run typecheck` — `tsc --noEmit`，改动后必跑
- `npm test` — Node test runner 自动化回归套件
- `npm run test:stream` — 流式集成/回归测试
- `npm run test:live` — 使用 `OCR_LIVE_*` 环境变量运行真实 Chat / Responses 上游矩阵
- `npm run build` — esbuild 打包成 `dist/server.js` 单文件
- `npm start` — 跑 build 产物
- `docker buildx build --platform linux/amd64,linux/arm64 -t riba2534/open-claude-router:latest --push .` — 推 Dockerhub（多架构）

完整验收先执行 `npm ci`，再执行 `npm run typecheck && npm test && npm run test:stream && npm run build`。协议转换改动必须补对应 fixture；需要真实上游 canary 时使用 `scripts/verify-live-upstream.ts`，不得把 endpoint、模型名或凭证写进仓库。

## 高层架构

### 两种客户端接入模式 + 一个协议选择 header

服务的两种接入模式 + 协议选择 header 是相互正交的——任意组合都成立：

| 路由 | mode | 上游凭证来源 | 服务自身鉴权（仅 `OCR_ACCESS_TOKENS` 启用时） | `X-Upstream-Format` |
|---|---|---|---|---|
| `POST /v1/messages` | header 模式 | `X-Upstream-Authorization` header | `Authorization: Bearer <service token>` | 可选 |
| `POST /*` catch-all（path 以 `/http(s)://` 开头） | embedded-path 模式 | `Authorization: Bearer <upstream value>`（剥 Bearer 前缀） | `X-OCR-Token` header（因为 Authorization 被上游凭证占用） | 可选 |

两条路径都汇入 `src/routes/messages.ts` 的 `forwardMessages()`。`src/utils/auth.ts` 里：`parseUpstreamConfig`（header 模式上游解析）、`parseUpstreamFromEmbeddedPath`（embedded-path 模式上游解析）、`parseUpstreamFormat`（协议变体），鉴权用 `checkServiceAuth`（header 模式 Bearer）和 `checkServiceAuthFromOcrTokenHeader`（embedded-path 模式 X-OCR-Token）。模型名与额外 header 由 `parseModelMap` + `resolveUpstreamModel`（`X-Upstream-Model-Map` 映射 / `X-Upstream-Model` 覆盖 → `unified.model`）和 `parseUpstreamHeaders`（`X-Upstream-Headers` 白名单 → 转发给上游，含 protected-header / 原型污染防注入）解析；这些 header 两种接入模式都可用，完整清单见 README 的"请求头"表。

> 术语：**unified** = 内部统一请求/响应形态，等同 OpenAI Chat Completions；**embedded-path 模式** = README 里的"path 模式 / 方式 A"（上游 URL 拼在请求 path 里）；**vendor** = 把上游 [musistudio/claude-code-router](https://github.com/musistudio/claude-code-router) 的 transformer 源码原样拷入本仓库维护。

### 协议转换核心：双 transformer 协作

服务有两个 transformer 实例，按对称方向分工：

| Transformer | 文件 | 方向 | 何时介入 |
|---|---|---|---|
| `AnthropicTransformer` | `src/transformers/anthropic.ts` | 客户端方向：Anthropic ↔ unified（unified 形态接近 OpenAI Chat Completions，并含内部保真 envelope） | **永远介入** |
| `OpenAIResponsesTransformer` | `src/transformers/responses.ts` | 上游方向：unified ↔ OpenAI Responses 协议 | 仅当 `X-Upstream-Format: responses` |

请求处理流水线（`forwardMessages`）：

```
client body (Anthropic Messages)
  ↓ anthropic.transformRequestOut
unified  (request.thinking → result.reasoning；保留 cache_control；
           document → 内部 file envelope；tool_result 保留 typed 多 block；
           每条 assistant 的 signed thinking 块 → message.thinking)
  ↓ 应用上游模型覆盖  (路由层已用 resolveUpstreamModel 按 X-Upstream-Model-Map /
                       X-Upstream-Model 算出 upstream.model，此处写入 unified.model)
  ↓ scrubAnthropicOnlyFields  (always: 只剥 protocol wrapper 位置的 cache_control，
                               不递归破坏用户 JSON Schema 同名字段)
  ↓ 分支:
      format=responses:
        scrubResponsesReasoningArtifacts  (剥 Chat 专用 reasoning_content；
                                            signed thinking 由 Responses transformer 消费)
        → responses.transformRequestIn    (消费 unified.reasoning 转成
                                            Responses reasoning:{effort,summary}；
                                            file → input_file；tool output 保留 typed 数组)
      format=chat-completions (默认):
        convertThinkingToReasoningContent  (assistant thinking → reasoning_content；
                                            reasoning 启用时给带 tool_calls 的消息
                                            兜底补 reasoning_content，满足 DeepSeek
                                            类上游"thinking 启用必带 reasoning_content")
        → normalizeMultimodalToolResultsForChatCompletions
                                           (tool image/file → user sidecar；
                                            file_data/file_id 保留，URL-only file 转文本)
        → scrubChatCompletionsIncompatibleFields  (剥顶层 reasoning)
        → [若流式] 注入 stream_options:{include_usage:true}
upstream-shaped body
  ↓ fetch upstream  (callUpstream 构造全新 headers：Content-Type/Authorization/
                     Accept + X-Upstream-Headers 白名单；不 spread 客户端 header)
upstream response
  ↓ [if format=responses] responses.transformResponseOut
unified-shaped response
  ↓ anthropic.transformResponseIn  (含上游 reasoning_content → Anthropic
                                    thinking 块归一化；流式路径额外合成 signature 封口；
                                    incomplete tool call 不产生 tool_use 成功终态)
  ↓ src/utils/anthropic-sse.ts  (客户端 stream 标志决定最终 SSE / JSON 形态；
                                 JSON↔SSE 转换保留 stop_reason/stop_details/usage)
client SSE / JSON
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [riba2534/open-claude-router](https://github.com/riba2534/open-claude-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
