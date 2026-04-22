---
trigger: always_on
description: 这是一个基于 Go + Gin + GORM 和 Vue3 + TypeScript + Ant Design Vue 的全栈项目管理软件，采用 TDD 开发方式。
---

# 项目管理系统 - Cursor 开发规则

## 项目概述

这是一个基于 Go + Gin + GORM 和 Vue3 + TypeScript + Ant Design Vue 的全栈项目管理软件，采用 TDD 开发方式。

## 技术栈

### 后端
- **语言**: Go 1.24+
- **框架**: Gin (v1.11.0)
- **ORM**: GORM (v1.30.5)
- **数据库**: SQLite (默认，支持 MySQL)
- **测试**: Go test + testify
- **认证**: JWT + 微信开放平台扫码登录

### 前端
- **框架**: Vue 3 (Composition API)
- **语言**: TypeScript
- **UI库**: Ant Design Vue (v4.2.6)
- **构建工具**: Vite
- **状态管理**: Pinia
- **路由**: Vue Router
- **HTTP客户端**: Axios
- **日期处理**: dayjs
- **图表**: ECharts + vue-echarts
- **Markdown**: marked + highlight.js

## 项目结构

### 后端结构
```
backend/
├── cmd/server/          # 应用入口
├── internal/
│   ├── api/            # API路由层（Handler模式）
│   ├── model/          # 数据模型（GORM）
│   ├── utils/          # 工具函数（response, pagination, database等）
│   ├── middleware/     # 中间件（auth, cors, logger, permission等）
│   ├── config/         # 配置管理
│   └── websocket/      # WebSocket支持
├── pkg/                # 可复用包（auth, wechat, permission等）
├── tests/unit/         # 单元测试
└── migrations/         # 数据库迁移
```

### 前端结构
```
frontend/src/
├── api/                # API接口定义
├── views/              # 页面组件
├── components/         # 公共组件
├── stores/             # Pinia状态管理
├── router/             # 路由配置
└── utils/              # 工具函数（request, date等）
```

## 编码规范

### Go 后端规范

#### Handler 模式
- 每个 Handler 结构体包含 `db *gorm.DB` 字段
- 使用构造函数模式：`NewXxxHandler(db *gorm.DB) *XxxHandler`
- Handler 方法接收 `*gin.Context` 作为参数
- 私有辅助方法使用小写开头，不接收 `*gin.Context`

```go
type ProjectHandler struct {
    db *gorm.DB
}

func NewProjectHandler(db *gorm.DB) *ProjectHandler {
    return &ProjectHandler{db: db}
}

func (h *ProjectHandler) GetProjects(c *gin.Context) {
    // 实现逻辑
}

func (h *ProjectHandler) getProjectStatistics(projectID uint) gin.H {
    // 私有辅助方法
}
```

#### API 响应格式
- 统一使用 `utils.Success()` 和 `utils.Error()` 返回响应
- 成功响应：HTTP 200，body 包含 `{code: 200, message: "success", data: {...}}`
- 错误响应：HTTP 200，body 包含 `{code: 错误码, message: "错误信息"}`
- 错误码：200=成功，404=资源不存在，500=服务器错误

```go
// 成功响应
utils.Success(c, gin.H{
    "list":  projects,
    "total": total,
})

// 错误响应
utils.Error(c, 404, "项目不存在")
```

#### 数据模型规范
- 使用 GORM 标签定义模型
- 软删除使用 `gorm.DeletedAt`
- 时间字段使用 `time.Time`
- JSON 数组使用自定义 `StringArray` 类型（实现 `driver.Valuer` 和 `sql.Scanner`），仅用于测试类型等场景
- 标签使用独立表（Tag）和关联表（project_tags）实现多对多关系
- 关联关系使用 GORM 的 `Preload` 和 `Joins` 避免 N+1 问题

```go
type Project struct {
    ID          uint           `gorm:"primaryKey" json:"id"`
    Name        string         `gorm:"not null" json:"name"`
    Tags        []Tag          `gorm:"many2many:project_tags;" json:"tags,omitempty"`
    DeletedAt   gorm.DeletedAt `gorm:"index" json:"-"`
    CreatedAt   time.Time      `json:"created_at"`
    UpdatedAt   time.Time      `json:"updated_at"`
}

type Tag struct {
    ID          uint           `gorm:"primaryKey" json:"id"`
    Name        string         `gorm:"size:50;not null;uniqueIndex" json:"name"`
    Description string         `gorm:"size:200" json:"description"`
    Color       string         `gorm:"size:20;default:'blue'" json:"color"`
    Projects    []Project      `gorm:"many2many:project_tags;" json:"projects,omitempty"`
}
```

#### 查询和分页
- 使用 `utils.GetPage(c)` 和 `utils.GetPageSize(c)` 获取分页参数
- 搜索使用 `c.Query("keyword")` 和 `c.QueryArray("tags")`（标签ID数组）
- 多标签搜索使用 JOIN 查询：`JOIN project_tags ON ... WHERE tag_id IN ?`
- 标签筛选使用标签ID，不是标签名称
- 分页查询必须返回 `total` 字段

```go
page := utils.GetPage(c)
pageSize := utils.GetPageSize(c)
offset := (page - 1) * pageSize

// 标签筛选（使用JOIN）
if tagIDs := c.QueryArray("tags"); len(tagIDs) > 0 {
    query = query.Joins("JOIN project_tags ON project_tags.project_id = projects.id").
        Where("project_tags.tag_id IN ?", tagIDs).
        Group("projects.id")
}

var total int64
query.Model(&model.Project{}).Count(&total)

query.Preload("Tags").Offset(offset).Limit(pageSize).Order("created_at DESC").Find(&projects)
```

#### 日期处理
- 前端传递日期字符串格式：`"YYYY-MM-DD"` 或 `"YYYY-MM-DD HH:mm:ss"`
- 后端使用 `time.Time` 类型
- 日期解析使用 `time.Parse("2006-01-02", dateStr)` 或 `time.Parse("2006-01-02 15:04:05", dateStr)`
- 返回给前端的日期使用 JSON 序列化（自动转换为 RFC3339 格式）

#### 工时和进度计算
- 任务和 Bug 包含 `EstimatedHours`（预估工时）和 `ActualHours`（实际工时）
- `ActualHours` 自动从 `ResourceAllocation` 计算得出
- `Progress` 自动计算：`ActualHours / EstimatedHours * 100`（范围 0-100%）
- 更新工时时自动创建/更新 `ResourceAllocation` 记录

#### 数据库迁移
- 使用 GORM 的 `AutoMigrate` 进行自动迁移
- 对于 SQLite 的 NOT NULL 字段添加，使用手动迁移逻辑
- 使用 `gorm.io/gorm/migrator.Migrator` 接口进行细粒度控制
- 避免使用数据库特定的 SQL 语法，保持 SQLite 和 MySQL 兼容

### TypeScript/Vue 前端规范

#### 组件结构
- 使用 `<script setup>` 语法（Composition API）
- 使用 TypeScript 类型定义
- 组件文件使用 PascalCase 命名

```vue
<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { message } from 'ant-design-vue'
import type { Project } from '@/api/project'

const projects = ref<Project[]>([])
const loading = ref(false)

const loadProjects = async () => {
  // 实现逻辑
}
</script>
```

#### API 调用
- API 函数定义在 `src/api/` 目录下
- 使用统一的 `request` 实例（已配置拦截器）
- 响应数据自动提取 `data` 字段
- 错误自动显示 message 提示

```typescript
import request from '@/utils/request'

export interface Tag {
  id: number
  name: string
  description?: string
  color?: string
}

export interface Project {
  id: number
  name: string
  tags?: Tag[]  // 标签对象数组
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [funnywwh/prjflow](https://github.com/funnywwh/prjflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
