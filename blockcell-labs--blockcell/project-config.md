---
trigger: always_on
description: > BlockCell → BlueClaw: 自进化 AI 多智能体框架
---

# CLAUDE.md

> BlockCell → BlueClaw: 自进化 AI 多智能体框架

## 快速导航

- [项目概述](#项目概述) - 核心概念与架构
- [快速开始](#快速开始) - 安装、配置、运行
- [开发规范](#开发规范) - 工作流、代码风格、最佳实践
- [常用命令](#常用命令) - 开发、运行、发布命令
- [WebUI 开发](#webui-开发) - 前端技术栈与开发指南
- [调试技巧](#调试技巧) - 日志级别、调试命令
- [Crate 详解](#crate-详解) - 各模块详细说明

---

## 项目概述

BlockCell 是一个用 Rust 构建的自进化 AI 多智能体框架。它不只是聊天机器人，而是能真正执行任务的 AI 智能体：
读写文件、控制浏览器、分析数据、发送消息，甚至自我进化修复 bug。

### 核心概念

| 概念 | 说明 |
| ------ | ------ |
| **Agent** | 智能体运行时，负责接收消息、调用 LLM、执行工具、管理状态 |
| **Tool** | 原子能力单元，如 `read_file`、`web_fetch`、`send_message` |
| **Skill** | 组合多个工具的技能，支持 Markdown 定义 + Rhai/Python 脚本 |
| **Channel** | 外部消息渠道适配器，如 Telegram、Slack、Discord |
| **Provider** | LLM 提供商客户端，支持 OpenAI、DeepSeek、Anthropic 等 |
| **Intent** | 用户意图分类，用于路由到不同的工具集和 Agent |
| **MCP** | Model Context Protocol，用于扩展工具能力 |

## 项目结构

```text
blockcell/
├── bin/blockcell/          # CLI 入口和命令定义
├── crates/
│   ├── core/               # 核心类型、消息、能力定义
│   ├── agent/              # Agent 运行时、任务管理、事件编排
│   ├── tools/              # 50+ 内置工具实现
│   ├── skills/             # 技能引擎、版本管理、自我进化
│   ├── scheduler/          # Cron 任务、心跳、后台作业
│   ├── channels/           # 多渠道适配 (Telegram/Slack/Discord/飞书等)
│   ├── providers/          # LLM 提供商客户端
│   ├── storage/            # SQLite 存储 (会话/记忆/审计)
│   └── updater/            # 自动更新机制
├── webui/                  # Web 前端 (React + TypeScript + Vite)
│   ├── src/
│   │   ├── components/     # React 组件
│   │   │   └── chat/       # 聊天相关组件 (消息列表、输入框、命令选择器)
│   │   ├── lib/            # 工具函数、i18n、API 客户端
│   │   ├── App.tsx         # 主应用组件
│   │   └── main.tsx        # 入口文件
│   ├── dist/               # 构建产物 (嵌入到 binary)
│   └── package.json
├── skills/                 # 用户技能目录
└── docs/                   # 文档
```

## 快速开始

### 安装

```bash
# 方式一: 安装脚本 (推荐)
curl -fsSL https://raw.githubusercontent.com/blockcell-labs/blockcell/main/install.sh | sh

# 方式二: 从源码构建
cargo build -p blockcell --release
```

### 配置

```bash
blockcell setup  # 首次设置，创建 ~/.blockcell/config.json5
```

最小配置示例 (`~/.blockcell/config.json5`):

```json
{
  "providers": {
    "deepseek": {
      "apiKey": "YOUR_API_KEY",
      "apiBase": "https://api.deepseek.com"
    }
  },
  "agents": {
    "defaults": { "model": "deepseek-chat" }
  }
}
```

### 环境变量

BlockCell 支持通过环境变量覆盖配置：

| 变量 | 说明 | 默认值 |
| ---- | ---- | ------ |
| `BLOCKCELL_CONFIG_PATH` | 自定义配置文件路径 | `~/.blockcell/config.json5` |
| `BLOCKCELL_API_TOKEN` | Gateway API 认证令牌 | (可选，用于 WebUI 认证) |
| `BLOCKCELL_HUB_URL` | 技能中心 URL | `https://hub.blockcell.dev` |
| `BLOCKCELL_HUB_API_KEY` | 技能中心 API 密钥 | (可选) |
| `RUST_LOG` | 日志级别 (tracing) | `info` |
| `EDITOR` / `VISUAL` | 配置编辑器 | 系统默认 |

Gateway 模式额外支持 `.env` 文件 (`~/.blockcell/.env`)：

```bash
# Gateway .env 示例
BLOCKCELL_API_TOKEN=your_secure_token_here
RUST_LOG=debug,blockcell_agent=trace
```

### 运行

```bash
blockcell status   # 检查状态
blockcell agent    # 交互模式
blockcell gateway  # 守护进程 + WebUI
```

### 部署

#### Docker 部署

项目使用多阶段构建优化镜像大小：

```bash
# 构建镜像
docker build -t blockcell:latest .

# 运行容器
docker run -d \
  --name blockcell \
  -p 3000:3000 \
  -v ~/.blockcell:/home/blockcell/.blockcell \
  blockcell:latest gateway

# 使用环境变量
docker run -d \
  -e BLOCKCELL_API_TOKEN=your_token \
  -e RUST_LOG=debug \
  -p 3000:3000 \
  blockcell:latest gateway
```

#### Docker Compose

```yaml
version: '3.8'
services:
  blockcell:
    image: blockcell:latest
    command: gateway
    ports:
      - "3000:3000"
    volumes:
      - ./data:/home/blockcell/.blockcell
    environment:
      - RUST_LOG=info
      - BLOCKCELL_API_TOKEN=${API_TOKEN}
    restart: unless-stopped
```

#### 系统服务 (systemd)

```bash
# 创建服务文件
sudo tee /etc/systemd/system/blockcell.service <<EOF
[Unit]
Description=BlockCell Gateway Service
After=network.target

[Service]
Type=simple
User=blockcell
ExecStart=/usr/local/bin/blockcell gateway
Restart=on-failure
Environment=RUST_LOG=info

[Install]
WantedBy=multi-user.target
EOF

# 启用并启动
sudo systemctl enable blockcell
sudo systemctl start blockcell
```

## 主要用例

BlockCell 支持多种 AI 智能体应用场景：

| 用例 | 说明 | 示例 |
| ---- | ---- | ---- |
| **个人 AI 助手** | 通过 Telegram/Slack/Discord 进行智能对话 | 查询天气、翻译文档、代码问答 |
| **数据处理** | 文件分析、图表生成、数据转换 | 解析 Excel、生成图表、格式转换 |
| **自动化任务** | Cron 定时执行、后台作业 | 定时备份、数据同步、状态监控 |
| **多 Agent 协作** | Intent 路由分发到不同 Agent | 客服机器人、任务分发、专业分工 |
| **浏览器控制** | Web 自动化、数据抓取 | 表单填写、页面监控、数据采集 |
| **技能进化** | 自我学习、版本管理、热更新 | Bug 修复、功能迭代、知识积累 |

## 架构说明

### 核心流程

```text
用户消息 → Channel Adapter → Agent Core → LLM Provider
                ↓                              ↓
            Task Manager ← Tool Execution ← Response
                ↓
            Storage (SQLite)
```

### 关键组件

| Crate       | 职责                                           |
| ----------- | ---------------------------------------------- |
| `core`      | Message, Capability, SystemEvent 等核心类型    |
| `agent`     | Agent 运行时、Intent 解析、任务调度            |
| `tools`     | 文件/浏览器/邮件/金融等 50+ 工具               |
| `skills`    | Rhai 脚本引擎、热更新、版本控制                |
| `scheduler` | Cron 作业、心跳检测、后台任务                  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blockcell-labs/blockcell](https://github.com/blockcell-labs/blockcell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
