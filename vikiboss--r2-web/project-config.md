---
trigger: always_on
description: 本文档为 Claude Code (claude.ai/code) 提供仓库开发指南。
---

# CLAUDE.md

本文档为 Claude Code (claude.ai/code) 提供仓库开发指南。

## 项目速览

**R2 Web** — 纯客户端 Cloudflare R2 存储桶文件管理器，零构建、零框架、零后端。

**核心特性** 文件上传、目录浏览、文件预览、文件操作、图片压缩、PWA、多语言（zh/en/ja）、浅色/深色主题、多选批量操作。

**快速启动**

```bash
npx serve src
# 或
python3 -m http.server 5500 --directory src
```

## 代码地图

### 快速定位表

| 任务               | 文件                                        |
| ------------------ | ------------------------------------------- |
| 修改文件名模板逻辑 | `src/js/utils.js` — `applyFilenameTemplate` |
| 修改图片压缩逻辑   | `src/js/upload-manager.js` — `compressFile` |
| 添加 i18n 文案     | `src/js/i18n.js` — `const I18N`             |
| 修改按钮样式       | `src/css/components.css` — `.btn`           |
| 添加设计 Token     | `src/css/tokens.css`                        |
| 修改 R2 API 操作   | `src/js/r2-client.js`                       |
| 修改文件浏览逻辑   | `src/js/file-explorer.js`                   |
| 修改上传管理逻辑   | `src/js/upload-manager.js`                  |
| 多选批量操作逻辑   | `src/js/file-explorer.js` — `toggleSelect` / `selectAll` / `clearSelection` |

### 工具函数列表

以下函数定义在 `src/js/utils.js`，供各模块导入：

- `t(key, vars)` — i18n 翻译（来自 `src/js/i18n.js`）
- `applyFilenameTemplate(tpl, file)` — 文件名模板处理，占位符：`[name]`、`[ext]`、`[hash:N]`、`[date:FORMAT]`、`[timestamp]`、`[uuid]`、`/`
- `compressFile(file, config, onStatus)` — 图片压缩（定义在 `upload-manager.js`）
  - PNG 特殊处理：直接优化缓冲区，不重新编码
  - 自适应逻辑：压缩后更大则使用原文件

## 项目结构

```
r2-web/
├── readme.md          — 项目说明、使用指南
├── package.json       — 依赖声明（仅用于类型提示）
├── jsconfig.json      — JSDoc 类型检查配置
└── src/               — 源码目录（即部署目录）
     ├── index.html    — 应用外壳、import map、对话框模板
     ├── main.js       — 入口（仅 new App()）
     ├── manifest.json — PWA 配置
     ├── style.css     — 样式主入口（仅导入 css 子目录）
     ├── js/           — 业务逻辑模块
     │    ├── app.js              — 主协调器
     │    ├── config-manager.js   — 配置持久化、Base64 分享
     │    ├── r2-client.js        — S3 API 客户端
     │    ├── ui-manager.js       — 主题、Toast、对话框、Tooltip
     │    ├── file-explorer.js    — 目录导航、排序、分页、缩略图
     │    ├── upload-manager.js   — 上传、文件名模板、图片压缩
     │    ├── file-preview.js     — 图片/视频/音频/文本预览
     │    ├── file-operations.js  — 重命名、复制、移动、删除
     │    ├── i18n.js             — 多语言（zh/zh_TW/en/ja）
     │    ├── constants.js        — 常量
     │    └── utils.js            — 工具函数
     └── css/          — 样式模块（CSS Layers）
          ├── reset.css       — CSS Reset
          ├── tokens.css      — 设计 Token（定义所有变量）
          ├── base.css        — 全局基础样式
          ├── layout.css      — 布局容器
          ├── components.css  — 通用 UI 组件
          ├── utilities.css   — 工具类
          └── animations.css  — 动画与过渡
```

## 开发环境

### 依赖管理

**重要** `package.json` 依赖仅用于类型提示，运行时通过 `import map` 从 CDN 加载。

添加新依赖：

```bash
# 1. 安装获取类型定义
pnpm add -D package-name@x.y.z

# 2. 在 src/index.html 的 <script type="importmap"> 中添加映射
# {
#   "imports": {
#     "package-name": "https://esm.sh/package-name@x.y.z"
#   }
# }

# 3. 在对应模块（如 src/js/utils.js）中导入使用
# import { something } from 'package-name'
```

### 类型检查

- JSDoc 注释提供类型信息
- 运行 `pnpm typecheck` 验证类型

## 架构速查

### JavaScript 类架构

每个类独立为一个模块，位于 `src/js/`：

| 类               | 文件                 | 职责                                         |
| ---------------- | -------------------- | -------------------------------------------- |
| `ConfigManager`  | `config-manager.js`  | localStorage 持久化、Base64 配置分享         |
| `R2Client`       | `r2-client.js`       | S3 API 客户端（基于 `aws4fetch` 签名）       |
| `UIManager`      | `ui-manager.js`      | 主题、Toast、对话框、上下文菜单、Tooltip     |
| `FileExplorer`   | `file-explorer.js`   | 目录导航、排序、分页、懒加载缩略图、列表缓存 |
| `UploadManager`  | `upload-manager.js`  | 拖拽/粘贴上传、文件名模板、图片压缩          |
| `FilePreview`    | `file-preview.js`    | 图片/视频/音频/文本预览                      |
| `FileOperations` | `file-operations.js` | 重命名、复制、移动、删除（递归删除目录）     |
| `App`            | `app.js`             | 主协调器、i18n 处理                          |

**应用初始化** 在 `src/main.js`：

```javascript
// 启动应用，构造函数内部自动创建所有管理器并初始化
new App()
```

`App` 构造函数内部会自动创建 `ConfigManager`、`R2Client`、`UIManager`，然后根据配置状态决定是否初始化文件浏览器等其他管理器。

### 列表缓存机制

`FileExplorer` 类内置缓存机制（搜索 `#cache`），缓存文件列表 5 分钟，减少 API 请求。

```javascript
/** @typedef {{ data: { folders: FileItem[], files: FileItem[], isTruncated: boolean, nextToken: string }, ts: number }} CacheEntry */
const CACHE_TTL = 5 * 60 * 1000 // 5 minutes
#cache = new Map()

// 缓存键包含 continuationToken，分页独立缓存
const cacheKey = `${prefix}::${continuationToken}`

// 刷新时可传 bypassCache = true 强制重新加载
await #loadPage(isInitial, bypassCache = false)
```

## CSS 速查

### CSS Layers

样式通过 `@layer` 组织优先级（`src/style.css`）：

```css
@layer reset, tokens, base, layout, components, utilities, animations;

@import './css/reset.css';
@import './css/tokens.css';
/* ... */
```

### 设计 Token

所有样式值通过 CSS 自定义属性定义（`src/css/tokens.css`）：

**Token 类别**

- **间距** `--sp-*`（1/2/3/4/5/6/8/10/12）
- **字体** `--text-*`（xs/sm/base/md/lg/xl）
- **颜色** `--bg-*`、`--text-*`、`--border-*`（light-dark 自适应）
- **圆角** `--radius-*`（sm/md/lg/xl/full）
- **动画** `--duration-*`（fast/normal/slow）、`--ease-*`（out/in-out）
- **Z-index** `--z-*`（dropzone/upload-panel/context-menu/dialog/toast/tooltip）

**使用方式** 在 `src/css/tokens.css` 查看完整定义。

**示例**

```css
.card {
  padding: var(--sp-4);
  gap: var(--sp-2);
  font-size: var(--text-base);
  color: var(--text-primary);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vikiboss/r2-web](https://github.com/vikiboss/r2-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
