---
trigger: always_on
description: 这份文档是 `paicli` 仓库给各类 Agent / 新线程使用的首读入口。
---

# AGENTS.md

这份文档是 `paicli` 仓库给各类 Agent / 新线程使用的首读入口。

目标只有两个：

1. 让首次进入仓库的线程，能在几分钟内建立对项目的正确认识。
2. 把后续协作规则沉淀到一个稳定入口，避免规则只存在于历史对话里。

如果本仓库的行为、目录结构、约定或协作方式发生了稳定变化，请在同一次改动里同步更新本文件。

## 信息优先级

当不同文档描述不一致时，按下面的优先级理解：

1. 代码实际行为
2. `AGENTS.md`
3. `README.md`
4. `ROADMAP.md`
5. `CLAUDE.md`

`ROADMAP.md` 代表演进方向，不代表已经交付。

## 项目快照

- 项目名：`PaiCLI`
- 定位：一个教学导向的 Java Agent CLI，目标是从简单 Agent CLI 逐步演进到更完整的 Agent 产品
- 当前主线：已完成第 1 期 `ReAct`、第 2 期 `Plan-and-Execute + DAG`、第 3 期 `Memory + 上下文工程`、第 4 期 `RAG 检索 + 代码库理解`、第 5 期 `Multi-Agent 协作 + 角色分工`、第 6 期 `HITL 人工审批 + 危险操作拦截`、第 7 期 `异步执行 + 并行工具调用`、第 8 期 `多模型适配 + 运行时切换`、第 9 期 `联网能力 + Web 工具`
- 当前用户可感知版本：CLI Banner 显示 `v9.0.0`
- 当前 Maven 产物版本：`pom.xml` 仍是 `1.0-SNAPSHOT`
- 结论：如果你看到运行界面是 `v7.0.0`，但 Jar 名仍是 `paicli-1.0-SNAPSHOT.jar`，这是当前仓库的真实状态，不是你看错

## 运行前提

- Java 17+
- Maven
- 可用的 `GLM_API_KEY`

API Key 当前读取顺序以代码为准：

1. 仓库当前目录下的 `.env`
2. 用户主目录下的 `.env`
3. 环境变量 `GLM_API_KEY`

`.env.example` 当前包含：

```bash
GLM_API_KEY=your_api_key_here
EMBEDDING_PROVIDER=ollama
EMBEDDING_MODEL=nomic-embed-text:latest
EMBEDDING_BASE_URL=http://localhost:11434
# EMBEDDING_API_KEY=your_api_key_here
# PAICLI_LOG_LEVEL=INFO
# PAICLI_LOG_DIR=/Users/yourname/.paicli/logs
# PAICLI_LOG_MAX_HISTORY=7
# PAICLI_LOG_MAX_FILE_SIZE=10MB
# PAICLI_LOG_TOTAL_SIZE_CAP=100MB
```

长期记忆默认持久化位置：

1. `~/.paicli/memory/long_term_memory.json`
2. 如果传入 `-Dpaicli.memory.dir=/path/to/dir`，则优先使用该目录

代码索引（RAG）默认持久化位置：

1. `~/.paicli/rag/codebase.db`
2. 如果传入 `-Dpaicli.rag.dir=/path/to/dir`，则优先使用该目录

Embedding 配置读取顺序（以代码实际行为为准）：

1. 环境变量：`EMBEDDING_PROVIDER`、`EMBEDDING_MODEL`、`EMBEDDING_BASE_URL`、`EMBEDDING_API_KEY`
2. 系统属性（同上）
3. 默认值：`ollama` / `nomic-embed-text:latest` / `http://localhost:11434`

日志配置读取顺序（以代码实际行为为准）：

1. 系统属性：`paicli.log.dir`、`paicli.log.level`、`paicli.log.maxHistory`、`paicli.log.maxFileSize`、`paicli.log.totalSizeCap`
2. 环境变量或 `.env`：`PAICLI_LOG_DIR`、`PAICLI_LOG_LEVEL`、`PAICLI_LOG_MAX_HISTORY`、`PAICLI_LOG_MAX_FILE_SIZE`、`PAICLI_LOG_TOTAL_SIZE_CAP`
3. 默认值：`~/.paicli/logs` / `INFO` / `7` / `10MB` / `100MB`

ReAct / SubAgent 预算配置读取顺序（以代码实际行为为准）：

1. 系统属性：`paicli.react.token.budget`、`paicli.react.stagnation.window`、`paicli.react.hard.max.iterations`
2. 默认值：`300000` / `3` / `50`

LLM HTTP 超时配置读取顺序（以代码实际行为为准）：

1. 系统属性：`paicli.llm.connect.timeout.seconds`、`paicli.llm.read.timeout.seconds`、`paicli.llm.write.timeout.seconds`、`paicli.llm.call.timeout.seconds`
2. 默认值：`60` / `300` / `60` / `600`（单位：秒）

注意：SSE 流式接口下，OkHttp 的 `readTimeout` 是"两次 read 之间最大间隔"而非请求总时长；GLM-5.1 在生成大段 reasoning_content 时服务端可能长时间静默，所以默认值放宽到 300 秒，再用 `callTimeout` 兜底整个请求。

Web 搜索 provider 配置读取顺序（以代码实际行为为准）：

1. 环境变量 / 系统属性 / `.env` 中的 `SEARCH_PROVIDER`：显式指定 `zhipu` / `serpapi` / `searxng`
2. 未指定时按 Key/URL 自动判断（优先级从高到低）：
   - `GLM_API_KEY` 存在 → `zhipu`（智谱 Web Search，与 GLM 推理共用 Key，国内首选）
   - `SERPAPI_KEY` 存在 → `serpapi`
   - `SEARXNG_URL` 存在 → `searxng`
3. 都没有时返回 `zhipu` 占位 provider，`web_search` 工具会提示用户配置

各 provider 配置读取顺序（环境变量 / 系统属性 / `.env`）：
- `zhipu`：`GLM_API_KEY`（必填，与 LLM 推理共用）+ `ZHIPU_SEARCH_ENGINE`（可选，默认 `search_std`，可选 `search_pro` / `search_pro_sogou` / `search_pro_quark`）
- `serpapi`：`SERPAPI_KEY`
- `searxng`：`SEARXNG_URL`（推荐本地 `docker run --rm -p 8888:8888 searxng/searxng`）

Web 抓取（`web_fetch`）安全策略（实现位于 `src/main/java/com/paicli/web/NetworkPolicy.java`）：

- scheme 白名单：仅允许 `http` / `https`
- 主机黑名单：屏蔽 `localhost`、`0.0.0.0`、loopback / link-local / site-local 地址（基础 SSRF 围栏，不防 DNS rebinding）
- 响应体上限：5MB（流式截断，避免 OOM）
- 整体超时：30 秒（OkHttp `callTimeout`）
- 限流：默认每 60 秒最多 30 次请求

## 常用命令

```bash
cp .env.example .env
mvn clean package
java -jar target/paicli-1.0-SNAPSHOT.jar
mvn clean compile exec:java -Dexec.mainClass="com.paicli.cli.Main"
mvn test
```

验证 RAG 相关测试：

```bash
mvn test -Dtest=CodeChunkerTest,CodeAnalyzerTest,VectorStoreTest,CodeIndexTest
```

如果只是验证一个测试类：

```bash
mvn test -Dtest=ExecutionPlanTest
```

## 当前产品行为

### 1. ReAct 模式

- 默认模式
- 主入口在 `src/main/java/com/paicli/agent/Agent.java`
- 维护对话历史
- 退出条件由 LLM 自决：只要它不再返回 `tool_calls`、直接给出 `content`，循环就结束
- `AgentBudget`（`src/main/java/com/paicli/agent/AgentBudget.java`）只承担保险阀职责，三种兜底任一命中即收尾：
  - 累计 `inputTokens + outputTokens` 超过 token 预算（默认 300_000）
  - 连续 N 轮（默认 3）出现完全相同的工具名 + 参数，判定为死循环
  - 累计轮数超过硬上限（默认 50），最终防御
- 不再使用"固定最多 10 轮"的策略；新代码改动前阅读 `AgentBudget` 的注释比读老 README 更可靠
- 支持工具调用后继续思考
- 用户默认看到的是流式输出的模型 `reasoning_content`（如果接口返回）和回复内容；ReAct 同一次用户输入只打印一次 `🧠 思考过程` 标题，工具调用前后的后续推理继续归在同一块下；ReAct 流式头标签使用 `🤖 回复`（而非 `最终结果`，避免在模型调用工具前先 narrate 时误导用户）；Plan 阶段同样走流式展示；终端会先渲染常见 Markdown 再输出；工具参数、工具返回片段、Token 使用量不再作为默认用户输出
- 会写入短期记忆

### 2. Plan-and-Execute 模式

- 通过 `/plan` 或 `/plan <任务>` 进入
- 主入口在 `src/main/java/com/paicli/agent/PlanExecuteAgent.java`
- 流程是：规划 -> 用户审阅 -> 执行 DAG -> 汇总结果
- 计划执行完后会回到默认 `ReAct`
- 简单任务应优先生成最小计划；不要为了凑步数引入无关读写文件或中间落盘步骤

### 3. Plan 审阅交互

以 `Main.java` 当前实现为准：

- `Enter`：执行当前计划
- `Ctrl+O`：展开完整计划
- `ESC`：如果当前在展开视图则先折叠，否则取消本次计划
- `I`：输入补充要求并重新规划

注意：

- 这里不是 README 里旧描述的“只有 Enter / ESC / I”
- 原始按键处理依赖 JLine raw mode
- 方向键属于终端控制序列，不应被误判成 `ESC` 取消
- 涉及这块的改动，不能只看字符串，要连输入模式和回退路径一起看

### 4. Memory 系统

- 主模块在 `src/main/java/com/paicli/memory/`
- 默认包含：短期记忆、长期记忆、摘要压缩、事实提取、Token 预算、相关记忆检索
- 注入到 system prompt 的“相关记忆”应只来自长期记忆；当前轮用户输入和短期对话已经在消息历史里，不应再被当成“历史记忆”重复注入
- 长期记忆默认只通过显式命令 `/save <事实>` 写入；不要在每轮对话结束或 `/clear` 时自动提取事实
- 长期记忆只应保存跨会话仍成立的稳定事实；一次性任务请求、临时文件名/目录名、模型猜测或“用户想要你做什么”这类指令，不应落入长期记忆

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itwanger/paicli](https://github.com/itwanger/paicli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
