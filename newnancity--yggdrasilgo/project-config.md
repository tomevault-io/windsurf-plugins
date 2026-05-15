---
trigger: always_on
description: **✅ 严重功能缺失已修复！Profile Properties 完全实现**
---

# Yggdrasil API Server (Go) - 开发文档

## 🎉 最新状态 (2025-08-15)
**✅ 严重功能缺失已修复！Profile Properties 完全实现**
- ✅ `/sessionserver/session/minecraft/profile/{uuid}` 现在正确返回 properties 字段
- ✅ 实现了完整的 textures 属性（包含皮肤和披风信息）
- ✅ 实现了 uploadableTextures 属性（值为 "skin,cape"）
- ✅ 支持纤细模型（alex）的 metadata 信息
- ✅ 兼容 BlessingSkin 和文件存储两种后端
- ✅ 完全符合 Yggdrasil 技术规范要求

## 项目概述

这是一个使用Go语言和Gin框架实现的简化版Yggdrasil API服务器，用于Minecraft身份验证。项目完全遵循[Yggdrasil服务端技术规范](https://github.com/yushijinhun/authlib-injector/wiki/Yggdrasil-%E6%9C%8D%E5%8A%A1%E7%AB%AF%E6%8A%80%E6%9C%AF%E8%A7%84%E8%8C%83)。

## 技术栈

- **语言**: Go 1.21+
- **框架**: Gin HTTP框架
- **认证**: JWT令牌
- **存储**: 多后端支持（内存、文件、BlessingSkin兼容）
- **缓存**: 独立缓存层（内存、Redis、Laravel兼容文件缓存）
- **加密**: bcrypt密码哈希
- **PHP兼容**: 使用 `github.com/trim21/go-phpserialize` 实现PHP序列化兼容

## 项目结构

```
yggdrasil-api-go/
├── cmd/server/          # 主程序入口
├── internal/
│   ├── config/          # 配置管理
│   ├── handlers/        # HTTP处理器
│   ├── middleware/      # 中间件
│   ├── storage/         # 存储层（用户、角色、材质）
│   │   ├── memory/      # 内存存储
│   │   ├── file/        # 文件存储
│   │   └── blessing_skin/ # BlessingSkin兼容存储
│   ├── cache/           # 缓存层（Token、Session）
│   │   ├── memory/      # 内存缓存
│   │   ├── redis/       # Redis缓存
│   │   └── file/        # Laravel兼容文件缓存
│   ├── storage_factory/ # 存储工厂
│   └── utils/           # 工具函数
├── pkg/yggdrasil/       # 公共类型定义
└── reference/           # 参考实现（PHP版本）
```

## 已实现的API端点

### 认证服务器 (`/authserver`)
- ✅ `POST /authserver/authenticate` - 用户登录认证
- ✅ `POST /authserver/refresh` - 刷新访问令牌
- ✅ `POST /authserver/validate` - 验证访问令牌
- ✅ `POST /authserver/invalidate` - 撤销令牌
- ✅ `POST /authserver/signout` - 全局登出

### 会话服务器 (`/sessionserver`)
- ✅ `POST /sessionserver/session/minecraft/join` - 客户端进入服务器
- ✅ `GET /sessionserver/session/minecraft/hasJoined` - 服务端验证客户端
- ✅ `GET /sessionserver/session/minecraft/profile/{uuid}` - 获取用户档案

### API端点 (`/api`)
- ✅ `POST /api/profiles/minecraft` - 批量查询角色
- ✅ `GET /api/users/profiles/minecraft/{username}` - 单个角色查询

### 元数据
- ✅ `GET /` - 获取API元数据和配置信息

## 核心功能特性

1. **双重登录支持**: 支持邮箱和角色名登录
2. **JWT令牌认证**: 使用标准JWT生成和验证访问令牌
3. **兼容离线验证**: UUID生成算法兼容Minecraft离线验证
4. **多存储后端**: 内存、文件、BlessingSkin兼容存储
5. **存储缓存分离**: 存储层负责持久化，缓存层负责临时数据
6. **Laravel缓存兼容**: 支持Laravel文件缓存格式和PHP序列化
7. **BlessingSkin兼容**: 完全兼容BlessingSkin数据库和配置（只读模式，不支持材质上传/删除）
8. **中间件支持**: CORS、内容类型检查、速率限制
9. **错误处理**: 严格按照Yggdrasil规范返回错误信息

## 测试用户数据

| 邮箱              | 密码        | 角色                    |
| ----------------- | ----------- | ----------------------- |
| test1@example.com | password123 | TestPlayer1, AltPlayer1 |
| test2@example.com | password456 | TestPlayer2             |
| admin@example.com | admin123    | AdminPlayer             |

## 运行方式

```bash
# 安装依赖
go mod tidy

# 运行服务器
go run cmd/server/main.go

# 服务器将在 http://localhost:8080 启动
```

## API测试示例

### 1. 获取API元数据
```bash
curl http://localhost:8080/
```

### 2. 用户登录（邮箱）
```bash
curl -X POST http://localhost:8080/authserver/authenticate \
  -H "Content-Type: application/json" \
  -d '{
    "username": "test1@example.com",
    "password": "password123",
    "agent": {"name": "Minecraft", "version": 1}
  }'
```

### 3. 用户登录（角色名）
```bash
curl -X POST http://localhost:8080/authserver/authenticate \
  -H "Content-Type: application/json" \
  -d '{
    "username": "TestPlayer1",
    "password": "password123",
    "agent": {"name": "Minecraft", "version": 1}
  }'
```

### 4. 查询角色
```bash
curl http://localhost:8080/api/users/profiles/minecraft/TestPlayer1
```

## 开发注意事项

### 重要架构变更（2024年更新）

1. **配置系统重构**:
   - 移除了BlessingSkin配置覆盖功能
   - 使用分离的存储配置结构（memory_options, file_options等）
   - BlessingSkin存储只从数据库options表读取配置
   - **移除了重复的类型定义**：统一使用config包的类型

2. **包结构调整**:
   - `pkg/yggdrasil` 移动到 `internal/yggdrasil`
   - 添加了 `internal/storage/interface` 存储接口层
   - 存储工厂移动到 `internal/storage/factory.go`
   - **简化main.go**：直接使用配置，无需类型转换

3. **缓存清理系统**:
   - 所有缓存实现都支持 `CleanupExpired()` 方法
   - 存储接口添加了 `CleanupExpiredTokens()` 和 `CleanupExpiredSessions()` 方法
   - 主程序自动每5分钟清理过期数据

4. **类型系统优化**:
   - 移除了 `internal/storage/interface/types.go` 中的重复类型定义
   - 所有存储相关类型统一使用 `internal/config` 包
   - 存储工厂直接接受config类型，无需转换

### 核心开发原则

1. **BlessingSkin兼容**: 使用blessing_skin存储时，**绝对不能**在Go配置中覆盖数据库配置
2. **配置分离**: 每种存储类型使用独立的配置结构，不再使用通用options字段
3. **PHP兼容**: 使用 `github.com/trim21/go-phpserialize` 库实现PHP兼容
4. **自动清理**: 系统自动清理过期Token和Session，防止内存泄漏
5. **接口设计**: 所有存储和缓存都通过接口访问，支持多种实现
6. **材质只读**: BlessingSkin存储只支持材质读取，不支持上传和删除操作
7. **缓存可配置**: BlessingSkin存储支持独立配置Token和Session缓存类型
8. **完全解耦架构**: Storage和Cache完全分离，Handler直接使用两者
9. **严格遵循Yggdrasil规范**: 删除所有非标准API，只保留规范定义的接口

## BlessingSkin材质处理机制

### 材质URL计算方式
- **数据关联**: players表的tid_skin/tid_cape字段指向textures表的tid主键
- **URL生成**: texture_url_prefix + textures表的hash字段
- **配置优先级**:
  1. 优先使用Go配置文件中的texture_url_prefix
  2. 回退到BlessingSkin options表中的site_url配置

### 材质类型映射
- **steve**: 标准皮肤（4像素手臂）
- **alex**: 纤细皮肤（3像素手臂）
- **cape**: 披风

### 只读限制
- **不支持上传**: UploadTexture方法返回错误
- **不支持删除**: DeleteTexture方法返回错误
- **不支持修改**: IsUploadSupported返回false
- **只支持读取**: 通过GetPlayerTextures获取现有材质

## BlessingSkin缓存配置

### 独立缓存配置
BlessingSkin存储支持独立配置Token和Session缓存，与全局缓存配置分离：

```yaml
storage:
  type: "blessing_skin"
  blessingskin_options:
    database_dsn: "user:pass@tcp(localhost:3306)/blessing_skin"
    # 独立的Token缓存配置
    token_cache:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NewNanCity/YggdrasilGo](https://github.com/NewNanCity/YggdrasilGo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
