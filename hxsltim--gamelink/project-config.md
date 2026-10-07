---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 📋 项目概述

GameLink是一个现代化的陪玩管理平台，专注于为游戏陪玩服务提供高效的订单分发、用户管理和打手管理功能。平台采用Go语言后端+React前端的架构，支持高并发、低延迟的业务场景。

### 🎯 核心目标
- **订单智能分发**: 基于算法的智能订单匹配系统
- **实时通信管理**: WebSocket实现的实时状态同步
- **多端用户支持**: 用户端、打手端、管理端三端协同
- **安全支付保障**: 多渠道支付集成和风控系统
- **高并发处理**: 支持万级用户同时在线

## 🏗 技术架构

### 系统架构概览
```
┌─────────────────────────────────────────────────────────┐
│                    前端应用层                           │
│    React Web + React Native + Admin Panel              │
└─────────────────────┬───────────────────────────────────┘
                      │ HTTPS + WebSocket
┌─────────────────────┴───────────────────────────────────┐
│                  API 网关层                             │
│         Kong Gateway + Custom Go Middleware            │
└─────────────────────┬───────────────────────────────────┘
                      │ Service Mesh (Istio)
┌─────────────────────┴───────────────────────────────────┐
│                  微服务层 (Go)                          │
│  用户服务 │ 订单服务 │ 支付服务 │ 通知服务 │ 游戏服务    │
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────┴───────────────────────────────────┐
│                  数据存储层                             │
│   MySQL Cluster + Redis Cluster + MongoDB + MinIO      │
└─────────────────────────────────────────────────────────┘
```

### 技术栈详情

#### 后端技术栈
- **运行环境**: Go 1.24.0
- **Web框架**: Gin + GORM
- **数据库**:
  - SQLite (开发环境)
  - PostgreSQL (生产环境)
  - Redis 7.0 (缓存)
- **认证**: JWT (golang-jwt/jwt/v5)
- **配置管理**: YAML配置文件
- **构建工具**: Go Modules + Makefile

#### 前端技术栈
- **框架**: React 18 + TypeScript
- **构建工具**: Vite
- **UI组件**: Arco Design
- **状态管理**: React Context (AuthContext, ThemeContext)
- **路由**: React Router v6
- **HTTP客户端**: 原生Fetch API
- **样式**: Less + CSS-in-JS
- **测试**: Vitest + Testing Library

## 📁 项目结构

```
GameLink/
├── backend/                 # Go后端服务
│   ├── cmd/                # 应用入口
│   │   └── user-service/   # 用户服务主程序
│   ├── internal/           # 内部包
│   │   ├── admin/          # 管理端处理器
│   │   ├── auth/           # 认证模块
│   │   ├── cache/          # 缓存层
│   │   ├── config/         # 配置管理
│   │   ├── db/             # 数据库连接
│   │   ├── handler/        # HTTP处理器
│   │   ├── model/          # 数据模型
│   │   ├── repository/     # 数据访问层
│   │   ├── service/        # 业务逻辑层
│   │   └── handler/middleware/ # 中间件
│   ├── scripts/            # 脚本文件
│   │   └── sql/            # SQL迁移脚本
│   ├── configs/            # 配置文件
│   ├── go.mod              # Go模块定义
│   └── Makefile            # 构建脚本
├── frontend/               # 前端应用
│   ├── src/                # 源代码
│   │   ├── api/            # API调用层
│   │   ├── components/     # 可复用组件
│   │   ├── contexts/       # React Context
│   │   ├── layouts/        # 布局组件
│   │   ├── pages/          # 页面组件
│   │   ├── services/       # 业务服务层
│   │   ├── types/          # TypeScript类型定义
│   │   └── utils/          # 工具函数
│   ├── public/             # 静态资源
│   ├── package.json        # NPM依赖配置
│   ├── tsconfig.json       # TypeScript配置
│   ├── vite.config.ts      # Vite构建配置
│   └── .eslintrc.cjs       # ESLint配置
├── configs/                # 全局配置文件
│   ├── config.development.yaml  # 开发环境配置
│   └── config.production.yaml   # 生产环境配置
├── docs/                   # 项目文档
├── scripts/                # 构建脚本
├── .gitignore              # Git忽略文件
├── README.md               # 项目说明
├── CONTRIBUTING.md         # 贡献指南
├── AGENTS.md               # AI开发指南
└── optimization_guide.md   # 性能优化指南
```

## 🚀 常用开发命令

### 后端开发命令

在 `backend/` 目录下执行：

```powershell
# 安装依赖
make deps

# 代码检查
make lint

# 运行测试
make test

# 启动用户服务 (开发模式)
make run CMD=user-service

# 构建所有服务
make build

# 手动运行服务
go run ./cmd/user-service
```

### 前端开发命令

在 `frontend/` 目录下执行：

```powershell
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览构建结果
npm run preview

# 代码检查
npm run lint

# 代码格式化
npm run format

# 类型检查
npm run typecheck

# 运行测试
npm run test

# 运行测试(单次)
npm run test:run
```

## 🗄️ 数据库结构和模型

### 数据模型层次结构

#### 基础模型 (Base)
```go
type Base struct {
    ID        uint64         `json:"id" gorm:"primaryKey"`
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
    DeletedAt gorm.DeletedAt `json:"deleted_at,omitempty" gorm:"index"`
}
```

#### 核心业务模型

1. **用户模型 (User)**
   ```go
   type User struct {
       Base
       Phone        string     `json:"phone,omitempty" gorm:"size:32;uniqueIndex"`
       Email        string     `json:"email,omitempty" gorm:"size:128;uniqueIndex"`
       PasswordHash string     `json:"-" gorm:"size:255"`
       Name         string     `json:"name" gorm:"size:64"`
       AvatarURL    string     `json:"avatar_url,omitempty" gorm:"size:255"`
       Role         Role       `json:"role" gorm:"size:32"`         // user/player/admin
       Status       UserStatus `json:"status" gorm:"size:32;index"`  // active/suspended/banned
       LastLoginAt  *time.Time `json:"last_login_at,omitempty"`
   }
   ```

2. **订单模型 (Order)**
   ```go
   type Order struct {
       Base
       UserID         uint64      `json:"user_id" gorm:"index"`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HXSLtim/GameLink](https://github.com/HXSLtim/GameLink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
