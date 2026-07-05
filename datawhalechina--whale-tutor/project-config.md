---
trigger: always_on
description: 写给参与维护本项目的 AI / 工程师。简明列出架构边界与开发约定。**面向用户的快速上手见 [README.md](README.md)。**
---

# CLAUDE.md

写给参与维护本项目的 AI / 工程师。简明列出架构边界与开发约定。**面向用户的快速上手见 [README.md](README.md)。**

## 项目本质

Whale Tutor 是一个 **AI 驱动的交互式学习陪伴产品**,不是 "AI 生成的课程"。核心差异：动态路径、个体化记忆、可重新进入、产物可带走。

完整产品理念与教育学第一性原理见 [notes/background_1.md](notes/background_1.md) / [notes/background_2.md](notes/background_2.md) / [notes/background_3.md](notes/background_3.md)。完整工程架构（4 层 18 模块）见 [notes/plan.md](notes/plan.md)。**v0.2 完成后的运行时业务逻辑详解(状态机 / decideNext / DB 写入语义 / event 映射)见 [notes/orchestrator.md](notes/orchestrator.md)** — 跨模块改动前必读。**Hint / Adaptive / Review-LO 三机制如何串成一套 stuck 处理协议** 见 [notes/stuck-handling.md](notes/stuck-handling.md)。

**当前阶段:v0.2 智能编排闭环 ✅ 已跑通**(单人开发,2026-05-08)。在 v0 基础上接通 PathOrchestrator 答错→换说法 / review_lo 兜底 / hint 折扣 / `subject` 学科参数化,4 种 pattern 全部支持 adaptive `generate`。范围、决策、分阶段设计见 [plan 文件](C:/Users/gyh/.claude/plans/readme-md-mvp-notes-3-background-md-luminous-shannon.md);**v0 / v0.2 实际实现清单与 v0.3 路线图见本文件末尾**。

## 技术栈

- **Monorepo**: pnpm workspaces (Node ≥22, pnpm 8.15.x via corepack)
- **Web**: Vue 3 + Vite + TypeScript + Element Plus + Pinia + Vue Router + axios
- **Server**: NestJS + Kysely + mysql2 + ConfigModule
- **共享类型**: [packages/tutor-types](packages/tutor-types)(workspace 协议引用)
- **DB**: MySQL 8.0(docker compose 启,初始化脚本走 `db/init/`)
- **AI 模型**: DeepSeek(OpenAI 兼容协议),通过 AI Gateway 抽象,可切换
- **代码沙盒**: Pyodide(浏览器端 Web Worker,服务端零参与)

## 目录结构

```
whale-tutor/
├── web/                  # Vue 前端
│   ├── src/views/        # 路由页(HomeView/OnboardingView/LearnView/ArchiveView)
│   ├── src/components/   # patterns/ 下放 4 种 Pattern 渲染卡片
│   ├── src/stores/       # Pinia(session/learner/pyodide)
│   ├── src/api/          # axios 薄封装,baseURL=/api
│   ├── src/workers/      # pyodide.worker.ts
│   └── src/router/
├── server/               # NestJS 后端
│   ├── src/database/     # Kysely 全局 provider(@Global)
│   ├── src/users/        # v0.2 认证骨架的占位示例
│   ├── src/session/      # 会话生命周期 + Pattern/Learner/Event 编排
│   ├── src/knowledge/    # 课程图谱(YAML 加载)
│   ├── src/pattern/      # Pattern 注册表 + 4 种实现
│   ├── src/learner/      # Learner Model
│   ├── src/event/        # Event Bus(唯一写入 events 表的入口)
│   └── src/ai/           # AI Gateway + prompt YAML
├── packages/
│   ├── tutor-types/      # 前后端共享 TS 类型(workspace 内部)
│   └── cli-node/         # ★ 课程作者 npm 包(发到 npm)
│       ├── package.json  # bin: whale-tutor → bin/cli.mjs
│       ├── bin/cli.mjs   # commander 入口:init / start / doctor / lint / build / generate
│       ├── lib/          # config.mjs / db.mjs / runner.mjs / scaffold.mjs / doctor.mjs / lint.mjs / build.mjs / generate.mjs
│       ├── _bundle/      # ⚠ 构建产物,不入 git(由 build:cli-bundle 填充)
│       └── README.md
├── scripts/
│   └── build-cli-bundle.mjs  # 填 cli-node/_bundle/(经 build/server-bundle/ 中间产物)
├── db/init/01-schema.sql # MySQL 初始化(docker 首启 + cli `start` idempotent 都跑)
├── notes/                # 产品理念 + 完整架构文档
└── docker-compose.yml
```

## 架构 5 条核心原则(任何改动都要遵守)

1. **内容与模式正交**。LO 是内容(YAML),Pattern 是模式(代码 + prompt 模板)。运行时由 PathOrchestrator 组合。新增课程零修改 Pattern,新增 Pattern 零修改课程。
2. **状态与行为分离**。`learner_state` 是数据,PathOrchestrator 是行为。所有状态读写通过定义好的 service API,不绕过。
3. **AI 调用必须收口**。任何业务模块禁止直调 LLM。所有 LLM 交互走 [server/src/ai/ai-gateway.service.ts](server/src/ai/ai-gateway.service.ts) 的 `complete()`。Prompt 在 `server/src/ai/prompts/*.yaml`,代码不内嵌 prompt 字符串。
4. **事件流是数据真相**。学习者每次行为先写 `events` 表(事实表,不可变),`learner_state` 等派生表理论上可重建。新增数据需求优先想"能不能从 events 派生",而不是新建一张表。
5. **评估是抽象而非功能**。Diagnostic / Formative / Summative / Delayed / Transfer 形态不同,底层是同一套"出题 + 评判 + 更新状态"。`assessment.type` 字段已就位。

## 学习模型(v0 双阶段 + 章末测试)

每个 **LO** 内部分两阶段:

1. **必做阶段(static)** — 学习者按序做完 LO 的 `requiredInteractions`(YAML 静态预置,完整含答案)。期间 mastery 走 `untouched → exposed → practicing`。
2. **自适应阶段(adaptive)** — 必做完成后,PathOrchestrator 根据 mastery 反馈决定:
   - mastered + 高 confidence → 推进下一 LO
   - practicing 但有犹豫 → 调 `Pattern.generate()` 由 AI 用 `adaptivePatterns` 中的某种模式动态出题巩固
   - 连续错 → 回退前置 LO

每个 **Chapter** 末有 **assessment** — 同样是一组 `requiredInteractions`(静态预置,使用现有 4 种 Pattern),所有 LO 都 mastered 后才解锁。Chapter 自身的状态机:`learning → assessment → completed`。

**数据来源**:

- 必做题(LO + Chapter Assessment)的 prompt **完全静态**,YAML 写死题干/选项/答案,AI 不参与生成
- 仅在自适应阶段、AI 评估(free_recall/spot_the_bug 解释)、章末档案生成等场景调 AI Gateway

**类型边界(server-only vs 公开)**:

- [packages/tutor-types/src/domain.ts](packages/tutor-types/src/domain.ts) 中 `*Definition` 系列(`CourseDefinition` / `ChapterDefinition` / `LearningObjectiveDefinition` / `ChapterAssessmentDefinition` / `RequiredInteraction`)是 **server-only**,KnowledgeService 解析 YAML 后的内存结构,含答案/expected/rubric
- 同名无后缀的 `Course` / `Chapter` / `LearningObjective` / `ChapterAssessmentSummary` 是 **公开版**,前端通过 HTTP 拿到的就是这个,只暴露元信息和"必做题数量"
- 服务层做转换:`Definition → Public`,永远禁止把 `*Definition` 直接 `JSON.stringify` 下发

## 课程内容存储格式(YAML + $ref .md)

`server/src/knowledge/data/<course-id>/` 是课程内容的根。**结构与短字段写在 YAML,长 markdown 抽到 .md 文件用 `$ref` 引用**。

```
server/src/knowledge/data/python-basics/
  course.yaml                                    # 课程元数据
  course-description.md
  chapters/
    list_and_iter/
      chapter.yaml                               # 章节元数据 + LO 引用 + assessment 引用
      description.md
      los/
        list_basics/
          lo.yaml                                # LO 定义(结构 + $ref)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datawhalechina/whale-tutor](https://github.com/datawhalechina/whale-tutor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
