---
trigger: always_on
description: > 大版本更新时同步更新 README.md
---

# Telegram 云图床 Pro - 项目架构文档

> 大版本更新时同步更新 README.md

## 技术栈

| 层级 | 技术 | 版本 |
|------|------|------|
| 后端框架 | Flask + Flask-CORS + waitress | 3.0 |
| 数据库 | SQLite3 (WAL模式) | 内置 |
| Bot SDK | python-telegram-bot | 21.0.1 |
| HTTP | requests + aiohttp | 2.31 / 3.9 |
| 前端框架 | Nuxt 3 (SPA, ssr=false) | 3.13 |
| UI | Nuxt UI (Tailwind CSS) | 2.18 |
| 状态管理 | Pinia | 2.2 |
| 工具库 | VueUse | 11.0 |
| 部署 | Docker 多阶段构建 | - |

## 目录结构

```
tg-telegram-imagebed/
├── main.py                    # 入口：Flask线程 + Bot线程（~230行）
├── requirements.txt
├── Dockerfile                 # 多阶段：node:20-alpine → python:3.11-slim
├── docker-compose.yml         # 单容器，端口18793，卷 ./data:/app/data
│
├── tg_imagebed/               # 后端 Python 包
│   ├── config.py              # 基础设施配置（路径/端口/日志，无业务配置）
│   ├── utils.py               # 工具函数（加密ID、MIME、缓存头、单实例锁）
│   ├── bot_control.py         # Bot Token热更新、重启信号（线程安全缓存）
│   ├── admin_module.py        # 管理员认证 + 统计/图片管理路由（Session Cookie）
│   ├── database/              # SQLite DAL 包（9张表，WAL模式，带重试装饰器）
│   │   ├── __init__.py        # 重导出全部公开函数，外部 import 零改动
│   │   ├── connection.py      # 连接管理 + 数据库初始化 + ALTER TABLE迁移
│   │   ├── files.py           # 文件 CRUD + 统计查询
│   │   ├── tokens.py          # Token 管理（用户 + 管理员）
│   │   ├── settings.py        # 系统设置 + 公告管理
│   │   ├── galleries.py       # 用户画集 + 访问控制 + 分享 + Token 授权
│   │   └── admin_galleries.py # 管理员画集（owner_type='admin'，无虚拟 token）
│   ├── bot/                   # Telegram Bot 包
│   │   ├── __init__.py        # 导出 start_telegram_bot_thread, _get/_set_bot_status
│   │   ├── state.py           # Bot 状态管理（全局状态字典 + 线程安全读写）
│   │   ├── media_batch.py     # 批量图片处理（media_group debounce 合并）
│   │   ├── handlers.py        # 消息处理器（/start + handle_photo）
│   │   └── runner.py          # Bot 主循环（轮询 + 指数退避 + 热重启）
│   ├── api/                   # Flask Blueprint 路由层
│   │   ├── __init__.py        # 4个蓝图注册：upload/images/admin/auth
│   │   ├── upload.py          # POST /api/upload（匿名上传，魔数校验）
│   │   ├── images.py          # GET /image/<id>（CDN重定向+ETag+Range+流式返回）
│   │   ├── admin.py           # /api/admin/* 入口（Session认证，副作用导入子模块）
│   │   ├── admin_setup.py     # 首次启动管理员设置
│   │   ├── admin_tokens.py    # Token 管理（CRUD + 批量操作 + 影响范围查询）
│   │   ├── admin_cdn.py       # CDN 管理
│   │   ├── admin_storage.py   # 存储配置
│   │   ├── admin_telegram.py  # Bot 配置
│   │   ├── admin_galleries.py # 管理员画集
│   │   ├── admin_helpers.py   # 共享辅助函数
│   │   ├── auth.py            # /api/auth/*（Token CRUD + Token上传）
│   │   ├── settings.py        # /api/admin/settings（系统设置）
│   │   └── galleries.py       # 用户画集API + 公开分享API
│   ├── services/              # 业务服务层
│   │   ├── __init__.py        # 导出 FileService, CDNService, TokenService
│   │   ├── file_service.py    # 上传核心：process_upload() + record_existing_telegram_file()
│   │   ├── cdn_service.py     # Cloudflare CDN集成（监控/预热/清除，队列调度）
│   │   └── token_service.py   # Token统一调度（级联删除/影响范围/批量操作）
│   └── storage/               # 多存储后端（策略模式）
│       ├── base.py            # 抽象基类 StorageBackend（put_bytes/download/delete）
│       ├── router.py          # StorageRouter（场景路由：guest/token/group/admin）
│       └── backends/
│           ├── telegram.py    # Telegram频道存储（默认）
│           ├── s3.py          # S3兼容存储
│           ├── local.py       # 本地文件系统
│           └── rclone.py      # Rclone远程存储
│
└── frontend/                  # Nuxt 3 SPA
    ├── nuxt.config.ts         # ssr:false, nitro preset:static
    ├── app.vue                # 根组件
    ├── middleware/auth.ts     # 管理后台路由守卫
    ├── plugins/api-error-handler.client.ts
    ├── stores/
    │   ├── auth.ts            # 管理员认证（login/logout/restoreAuth）
    │   ├── token.ts           # Token Vault（多Token本地管理+自动验证+无效清理）
    │   ├── adminUi.ts         # 侧边栏折叠状态
    │   └── notification.ts    # 通知队列（优先级/分组/自动超时）
    ├── composables/
    │   ├── useImageApi.ts     # 统计/图片管理
    │   ├── useUpload.ts       # 统一上传（自动检测模式+XHR进度）
    │   ├── useGalleryApi.ts   # 访客画廊API（含密码保护+访问控制错误解析）
    │   ├── useAdminGalleryApi.ts  # 管理员画廊API
    │   ├── useAdminMenu.ts    # 管理菜单数据
    │   ├── useLightToast.ts   # 轻量Toast通知
    │   ├── useNotification.ts # 通知composable
    │   └── useStatsRefresh.ts # 统计刷新事件总线
    ├── layouts/
    │   ├── default.vue        # 用户端布局（导航栏+网格背景）
    │   ├── admin.vue          # 管理后台布局（AdminShell）
    │   └── admin-login.vue    # 登录页布局
    ├── components/
    │   ├── admin/AdminShell.vue, AdminSidebar.vue, AdminTopbar.vue
    │   ├── album/             # 相册子组件
    │   │   ├── AlbumGalleryList.vue    # 画集列表（分页+创建入口）
    │   │   ├── AlbumGalleryDetail.vue  # 画集详情（图片网格+分享管理）
    │   │   ├── AlbumGalleryCard.vue    # 画集卡片
    │   │   ├── AlbumMyUploads.vue      # 我的上传（分页+灯箱）
    │   │   └── AlbumCreateGalleryModal.vue  # 创建画集弹窗
    │   ├── docs/Layout.vue, Sidebar.vue, EndpointCard.vue, ParamsTable.vue, CodeBlock.vue
    │   ├── GalleryLightbox.vue    # 灯箱（手势/缩放/键盘）
    │   ├── LightToast.vue         # 非阻塞通知
    │   ├── TokenVaultSwitcher.vue # Token管理器（验证+无效自动移除）
    │   ├── AnnouncementModal.vue
    │   └── AuthLoginModal.vue
    └── pages/
        ├── index.vue              # 首页（拖拽/粘贴上传）
        ├── setup.vue              # 首次启动管理员设置页
        ├── album.vue              # 相册（Token切换自动刷新+refreshKey机制）
        ├── gallery.vue, guest.vue # 重定向到 /album
        ├── docs.vue               # 交互式API文档

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yziqpwrltf/tg-telegram-imagebed](https://github.com/yziqpwrltf/tg-telegram-imagebed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
