---
trigger: always_on
description: > **本文件是 wg-skills 仓库的主控规范文档**，所有项目级逻辑、结构、流程、规范均沉淀于此。
---

# wg-skills 仓库规范（Agents Skills 主控文档）

> **本文件是 wg-skills 仓库的主控规范文档**，所有项目级逻辑、结构、流程、规范均沉淀于此。
>
> - **Claude 用户**：Claude Code 通过 `CLAUDE.md` 间接引用本文件
> - **Codex / Workbuddy / OpenCode / VS Code / Kimi 等其他智能体**：直接读取 `AGENTS.md`
> - **人类维护者**：直接阅读本文
>
> **任何对本仓库的修改都必须先更新本文件**，再同步到具体 skill 目录与根 README。

---

## 一、仓库定位

本仓库是一套 **Agents Skills 集合（智能体技能集合）**，不绑定任何一家厂商、不依赖任何私有协议。

每个子目录对应一个独立 skill（技能），技能通过 `SKILL.md` 定义触发条件、审查维度与输出格式。**任何支持 Skill / Agent 协议的智能体都可以加载并使用本仓库的技能**。

**核心设计理念**：前后端一体化。前端 4 套体系（Vue / UniApp / React / HTML）与后端 5 语言矩阵（Java / Python / Go / Node.js / Rust）通过 `api-contract.md` 契约连接，前端请求层自动适配后端响应信封和错误码，Design Token 命名体系跨框架对齐。

### 1.1 核心理念

> **将"古法编程"容纳到 AI 体系之中。**

AI 时代的编程不是"取代程序员"，而是把传统工程中**重复、规范、体系化**的部分抽出来，沉淀成可被 AI 调用的技能，让人类专注于业务决策与架构判断。

| 维度 | 古法编程 | 本仓库技能体系 |
|------|----------|---------------|
| 写一个 CRUD 接口 | 查文档、写代码、写测试、写文档（半天） | 一句话触发技能（5 分钟） |
| 搭一套后端骨架 | 选型、写配置、写中间件、写鉴权、写重启脚本（1-2 天） | 触发 `xxx-init-skill`（10 分钟） |
| 前后端联调 | 口头约定接口格式、反复沟通 | `api-contract.md` 契约连接，零沟通成本 |
| 前端样式不统一 | 各页面颜色/间距/字号随意 | Design Token 体系，Vue 与 UniApp 命名完全对齐 |
| 学一门新语言 | 买书、查文档、写 demo（1-2 周） | 触发"一天学会 xxx 体系"技能（半天） |
| 多人协作 | 各自风格、风格不统一 | 技能强制规范（生成即遵守） |
| 项目迭代 | 重构历史债、文档落后 | 技能可平滑升级，文档同步 |

### 1.2 适用场景

本仓库的技能不是"演示玩具"，而是面向**真实的工程项目**：

#### 场景 1：具体业务项目
- 🛒 **商城系统**：商品、订单、支付、库存、优惠券
- 📋 **后台管理系统**：用户、权限、组织、数据看板、表单工作流
- 📱 **App / 小程序**：uniapp 一套代码多端发布
- 💬 **社区 / 内容平台**：帖子、评论、点赞、关注
- 🎓 **在线教育**：课程、章节、播放进度、考试

#### 场景 2：大型平台项目
需要大量中间件与多插件部署：
- 📨 **消息中心**：Kafka / Redis Stream / RabbitMQ 接入
- 🔍 **搜索引擎**：Elasticsearch 集成
- ⚡ **缓存层**：Redis Cluster / 多级缓存策略
- 📊 **监控告警**：Prometheus / Grafana / 链路追踪
- 🔐 **统一认证**：OAuth2 / SSO / JWT 多端
- 🗄️ **分库分表**：ShardingSphere / MyCAT
- 📦 **对象存储**：MinIO / OSS / S3 兼容

> 对应技能：`database/` 下各模块技能（mysql/pgsql/mongodb/redis/kafka/sqlite）提供中间件配置参考。

#### 场景 3：学习各种语言规范
**针对小白、追求更高效的 Vibecoding**：

- ☕ **一天学会 Java 体系** → 触发 `java-fast-skill` / `springboot-init-skill`
- 🐍 **一天学会 Python 体系** → 触发 `python-fast-skill` / `fastapi-init-skill`
- 🐹 **一天学会 Go 体系** → 触发 `go-gin-init-skill`
- 🟢 **一天学会 Node.js 体系** → 触发 `nodejs-init-skill`
- 🎨 **一天学会 Vue 体系** → 触发 `vue-base-skill` / `vue-generate-skill`
- 📱 **一天学会 React 体系** → 触发 `react-generate-skill`
- 📱 **一天学会 uni-app 体系** → 触发 `uniapp-base-skill` 及其 21 个子技能

> 一句"帮我用 Java 搭一个商城"，智能体会按"选型 → 骨架 → 业务模块"的顺序串起多个技能，**小白也能跟着提示词完成一个完整项目**。

### 1.3 与"传统 AI 一键做项目"的区别

| 维度 | 传统 AI 一键做项目 | 本仓库技能体系 |
|------|-------------------|----------------|
| 输出 | 一次性代码片段 | **完整可运行、可演进**的项目 |
| 文档 | 通常无 | 强制交付 `api-contract.md` + `docs/project-guide.md` |
| 前后端 | 只生成一端 | **完整前后端 + 数据库 + 架构**，通过 `api-contract.md` 契约连接 |
| 前端体系 | 单一框架 | **4 套体系**（Vue/UniApp/React/HTML），共享 Design Token 和请求层 |
| 规范 | AI 自由发挥 | 技能强制约束（生成即遵守） |
| 迭代 | 重做 | 技能可平滑升级、向下兼容 |
| 学习曲线 | 一次性的"惊喜" | 可**循序渐进**的体系 |
| 中间件 | 通常无 | 内置部署套件（K8s / Docker / Nginx / Redis / Kafka / DB） |

**一句话**：本仓库提供的不是"AI 帮我写了一个项目"，而是"**一套完整的可进阶可迭代的体系化项目**，包括完整的前后端和数据库，包括架构"。

---

## 二、支持的多智能体

本仓库的技能遵循**通用的 Skill / Agent 协议**，任何支持以下特性的智能体都可以直接使用：

| 智能体 | 类型 | 加载方式 |
|--------|------|----------|
| **Claude Code** | Anthropic 官方 | 通过 `CLAUDE.md` 引用本文件，自动加载 `SKILL.md` |
| **Codex** | OpenAI | 通过 `AGENTS.md` 直接加载 |
| **Workbuddy** | AI 编程助手 | 通过 `AGENTS.md` 加载 |
| **OpenCode** | 开源 AI IDE | 通过 `AGENTS.md` 加载 |
| **VS Code** | GitHub Copilot | 通过 `.github/copilot-instructions.md` 或 `AGENTS.md` |
| **Cursor** | AI 编辑器 | 通过 `.cursorrules` 引用本文件 |
| **Cline** | VS Code 插件 | 通过 `.clinerules` 引用本文件 |
| **Kimi** | Moonshot AI | 通过 `AGENTS.md` 加载 |
| **通义灵码** | 阿里云 | 通过自定义指令加载 |
| **文心一言** | 百度 | 通过系统提示词加载 |

**技能文件兼容性**：所有 `SKILL.md` 文件使用统一的 YAML frontmatter：

```yaml
---
name: skill-name              # 必填，kebab-case
description: 一句话描述技能用途  # 必填，含触发词
---
```

**任何智能体只要能识别这个 frontmatter + Markdown 正文结构，就能加载本仓库的技能。**

---

## 三、目录结构（三大板块）

本仓库采用 **3 + N** 板块布局，将所有 skill 按业务域归类：

```
wg-skills/
├── AGENTS.md                          # 本文件：主控规范（所有智能体的入口）
├── CLAUDE.md                          # Claude Code 入口（仅引用 AGENTS.md）
├── README.md                          # 仓库总览（人类视角）
├── .gitignore
├── LICENSE
│
├── vibeCoding/                        # 🎯 板块一：编程开发（占比 85%）
│   ├── backend/                       # 后端语言矩阵
│   │   ├── java/                            # Java 语言矩阵
│   │   │   ├── java-fast-skill/             # Java 快速入门（小白友好）
│   │   │   ├── springboot-init-skill/        # Spring Boot 一键初始化
│   │   │   └── springboot-module/            # Java 业务模块矩阵
│   │   │       ├── springboot-auth-module-skill/
│   │   │       ├── springboot-agent-module-skill/
│   │   │       ├── springboot-dict-module-skill/
│   │   │       ├── springboot-kafka-module-skill/
│   │   │       ├── springboot-log-module-skill/
│   │   │       ├── springboot-notification-module-skill/
│   │   │       ├── springboot-org-permission-module-skill/
│   │   │       ├── springboot-payment-module-skill/
│   │   │       ├── springboot-redis-module-skill/
│   │   │       ├── springboot-storage-module-skill/
│   │   │       └── springboot-upload-module-skill/
│   │   ├── python/                          # Python 语言矩阵
│   │   │   ├── python-fast-skill/           # Python 快速入门（小白友好）
│   │   │   ├── fastapi-init-skill/          # FastAPI 一键初始化
│   │   │   ├── article-generator/           # 多平台文章生成
│   │   │   ├── hot-trend-collector/         # 热点抓取工具

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jiushiwon/wg-skills](https://github.com/jiushiwon/wg-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
