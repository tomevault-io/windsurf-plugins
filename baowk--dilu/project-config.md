---
trigger: always_on
description: > 本文件由 Claude Code 自动加载。所有代码编写、修改必须遵循以下规范。
---

# Dilu 项目 AI 协作规范

> 本文件由 Claude Code 自动加载。所有代码编写、修改必须遵循以下规范。

---

## 项目概述

基于 **Gin + GORM** 的 Go Web 框架，Go 模块名 `dilu`，核心依赖 `github.com/baowk/dilu-core`。

---

## 一、文件读取策略（降低 token 消耗）

**以下目录禁止主动读取**，AI 知道其存在即可，不要 cat/read：

```
docs/    # Swagger 生成文件
```

**需要了解某张表的字段时**，读对应 `model/*.go` 的前 60 行即可（结构体定义在开头）。

---

## 二、目录结构

新增业务模块严格遵循以下结构，**不得自创目录层级**：

```
internal/modules/{module}/
├── model/
│   └── {entity}.go        # AI 直接编写，可自由修改
├── apis/
│   └── {module}_{entity}_api.go
├── service/
│   ├── dto/
│   │   └── {module}_{entity}_dto.go
│   └── {module}_{entity}_service.go
└── router/
    └── {module}_router.go
```

**示例**：用户模块 → `internal/modules/sys/`，文件名前缀 `sys_`。

---

## 三、Model 层

AI 直接根据建表 SQL 编写 model 结构体，无需任何代码生成工具。

```go
// internal/modules/sys/model/sys_user.go
package model

import "time"

type SysUser struct {
    Id        int       `json:"id"        gorm:"primaryKey;autoIncrement"`
    Username  string    `json:"username"  gorm:"size:64;not null;uniqueIndex"`
    Password  string    `json:"-"         gorm:"size:128;not null"`
    Nickname  string    `json:"nickname"  gorm:"size:64"`
    Status    int       `json:"status"    gorm:"default:1"`
    CreatedAt time.Time `json:"createdAt"`
    UpdatedAt time.Time `json:"updatedAt"`
}

// 需要扩展关联字段时直接在同文件或新建同包文件中添加
type SysUserWithRoles struct {
    SysUser
    Roles []SysRole `json:"roles"`
}
```

---

## 四、Service 层

### 4.1 标准结构

```go
package service

import (
    "context"
    "strings"
    "dilu/internal/modules/sys/model"
    "dilu/internal/modules/sys/service/dto"
    "github.com/baowk/dilu-core/core/base"
)

type SysUserService struct {
    *base.BaseService
}

var SerSysUser = SysUserService{
    base.NewService(""),  // 多库时填库名，如 base.NewService("notice")
}
```

### 4.2 标准 CRUD 方法签名

```go
// 分页查询
func (s *SysUserService) QueryPage(req dto.SysUserGetPageReq, list *[]model.SysUser, total *int64) error {
    db := s.DB().WithContext(context.Background()).Model(&model.SysUser{})

    if req.Username != "" {
        escaped := strings.NewReplacer(`%`, `\%`, `_`, `\_`).Replace(req.Username)
        db = db.Where("username LIKE ?", "%"+escaped+"%")
    }
    if req.Status != 0 {
        db = db.Where("status = ?", req.Status)
    }

    if err := db.Count(total).Error; err != nil || *total == 0 {
        return err
    }

    return db.Offset(req.GetOffset()).Limit(req.GetSize()).Find(list).Error
}

// 获取单条
func (s *SysUserService) Get(id int, data *model.SysUser) error {
    return s.DB().WithContext(context.Background()).
        Where("id = ?", id).First(data).Error
}

// 创建
func (s *SysUserService) Create(data *model.SysUser) error {
    return s.DB().WithContext(context.Background()).Create(data).Error
}

// 更新（只更新非零字段）
func (s *SysUserService) UpdateById(data *model.SysUser) error {
    return s.DB().WithContext(context.Background()).
        Where("id = ?", data.Id).Updates(data).Error
}

// 删除（支持批量）
func (s *SysUserService) DelIds(ids []int) error {
    if len(ids) == 0 {
        return nil
    }
    return s.DB().WithContext(context.Background()).
        Where("id IN ?", ids).Delete(&model.SysUser{}).Error
}
```

### 4.3 Service 层规则

- ✅ 所有业务逻辑在此层
- ✅ 事务在此层开启：`s.DB().Transaction(func(tx *gorm.DB) error { ... })`
- ✅ 跨表操作在此层组合
- ❌ 禁止引用 `gin.Context`
- ❌ 禁止直接写 `c.JSON()`

---

## 五、DTO 层

```go
// internal/modules/sys/service/dto/sys_user_dto.go
package dto

import "github.com/baowk/dilu-core/core/base"

// 分页查询请求（嵌入 ReqPage 获得 Page/PageSize/GetOffset/GetSize 方法）
type SysUserGetPageReq struct {
    base.ReqPage
    Username string `json:"username" form:"username"`
    Status   int    `json:"status"   form:"status"`
}

// 创建请求
type SysUserCreateReq struct {
    Username string `json:"username" binding:"required"`
    Password string `json:"password" binding:"required,min=6"`
    Nickname string `json:"nickname"`
}

// 更新请求
type SysUserUpdateReq struct {
    Id       int    `json:"id"       binding:"required"`
    Nickname string `json:"nickname"`
    Status   int    `json:"status"`
}
```

---

## 六、Api 层

### 6.1 标准结构

```go
// internal/modules/sys/apis/sys_user_api.go
package apis

import (
    "dilu/internal/modules/sys/model"
    "dilu/internal/modules/sys/service"
    "dilu/internal/modules/sys/service/dto"
    "github.com/baowk/dilu-core/core/base"
    "github.com/gin-gonic/gin"
)

type SysUserApi struct {
    base.BaseApi
}
```

### 6.2 标准 CRUD Handler

```go
// @Summary  分页查询用户
// @Tags     sys-user
// @Accept   json
// @Produce  json
// @Param    body body     dto.SysUserGetPageReq true "查询参数"
// @Success  200  {object} base.Resp{data=base.PageResp{list=[]model.SysUser}}
// @Router   /v1/sys/user/page [post]
func (a *SysUserApi) GetPage(c *gin.Context) {
    var req dto.SysUserGetPageReq
    if err := c.ShouldBindJSON(&req); err != nil {
        a.Error(c, err)
        return
    }
    var list []model.SysUser
    var total int64
    if err := service.SerSysUser.QueryPage(req, &list, &total); err != nil {
        a.Error(c, err)
        return
    }
    a.Page(c, list, total, req.GetPage(), req.GetSize())
}

// @Summary  获取用户详情
// @Tags     sys-user
// @Param    id  path     int true "用户ID"
// @Success  200 {object} base.Resp{data=model.SysUser}
// @Router   /v1/sys/user/{id} [get]
func (a *SysUserApi) Get(c *gin.Context) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baowk/dilu](https://github.com/baowk/dilu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
