---
trigger: always_on
description: 本文件为 AI 编程助手（Claude Code、Aone Copilot、Cursor、OpenCode、Qoder、悟空 等）提供项目上下文，帮助 AI 更准确地理解项目结构和开发规范。
---

# OpenYida — AI Agent 开发指引

本文件为 AI 编程助手（Claude Code、Aone Copilot、Cursor、OpenCode、Qoder、悟空 等）提供项目上下文，帮助 AI 更准确地理解项目结构和开发规范。

## 项目简介

OpenYida 是一个 CLI 工具，让开发者通过 AI 对话驱动宜搭低代码平台，实现应用的创建、表单管理、页面发布等全流程操作。

**核心定位**：AI 编程工具 × 宜搭低代码平台 的桥接层。

## 项目结构

```
openyida/
├── bin/
│   └── yida.js              # CLI 入口，解析命令并路由到 lib/
├── lib/
│   ├── core/                # 核心基础模块
│   │   ├── utils.js         # 公共工具函数（Cookie、HTTP、路径等）
│   │   ├── chalk.js         # 终端彩色输出公共样式模块（统一 chalk 风格）
│   │   ├── i18n.js          # 国际化支持
│   │   ├── locales/         # 语言包（zh、en、zh-TW、ja、ko、fr、de、es、pt、ar、hi、vi）
│   │   ├── env.js           # 检测 AI 工具环境（Claude/Cursor/Copilot/Qoder/悟空 等）
│   │   ├── env-cmd.js       # env 命令入口（显示当前环境信息）
│   │   ├── env-manager.js   # 多环境配置管理（私有化部署多环境切换）
│   │   ├── copy.js          # 初始化 project 工作目录
│   │   ├── sample.js        # sample 命令（输出代码示例到工作目录）
│   │   ├── check-update.js  # 版本检测（每天一次）
│   │   ├── check-data.js    # 数据异常检测（流程表单数据校验）
│   │   ├── update.js        # self-update 命令（通过 npm 自动更新 openyida）
│   │   ├── doctor.js        # 环境诊断与自动修复
│   │   ├── query-data.js    # 统一数据管理命令（表单/流程/任务/子表单）
│   │   ├── task-center.js   # 全局任务中心（待办/我创建的/我已处理/抄送/代提交）
│   │   └── babel-transform/ # Babel 编译器（用于自定义页面）
│   ├── auth/                # 登录认证模块
│   │   ├── login.js         # 宜搭登录（Cookie 缓存 + 扫码）
│   │   ├── auth.js          # 登录态管理（status/login/refresh/logout）
│   │   ├── org.js           # 组织管理（列出/切换组织）
│   │   └── qr-login.js      # 终端二维码扫码登录
│   ├── samples/             # 代码示例/模板（通过 openyida sample 命令输出到工作目录）
│   │   ├── yida-chart/            # ECharts 图表示例（7个）
│   │   ├── yida-custom-page/      # 自定义页面模板（2个）
│   │   ├── yida-create-app/       # 应用创建模板（1个）
│   │   ├── yida-data-management/  # 表单字段模板（1个）
│   │   ├── yida-density/          # 密度切换示例（1个）
│   │   └── yida-table-form/       # 表格表单示例（1个）
│   ├── app/                 # 应用 / 表单 / 页面管理
│   │   ├── app-list.js      # yida-app-list：查询我的应用列表（名称/appType/地址）
│   │   ├── create-app.js    # 创建宜搭应用
│   │   ├── create-page.js   # 创建自定义展示页面
│   │   ├── create-form.js   # 创建 / 更新表单页面
│   │   ├── get-schema.js    # 获取表单 Schema
│   │   ├── publish.js       # 编译并发布自定义页面（Babel 转译）
│   │   ├── export-app.js    # 导出应用（生成迁移包）
│   │   ├── import-app.js    # 导入迁移包，重建应用
│   │   └── update-form-config.js  # 更新表单配置
│   ├── page-config/         # 页面公开访问 / 分享配置
│   │   ├── verify-short-url.js    # 验证短链接 URL
│   │   ├── save-share-config.js   # 保存公开访问 / 分享配置
│   │   └── get-page-config.js     # 查询页面公开访问 / 分享配置
│   ├── permission/          # 表单权限管理
│   │   ├── get-permission.js      # 查询表单权限配置
│   │   └── save-permission.js     # 保存表单权限配置
│   ├── process/             # 流程管理
│   │   ├── configure-process.js   # 配置并发布流程规则
│   │   ├── create-process.js      # 创建流程表单（一体化）
│   │   └── preview-process.js     # 流程预览（可视化流程图 + 高亮当前节点）
│   ├── conversation/        # AI 对话管理
│   │   ├── collector.js     # 对话记录收集
│   │   ├── formatter.js     # 对话格式化
│   │   └── export-conversation.js  # 导出对话记录
│   ├── flash-note/          # 闪记转 PRD
│   │   └── flash-to-prd.js  # 闪记转高质量 prompt（支持会议识别）
│   ├── dws/                 # 钉钉 CLI 集成
│   │   └── dws-wrapper.js   # 钉钉 CLI 包装器（通讯录/日历/待办/审批等）
│   ├── integration/         # 集成 & 自动化
│   │   └── integration-create.js  # 创建集成逻辑流
│   ├── connector/           # HTTP 连接器管理
│   │   ├── api.js                 # 连接器 API 请求封装
│   │   ├── connector-list.js
│   │   ├── connector-create.js
│   │   ├── connector-detail.js
│   │   ├── connector-delete.js
│   │   ├── connector-add-action.js
│   │   ├── connector-list-actions.js
│   │   ├── connector-delete-action.js
│   │   ├── connector-test.js
│   │   ├── connector-list-connections.js
│   │   ├── connector-create-connection.js
│   │   ├── connector-smart-create.js
│   │   ├── connector-parse-api.js
│   │   ├── connector-gen-template.js
│   │   ├── curl-parser.js         # cURL 命令解析
│   │   ├── doc-parser.js          # API 文档解析
│   │   ├── response-parser.js     # 响应结构解析
│   │   ├── action-generator.js    # Action 自动生成
│   │   └── desc-generator.js      # 描述自动生成
│   ├── cdn/                 # CDN / OSS 管理
│   │   ├── cdn-config.js          # CDN 配置读写
│   │   ├── cdn-config-cmd.js      # CDN 配置命令
│   │   ├── cdn-upload.js          # 上传图片到 OSS/CDN
│   │   └── cdn-refresh.js         # 刷新 CDN 缓存
│   ├── report/              # 宜搭报表管理
│   │   ├── create-report.js       # 创建报表（入口）
│   │   ├── index.js               # 创建报表主流程
│   │   ├── append.js              # 向已有报表追加图表
│   │   ├── chart-builder.js       # 图表 Schema 构建
│   │   ├── http.js                # 报表 HTTP 请求封装
│   │   └── constants.js           # 常量与 ID 生成工具
│   ├── db/                  # 数据库工具
│   │   └── db-seq-fix.js          # PostgreSQL Sequence 自动修复（解决主键序列漂移）
├── project/
│   ├── config.json          # 应用配置（appType、pageId 等）
│   └── pages/               # 自定义页面源码目录
├── yida-skills/
│   ├── SKILL.md             # 技能入口（索引表，列出所有子技能）
│   ├── skills/              # 子技能目录（每个 skill 自包含 SKILL.md + references/）
│   └── references/           # 跨 skill 共享参考文档（yida-api、model-api、query-condition-guide）
└── scripts/
    ├── postinstall.js       # 安装后脚本（环境检测 + 配置注入）
    ├── validate-ci.sh       # CI 校验脚本
    └── validate-structure.js # 项目结构校验
```

## 关键约定

### 命令实现规范
- 每个 CLI 命令对应 `lib/` 下一个独立的 `.js` 文件

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openyida/openyida](https://github.com/openyida/openyida) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
