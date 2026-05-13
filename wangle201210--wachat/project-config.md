---
trigger: always_on
description: wachat 是一个使用 Wails (Go + Web) 构建的跨平台 AI 聊天桌面应用。
---

# wachat - Claude 项目上下文

## 项目概述

wachat 是一个使用 Wails (Go + Web) 构建的跨平台 AI 聊天桌面应用。

**技术栈**: Go 1.22+ (后端) + Vue 3 + TypeScript (前端) + Wails v2.10.2 (框架) + SQLite (数据库)

**目标**: 提供轻量级、高性能的本地 AI 聊天体验，支持流式响应和会话持久化。

## 项目结构

```
wachat/
├── backend/              # Go 后端 - 严格分层架构
│   ├── api.go           # API Facade - 统一对外接口
│   ├── config/          # 配置层 - YAML 配置读取
│   ├── database/        # 数据库层 - SQLite 连接和迁移
│   ├── model/           # 模型层 - 数据结构定义
│   ├── repository/      # Repository 层 - 数据访问抽象
│   └── service/         # Service 层 - 业务逻辑
│       ├── ai.go        # AI 服务
│       ├── chat.go      # 聊天服务
│       ├── rag_service.go  # RAG 文档检索服务
│       └── binary_manager.go  # 二进制管理服务
├── frontend/            # Vue 3 前端 - Composition API
│   ├── src/
│   │   ├── components/  # UI 组件（单一职责）
│   │   ├── composables/ # 可复用逻辑
│   │   ├── views/       # 页面视图
│   │   └── wailsjs/     # Wails 自动生成的绑定（不要手动修改）
├── bin/                 # 嵌入的二进制文件（qdrant等）
├── config.yaml          # 配置文件（从 config.example.yaml 复制）
├── main.go              # Wails 应用入口
└── app.go               # 应用主逻辑 - 连接前后端
```

## 架构设计原则

### 后端分层

**各层职责**:
- `app.go`: Wails 生命周期管理，前端方法绑定，事件发送，*请不要在这里写复杂的业务逻辑*
- `api.go`: API 外观层，初始化各层依赖，提供统一接口
- `service/`: 业务逻辑，不直接访问数据库
  - `ai.go`: AI 对话服务，集成 RAG 增强
  - `chat.go`: 聊天会话管理
  - `rag_service.go`: RAG 文档检索服务（基于 go-rag）
  - `binary_manager.go`: 管理嵌入的二进制文件（qdrant等）
- `repository/`: 数据访问，GORM 操作封装
- `database/`: 数据库连接、迁移
- `model/`: 数据结构（DB 模型 + 业务模型）
- `config/`: YAML 配置读取和管理（使用 Viper）

### 前端组件化

**Composition API + 单一职责**:
- `views/`: 页面级组件，负责布局和组合
- `components/`: UI 组件，接收 props，发出 events，无业务逻辑
- `composables/`: 可复用的逻辑（useChat, useAutoScroll）

## 代码约定

### Go 代码规范

1. **错误处理**: 所有错误必须向上传递，使用 `fmt.Errorf` 包装
   ```go
   if err != nil {
       return fmt.Errorf("failed to do something: %w", err)
   }
   ```

2. **命名约定**:
   - 文件名: 小写+下划线 (`chat_service.go`)
   - 结构体: 大驼峰 (`ChatService`)
   - 方法/函数: 大驼峰（导出）或小驼峰（私有）

3. **依赖注入**: 通过构造函数传递依赖
   ```go
   func NewChatService(
       convRepo *repository.ConversationRepository,
       msgRepo *repository.MessageRepository,
       aiService *AIService,
   ) *ChatService
   ```

4. **不使用全局变量**: 所有依赖通过结构体字段传递

### TypeScript/Vue 代码规范

1. **组件命名**: 大驼峰 (`ChatSidebar.vue`)

2. **Composition API 风格**:
   ```typescript
   import { ref, computed, onMounted } from 'vue'

   const data = ref<Type>()
   const computed = computed(() => ...)
   ```

3. **类型安全**: 所有函数参数和返回值必须有类型标注

4. **Props 定义**:
   ```typescript
   interface Props {
     message: Message
     active?: boolean
   }
   const props = defineProps<Props>()
   ```

5. **事件发送**:
   ```typescript
   const emit = defineEmits<{
     'select-conversation': [id: string]
   }>()
   ```

### CSS/样式规范

1. **使用 TailwindCSS**: 优先使用 Tailwind utility classes
2. **自定义样式**: 使用 `<style scoped>` + `:deep()` 修改子组件
3. **颜色方案**: 使用 Tailwind 灰色调（gray-100, gray-700 等）

## 开发工作流程

### 添加新功能的标准流程

1. **后端**:
   ```
   1. 定义数据模型 (model/types.go)
   2. 创建 Repository 方法 (repository/*.go)
   3. 实现 Service 逻辑 (service/*.go)
   4. 在 API 层暴露接口 (api.go)
   5. 在 App 层绑定前端 (app.go)
   ```

2. **前端**:
   ```
   1. 运行 wails dev 自动生成 Go 绑定
   2. 创建/更新 Composable (composables/*.ts)
   3. 创建/更新组件 (components/*.vue)
   4. 在 View 中使用 (views/*.vue)
   ```

### 数据库修改流程

1. 修改 `model/types.go` 中的结构体
2. GORM 会在下次启动时自动迁移
3. 如需手动迁移，修改 `database/database.go` 的 `AutoMigrate` 调用

### 配置管理流程

**初始化配置**:
1. 复制 `config.example.yaml` 为 `config.yaml`
2. 修改 `config.yaml` 中的配置项（AI API、数据库路径等）
3. 配置文件会被 `.gitignore` 忽略，不会提交到版本控制

**配置文件搜索顺序**:
1. `WACHAT_CONFIG_PATH` 环境变量指定的目录
2. 当前工作目录
3. 当前工作目录向上查找的项目根目录（通过 go.mod 定位）
4. 可执行文件所在目录
5. 可执行文件向上查找的项目根目录
6. 用户配置目录 `~/.config/wachat`

**开发模式配置**:
```bash
# 如果 wails dev 找不到配置文件，设置环境变量
export WACHAT_CONFIG_PATH=$(pwd)
wails dev
```

**配置结构**:
```yaml
ai:
  base_url: "https://api.openai.com/v1"
  api_key: "your-api-key"
  model: "gpt-3.5-turbo"

binaries:
  enabled: true
  use_embedded: false
  bin_path: "./bin"
  startup_order:
    - qdrant
    - wailsproject

rag:
  enabled: false  # 需要 Elasticsearch 支持
  top_k: 5  # 检索返回的文档数量
```

**添加新配置项**:
1. 在 `backend/config/config.go` 中添加对应的结构体字段
2. 在 `config.Load()` 函数中设置默认值
3. 更新 `config.yaml` 和 `config.example.yaml`
4. 使用 `config.Get()` 获取配置

### 事件通信

**后端 → 前端**:
```go
runtime.EventsEmit(ctx, "stream:response", map[string]interface{}{
    "conversationId": id,
    "chunk": content,
})
```

**前端监听**:
```typescript
const runtime = (window as any).runtime
runtime.EventsOn('stream:response', (data: any) => {
    // 处理事件
})
```

## 已知问题和注意事项

### 1. Wails 自动生成的绑定
- **位置**: `frontend/src/wailsjs/`
- **不要手动修改**: 每次 `wails dev` 都会重新生成
- **如何更新**: 修改 Go 代码后，Wails 会自动重新生成

### 2. Node.js 版本要求
- Vite 7 需要 Node.js 20.19+ 或 22.12+
- 如果版本过低会有警告但仍能运行

### 3. 前端依赖安装
- 必须使用 `pnpm` 包管理器
- 安装命令: `pnpm --dir frontend install`
- 不要在 frontend 目录下直接运行 `cd frontend && pnpm install`（shell 配置问题）

## 测试策略

### 当前状态
- 项目处于早期阶段，暂无自动化测试
- 主要依靠手动测试


## 构建和部署

### 开发环境
```bash
wails dev  # 启动开发服务器
```

### 生产构建
```bash
wails build  # 构建当前平台
wails build -platform darwin/amd64  # 指定平台
```

### 构建产物
- **位置**: `build/bin/`
- **macOS**: `.app` 应用包
- **Windows**: `.exe` 可执行文件
- **Linux**: 二进制文件

## 给 AI 助手的指导

### 修改代码时

1. **遵循分层架构**: 不要让 Service 直接调用 Database，必须通过 Repository

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangle201210/wachat](https://github.com/wangle201210/wachat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
