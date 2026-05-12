---
trigger: always_on
description: **FlashClaw = 轻量稳定的地基 + 乐高式插件扩展**
---

# FlashClaw 开发规则

## 核心理念

**FlashClaw = 轻量稳定的地基 + 乐高式插件扩展**

### 设计哲学

```
┌─────────────────────────────────────────┐
│           用户插件 (可选)                │
│   天气、翻译、自动化、插件仓库...        │
├─────────────────────────────────────────┤
│           官方插件                       │
│   飞书、定时任务、记忆、web-fetch...     │
├─────────────────────────────────────────┤
│           核心地基 (极简)                │
│   消息路由 | 插件加载器 | AI Provider    │
└─────────────────────────────────────────┘
```

### 三大原则

| 原则 | 含义 | 实践 |
|------|------|------|
| **轻量** | 核心极简，功能通过插件实现 | 新功能 = 新插件，核心只做路由/加载/通信 |
| **稳定** | 地基稳固，插件可热插拔 | 插件崩溃不影响核心，稳定性优先于精简 |
| **迅速** | 响应快，加载快，开发快 | 热加载、按需加载、简单 API |
| **小模型友好** | 4B 小模型也能流畅运行 | 不依赖模型能力，代码层面保证功能正确 |

### 核心 vs 插件

**核心只做三件事：**
1. 消息路由 - 接收消息，分发到 Agent
2. 插件加载 - 发现、加载、管理插件生命周期
3. AI Provider - 可插拔的 AI 客户端（内置 Anthropic，支持 OpenAI 等）

**其他一切都是插件：**
- 渠道插件：飞书、Telegram、企业微信...
- 工具插件：记忆、定时任务、web-fetch...
- 扩展插件：插件仓库、Web UI...（这些也是插件！）

## 插件规范

### 插件结构

```
plugins/{plugin-name}/
├── plugin.json      # 必需：元信息
└── index.ts         # 必需：入口文件
```

### plugin.json 格式

```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "type": "channel | tool",
  "description": "简短描述",
  "main": "index.ts",
  "dependencies": []
}
```

### 插件类型

| 类型 | 接口 | 用途 |
|------|------|------|
| `channel` | `ChannelPlugin` | 消息渠道（飞书、钉钉） |
| `tool` | `ToolPlugin` | AI 可调用的工具 |
| `provider` | `AIProviderPlugin` | AI Provider（Anthropic、OpenAI 等） |

> `type` 字段仅存在于 `plugin.json` 清单文件中，插件对象本身不需要 `type` 字段。

### 工具插件模板

```typescript
import type { ToolPlugin, ToolContext, ToolResult } from '../../src/plugins/types';

const plugin: ToolPlugin = {
  name: 'my-tool',
  version: '1.0.0',
  description: '工具描述',

  schema: {
    name: 'tool_name',
    description: '工具功能描述',
    input_schema: {
      type: 'object',
      properties: {
        param: { type: 'string', description: '参数说明' }
      },
      required: ['param']
    }
  },

  async execute(params: unknown, context: ToolContext): Promise<ToolResult> {
    // 实现逻辑
    return { success: true, data: {} };
  }
};

export default plugin;
```

### 渠道插件模板

```typescript
import type { ChannelPlugin, MessageHandler } from '../../src/plugins/types';

const plugin: ChannelPlugin = {
  name: 'my-channel',
  version: '1.0.0',

  async init() {
    // 初始化配置
  },

  onMessage(handler: MessageHandler): void {
    // 保存 handler，收到消息后调用
  },

  async start(): Promise<void> {
    // 启动连接
  },

  async stop(): Promise<void> {
    // 清理资源
  },

  async sendMessage(chatId: string, content: string): Promise<void> {
    // 发送消息到指定会话
  }
};

export default plugin;
```

## 开发规范

### 添加新功能 = 创建新插件

**不要修改核心代码！**

```bash
# 1. 创建插件目录
mkdir plugins/my-feature

# 2. 创建 plugin.json 和 index.ts

# 3. 重启服务，插件自动加载
npm run dev
```

### 热加载

- 开发模式下修改插件自动重载
- 生产模式需重启服务以加载新插件
- 插件崩溃自动隔离，不影响其他插件

### 文件组织

```
src/                    # 核心代码（尽量不动）
├── index.ts            # 主入口、消息路由
├── core-api.ts         # 核心 API 层（所有渠道的统一入口）
├── cli.ts              # CLI 命令行工具
├── commands.ts         # 聊天命令处理
├── commands/           # CLI 子命令
│   ├── init.ts         # 交互式初始化向导
│   ├── doctor.ts       # 环境诊断
│   ├── security.ts     # 安全审计
│   └── daemon.ts       # 后台服务管理
├── channel-manager.ts  # 渠道管理器
├── session-tracker.ts  # Token 追踪
├── message-queue.ts    # 消息队列
├── health.ts           # 健康检查
├── metrics.ts          # 运行指标
├── plugins/            # 插件系统
│   ├── loader.ts       # 插件加载器
│   ├── manager.ts      # 插件管理器
│   ├── types.ts        # 插件类型定义
│   ├── installer.ts    # 插件安装/卸载
│   └── index.ts        # 插件入口
├── core/               # 核心模块（尽量不动）
│   ├── api-client.ts   # 向后兼容的 AI 客户端（已迁移到插件）
│   ├── memory.ts       # 记忆管理 (短期+长期+压缩+每日日志)
│   ├── context-guard.ts # 上下文窗口保护
│   └── model-capabilities.ts # 模型能力检测
├── utils/              # 工具模块
│   ├── network.ts      # 网络工具（IP 检测、URL 提取）
│   ├── env-substitute.ts # 环境变量替换
│   ├── log-rotate.ts
│   ├── rate-limiter.ts
│   └── retry.ts
└── ...

plugins/                # 核心插件（3个，系统运行必需）
├── anthropic-provider/ # Anthropic AI Provider（默认）
├── memory/             # 记忆工具（remember/recall/log）
└── send-message/       # 发送消息工具

community-plugins/      # 社区/官方扩展插件（按需加载）
├── feishu/             # 飞书渠道
├── telegram/           # Telegram 渠道
├── openai-provider/    # OpenAI/Ollama Provider
├── schedule-task/      # 定时任务
├── list-tasks/         # 查看任务列表
├── cancel-task/        # 取消任务
├── pause-task/         # 暂停任务
├── resume-task/        # 恢复任务
├── register-group/     # 注册群组
├── memory-vector/      # 语义记忆搜索（Ollama embedding）
├── web-fetch/          # 网页抓取
├── web-search/         # 互联网搜索（DuckDuckGo，代理支持）
├── local-file-read/    # 本地文件读取 + 目录列表
├── reminder/           # 简化版定时提醒
├── agent-manager/      # 多 Agent 注册表（路由、白名单、agent_send）
├── hello-world/        # 测试插件
├── browser-control/    # 浏览器自动化控制 (Playwright)
└── web-ui/             # Web 管理界面
```

### 命名约定

| 类型 | 约定 | 示例 |
|------|------|------|
| 插件目录 | kebab-case | `my-plugin` |
| 工具名称 | snake_case | `my_tool` |
| 类名 | PascalCase | `MyPlugin` |
| 函数名 | camelCase | `myFunction` |

## 配置系统

### 环境变量替换

配置文件支持 `${VAR}` 语法，运行时自动替换：

```json
{
  "apiUrl": "${API_URL:-http://localhost:3000}",
  "appId": "${FEISHU_APP_ID}"
}
```

- `${VAR}` - 从环境变量获取值
- `${VAR:-default}` - 有默认值

### 配置备份

每次修改配置前自动备份（最多 5 个）：
- `config.json.bak.1` - 最新备份

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GuLu9527/flashclaw](https://github.com/GuLu9527/flashclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
