---
trigger: always_on
description: 本项目是一个医疗3D模型网页展示系统，支持上传、配置和展示STL格式的医疗三维模型。
---

# 医疗3D模型展示系统 - 项目规范

## 项目概述

本项目是一个医疗3D模型网页展示系统，支持上传、配置和展示STL格式的医疗三维模型。

## 技术栈

- **Framework**: Next.js 16 (App Router)
- **Core**: React 19
- **Language**: TypeScript 5
- **3D渲染**: Three.js
- **UI组件**: shadcn/ui (基于 Radix UI)
- **样式**: Tailwind CSS 4
- **数据库**: PostgreSQL (pg 驱动直连)
- **认证**: JWT Cookie (jose)
- **安全**: IP白名单 + 登录频率限制 (middleware)
- **二维码**: qr-code-styling

## 项目结构

```
/workspace/projects/
├── src/
│   ├── app/
│   │   ├── page.tsx              # 首页（系统介绍）
│   │   ├── layout.tsx            # 根布局
│   │   ├── login/                # 登录页面
│   │   │   └── page.tsx
│   │   ├── upload/               # 上传端页面（需登录）
│   │   │   └── page.tsx
│   │   ├── view/                 # 展示端页面
│   │   │   └── page.tsx
│   │   ├── verify/               # 患者验证页面
│   │   │   └── page.tsx
│   │   ├── list/                 # 患者模型列表页面
│   │   │   └── page.tsx
│   │   ├── history/              # 医生历史记录页面（需登录）
│   │   │   └── page.tsx
│   │   ├── admin/users/          # 用户管理页面（需管理员权限）
│   │   │   └── page.tsx
│   │   └── api/
│   │       ├── auth/             # 认证相关接口
│   │       │   ├── login/route.ts
│   │       │   ├── logout/route.ts
│   │       │   └── me/route.ts
│   │       ├── admin/users/      # 用户管理接口
│   │       │   ├── route.ts
│   │       │   └── [id]/route.ts
│   │       ├── admin/delete-logs/ # 管理员删除日志接口
│   │       │   └── route.ts
│   │       ├── doctor/           # 医生相关接口
│   │       │   ├── history/route.ts
│   │       │   └── delete-logs/route.ts
│   │       ├── patient/          # 患者相关接口
│   │       │   ├── verify/route.ts
│   │       │   └── [id]/configs/route.ts
│   │       ├── upload/           # 文件上传接口
│   │       │   ├── route.ts              # POST 小文件上传 / DELETE 批量删除 / PATCH 重命名
│   │       │   └── chunk/route.ts        # POST 分块上传 / PUT 合并分块 / DELETE 取消分块
│   │       └── medical/
│   │           └── config/
│   │               ├── route.ts           # POST 创建配置
│   │               └── [code]/route.ts    # GET 获取配置
│   ├── components/
│   │   ├── ui/                   # shadcn/ui 组件库
│   │   ├── ThreeDViewer.tsx      # Three.js 3D视图组件
│   │   └── QRCode.tsx            # 二维码生成组件
│   ├── lib/
│   │   └── auth.ts               # JWT认证工具
│   ├── types/
│   │   └── medical.ts            # 医疗模型类型定义
│   └── storage/
│       └── database/
│           ├── db.ts                  # PostgreSQL 连接池 (pg 驱动)
│           ├── medical-service.ts     # 医疗数据服务
│           ├── patient-service.ts     # 患者数据服务
│           ├── user-service.ts        # 用户数据服务
│           └── shared/
│               └── schema.ts         # 数据库Schema类型
├── public/
│   ├── logo.png                  # 二维码嵌入Logo
│   └── STL文件/                  # STL文件存储目录
├── .coze                         # Coze CLI配置
├── .env.local                    # 环境变量配置
├── package.json
└── tsconfig.json
```

## 核心页面

### 1. 首页 (`/`)
- 系统介绍
- 功能导航卡片
- 跳转至上传页面

### 2. 登录页 (`/login`)
- 用户名/密码登录
- JWT Cookie认证
- 根据角色跳转（admin→用户管理，doctor→上传页面）

### 3. 上传端 (`/upload`) - 需登录
- 全局配置区：标题、患者姓名、患者手机号、医院、科室
- 模型文件上传：STL格式
- 模型参数配置：名称、颜色、透明度
- 提交后生成二维码（带医院Logo）
- 显示访问链接

### 4. 展示端 (`/view?code=xxx`)
- Three.js 3D渲染视图
- 鼠标左键旋转、右键平移、中键缩放
- 左上角悬浮模型控制面板（可折叠）
- 颜色映射、显隐控制、透明度调整
- 显示模型体积

### 5. 患者验证页 (`/verify`)
- 输入姓名+手机号验证身份
- 验证成功跳转到模型列表

### 6. 模型列表页 (`/list?patient_id=xxx`)
- 显示患者所有模型配置
- 按时间倒序排列
- 点击查看具体模型

### 7. 用户管理页 (`/admin/users`) - 需管理员权限
- 用户列表
- 创建新用户
- 重置密码、禁用/启用、删除用户
- 操作日志 Tab：查看所有医生用户的删除操作记录，支持按用户筛选

### 8. 医生历史记录页 (`/history`) - 需登录
- 显示当前用户上传的所有模型配置
- 按时间倒序排列
- 显示标题、患者姓名、医院、科室、日期、模型数量
- 可查看二维码
- 可点击查看模型或复制访问链接
- 删除功能：可删除整条配置记录，同步删除关联的 STL 文件和文件夹
- 删除日志 Tab：显示所有删除操作记录，不可修改

## 数据库表结构

### users
| 字段 | 类型 | 说明 |
|------|------|------|
| id | serial | 主键 |
| username | text | 用户名（唯一） |
| password_hash | text | bcrypt哈希密码 |
| role | text | 角色：admin/doctor |
| status | text | 状态：active/disabled |
| created_at | timestamp | 创建时间 |
| updated_at | timestamp | 更新时间 |

### patients
| 字段 | 类型 | 说明 |
|------|------|------|
| id | serial | 主键 |
| name | text | 患者姓名 |
| phone | text | 手机号 |
| created_at | timestamp | 创建时间 |

### medical_configs
| 字段 | 类型 | 说明 |
|------|------|------|
| id | serial | 主键 |
| code | text | 访问码 |
| title | text | 页面标题 |
| patient_name | text | 患者姓名 |
| patient_id | integer | 关联患者ID |
| patient_phone | text | 患者手机号 |
| patient_gender | text | 患者性别 |
| patient_age | integer | 患者年龄 |
| hospital | text | 医院名称 |
| department | text | 科室名称 |
| creator_id | integer | 创建者用户ID |
| created_at | timestamp | 创建时间 |
| updated_at | timestamp | 更新时间 |

### medical_models
| 字段 | 类型 | 说明 |
|------|------|------|
| id | serial | 主键 |
| config_id | integer | 关联配置ID |
| name | text | 模型名称 |
| color | text | 渲染颜色 |
| opacity | integer | 透明度 |
| file_path | text | STL文件路径 |
| visible | integer | 是否可见 |
| sort_order | integer | 排序 |
| created_at | timestamp | 创建时间 |

### delete_logs
| 字段 | 类型 | 说明 |
|------|------|------|
| id | serial | 主键 |
| operator_id | integer | 操作人用户ID |
| operator_name | text | 操作人用户名 |
| config_id | integer | 被删除的配置ID |
| config_code | text | 被删除的配置访问码 |
| config_title | text | 配置标题 |
| patient_name | text | 患者姓名 |
| hospital | text | 医院名称 |
| department | text | 科室名称 |
| model_count | integer | 模型数量 |
| deleted_files | text[] | 被删除的文件路径列表 |
| deleted_at | timestamp | 删除时间 |

## API接口

### 认证接口

#### POST /api/auth/login
登录

**请求体：**
```json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Coelophysis1/medical-3d-viewer](https://github.com/Coelophysis1/medical-3d-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
