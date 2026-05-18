---
trigger: always_on
description: 本文档为 AI 代理提供项目上下文和工作指南。
---

# AGENTS.md

本文档为 AI 代理提供项目上下文和工作指南。

## 用户偏好设置

1. 请保持对话语言为中文
2. 我的系统为 Mac
3. 遇到复杂且不确定的问题请积极查阅官方文档，使用 context7 mcp
4. 前端 ts 项目不要使用 any 类型
5. 禁止修改 gitignore 中的文件
6. 最后用 `pnpm lint --fix` 修复代码风格问题

---

## 项目概述

**EXIF Gallery Nuxt** 是一个全栈照片管理解决方案，可免费部署在 Cloudflare Workers 上，集成了 AI 智能处理、浏览器端图片压缩等功能。

### 核心功能

- **免费 Cloudflare 部署** - 基于 Cloudflare Workers 的零成本托管
- **AI 智能图像分析** - 支持 OpenAI 和 Gemini 进行语义分析和智能描述生成
- **浏览器端图片压缩** - 使用 JSQuash 在浏览器端进行 JPEG、WebP、AVIF 格式转换和压缩
- **EXIF 数据管理** - 完整的 EXIF 信息读取、展示和编辑
- **标签系统** - 灵活的照片标签管理和筛选
- **相机/镜头筛选** - 按相机品牌、型号和镜头筛选照片
- **云存储** - 基于 NuxtHub (Cloudflare R2) 的对象存储
- **现代 UI** - 基于 shadcn-vue 和 inspira-ui 的美观界面
- **响应式设计** - 支持桌面、平板和移动端的完美适配
- **视图过渡** - 使用 Vue View Transition API 实现流畅的页面切换效果

### 技术栈

| 类别           | 技术                                                                              |
| -------------- | --------------------------------------------------------------------------------- |
| **框架**       | Nuxt 4 (Vue 3.5)                                                                  |
| **边缘运行时** | [NuxtHub](https://hub.nuxt.com) v0.10.4 - Cloudflare 部署                         |
| **数据库**     | D1 (SQLite) + [Drizzle ORM](https://orm.drizzle.team/)                            |
| **存储**       | Cloudflare R2 (via NuxtHub Blob)                                                  |
| **样式**       | [UnoCSS](https://unocss.dev/) + Tailwind CSS                                      |
| **UI 组件**    | [shadcn-vue](https://www.shadcn-vue.com/) + [inspira-ui](https://inspira-ui.com/) |
| **状态管理**   | Pinia 3                                                                           |
| **认证**       | [nuxt-auth-utils](https://github.com/Atinux/nuxt-auth-utils)                      |
| **国际化**     | @nuxtjs/i18n (English/中文)                                                       |
| **验证**       | vee-validate + Zod 4                                                              |
| **AI**         | ai-sdk + @ai-sdk/openai + @ai-sdk/google                                          |
| **图片处理**   | @jsquash/\* (JPEG, WebP, AVIF, PNG, Resize) - 浏览器端压缩                        |
| **设备检测**   | @nuxtjs/device                                                                    |
| **图标**       | Lucide Icons (via @iconify-json/lucide)                                           |

---

## 项目结构

```
exif-gallery-nuxt/
├── app/                    # 前端应用
│   ├── components/         # Vue 组件
│   │   ├── ui/            # shadcn-vue 风格组件
│   │   ├── inspira/       # inspira-ui 动画组件
│   │   ├── ui-pro/        # 项目扩展 UI 组件
│   │   ├── AdminUpload.vue    # 管理员上传页面
│   │   ├── UploadConfig.vue   # 上传配置组件
│   │   ├── PhotoItem.vue      # 照片项组件
│   │   ├── EditPhotoDialog.vue # 编辑照片对话框
│   │   └── [其他组件...]
│   ├── composables/       # Vue 组合式函数
│   │   ├── usePhotos.ts       # 照片数据获取逻辑
│   │   ├── useUploadConfig.ts # 上传配置管理
│   │   ├── useAIConfig.ts     # AI 配置管理
│   │   ├── useTheme.ts        # 主题管理
│   │   ├── usePhotoSort.ts    # 照片排序
│   │   └── [其他 composables...]
│   ├── layouts/           # 页面布局
│   │   ├── admin.vue      # 管理后台布局
│   │   ├── default.vue    # 默认布局
│   │   └── home.vue       # 首页布局
│   ├── middleware/        # 路由中间件
│   │   └── auth.ts        # 认证中间件
│   ├── pages/             # 应用页面
│   │   ├── index.vue      # 首页
│   │   ├── grid.vue       # 网格视图
│   │   ├── admin/         # 管理页面
│   │   ├── p/[...id].vue  # 照片详情页
│   │   ├── tag/[...tag].vue # 标签筛选页
│   │   ├── camera/[...camera].vue # 相机筛选页
│   │   └── lens/[...lens].vue     # 镜头筛选页
│   ├── stores/            # Pinia 状态存储
│   │   ├── photos.ts      # 照片数据存储
│   │   └── navigation.ts  # 导航状态存储
│   ├── utils/             # 工具函数
│   │   ├── compress.ts    # 浏览器端图片压缩
│   │   ├── exif.ts        # EXIF 数据提取
│   │   ├── ai.ts          # AI 集成
│   │   ├── aiProviders.ts # AI 供应商管理
│   │   └── [其他工具...]
│   ├── workers/           # Web Workers
│   │   ├── encode.worker.ts # 图片编码 Worker
│   │   └── decode.worker.ts # 图片解码 Worker
│   └── app.vue            # 根 Vue 组件
├── server/                # 后端 API
│   ├── api/               # API 路由
│   │   ├── auth.post.ts       # 管理员认证
│   │   ├── photos/            # 照片管理 API
│   │   │   ├── [id].delete.ts # 删除照片
│   │   │   ├── [id].get.ts    # 获取单张照片
│   │   │   ├── [id].put.ts    # 更新照片
│   │   │   ├── index.get.ts   # 列出照片（分页、筛选）
│   │   │   └── upload.post.ts # 上传照片
│   │   └── tags/              # 标签 API
│   │       └── index.get.ts   # 列出所有标签
│   ├── db/                # 数据库
│   │   ├── schema.ts      # 数据库 Schema 定义
│   │   └── migrations/    # 迁移文件
│   ├── routes/            # 额外的服务端路由
│   │   └── photos/[pathname].get.ts # 照片文件服务
│   └── utils/             # 服务端工具
│       ├── drizzle.ts     # Drizzle 工具
│       └── tag.ts         # 标签工具
├── i18n/                  # 国际化
│   └── locales/           # 翻译文件
│       ├── en.yml         # 英文
│       └── zh.yml         # 中文
├── types/                 # TypeScript 类型定义
│   ├── auth.d.ts          # 认证类型
│   └── index.ts           # 主类型定义
├── public/                # 静态资源
├── .github/workflows/     # GitHub 工作流
│   └── migrate.yml        # 数据库迁移工作流
├── wrangler.jsonc         # Wrangler 部署配置
├── nuxt.config.ts         # Nuxt 配置
├── tsconfig.json          # TypeScript 配置
├── uno.config.ts          # UnoCSS 配置
└── package.json           # 依赖和脚本
```

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wiidede/exif-gallery-nuxt](https://github.com/wiidede/exif-gallery-nuxt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
