---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目定位

**说说播放器（Shuoshuo Player）** 是一款基于 Bilibili 的第三方音乐播放器，将 UP 主的视频投稿、收藏夹、直播切片转换为音频播放列表。当前仓库正在进行 **v2 重构**：

- `v1/`：旧版只读参考代码，**禁止直接修改**。本堂主可以读取 v1 实现作为对照，但所有新增代码必须落在 v2 工作区
- `packages/`（v2 工作区）：基于 pnpm Monorepo 的 React + TypeScript + Tauri 实现
  - `packages/shared/`：跨平台共享逻辑（API、Store、Types、Utils、Hooks）
  - `packages/web/`：Chrome 扩展 (Manifest V3) + Web 调试入口
  - `packages/desktop/`：Tauri v2 桌面端
- `docs/`：用户文档与开发指南（项目根级，可入 git）
- `LICENSE`：MIT License + 项目附加条款

仓库根目录 `package.json` 即 v2 工作区根，v1 不参与 v2 构建，仅做架构对照。

## v2 技术栈与边界

### 技术栈

| 层 | 选型 |
|---|---|
| 语言 | TypeScript（严格模式） |
| UI | React 18 + shadcn/ui + Tailwind CSS + lucide-react |
| 路由 | React Router v6 (Hash Router) |
| 状态 | Zustand（保留持久化中间件） |
| 表单 | React Hook Form + Zod |
| 音频 | Howler.js |
| HTTP | Axios |
| 构建 | Vite |
| 桌面端 | Tauri v2（Rust） |
| 包管理 | pnpm workspace |

### 不可逾越的边界

1. **MUI / Emotion 不可引入**：v2 已弃用 MUI，所有新组件必须使用 shadcn/ui + Tailwind
2. **Electron 相关代码不再添加**：桌面端统一走 Tauri；如需调用本地能力，写 Tauri command（Rust）+ TypeScript 适配器
3. **CRA / Craco 不再使用**：所有构建走 Vite
4. **JavaScript 仅出现在 v1/**：v2 工作区文件全部 `.ts` / `.tsx` / `.rs`
5. **v1 代码只读**：v1 中的任何文件除非被显式要求迁移否则不修改
6. **响应式分包**：Chrome 扩展产物体积软约束 < 10MB，Tauri 桌面端 < 10MB（10MB 是软警戒线，避免依赖无意识膨胀；Chrome 扩展商店实际允许 < 100MB）

## 仓库与开发命令

> v2 工作区初始化后下列命令生效；初始化前仅 `v1/` 目录可执行旧版命令。

### 根级（v2 Monorepo）

```bash
pnpm install                    # 安装所有依赖
pnpm dev:web                    # Web/Chrome 扩展开发模式（Vite dev）
pnpm dev:desktop                # Tauri 桌面端开发模式
pnpm build:web                  # Web 端构建
pnpm build:extension            # Chrome 扩展打包到 dist-extension/
pnpm build:desktop              # Tauri 桌面端打包
pnpm lint                       # ESLint 检查 packages/*/src
pnpm format                     # Prettier 格式化
```

### 包级别

```bash
pnpm --filter @shuoshuo-player/shared typecheck       # 仅检查 shared 类型
pnpm --filter @shuoshuo-player/web build              # 单独构建 web 包
pnpm --filter @shuoshuo-player/desktop tauri build    # 单独构建桌面端
```

### Chrome 扩展加载

1. `pnpm build:extension` → 产物在 `packages/web/dist-extension/`
2. Chrome → 扩展程序 → 开发者模式 → 加载已解压的扩展程序 → 选 `dist-extension/`

### v1 旧版（仅参考用，不日常运行）

旧版项目位于 `v1/shuoshuo-player`、`v1/shuoshuo-player-pc`、`v1/cloud-services`，依赖 `yarn` + CRA。**v2 开发禁止依赖 v1 任何脚本或产物**。

## 架构关键约束

### 平台桥接（Platform Bridge）

`packages/shared` 通过抽象接口（`StorageAdapter` / `AuthAdapter` / `SpiderAdapter`）暴露平台能力；具体实现由各平台包提供：

- `packages/web/src/lib/`：Chrome 扩展实现（chrome.storage / Cookie 由浏览器自动管理）
- `packages/desktop/src/lib/`：Tauri 实现（IPC 调用 Rust commands）

**业务代码只面向接口，不直接调用 `chrome.*` 或 `@tauri-apps/api`**。

### 状态管理与持久化

- 持久化 root key：`ssp_v2_player_data`（带 `ssp_v2_` 命名空间前缀），统一节流写入（PERSIST_THROTTLE_MS 默认 1000ms）
- 持久化 store 列表通过 `PERSIST_KEYS` 常量声明
- 任何含 `isLoading` / 临时态的 store 都需提供 `persistSnapshot()` 钩子在持久化前清理瞬态字段
- 云服务 API baseURL 持久化 key：`ssp_v2_cloud_api_base_url`（独立于 player_data，启动时早于任何云服务调用读取）
- v2 storage key 命名空间：所有 v2 自有 key 都以 `ssp_v2_` 前缀（常量 `SSP_V2_NAMESPACE`），与 v1 老 key（`fav_list` / `lyrics` 等裸名）彻底隔离；未来 v3 平行使用 `ssp_v3_`

### 云服务 API 对接

后端为 **`shuoshuo-crystal/backend`**（Go + GORM + PostgreSQL，**与 v1 的 cloud-services 不兼容**）：

- 默认 baseURL：`https://shuoshuo.sikong.ren/api`，可被用户在前端"调试"页（dev 模式）或 Web/扩展端"关于"页覆盖；空值 fallback 默认值
- baseURL 覆盖入口位置：
  - **Web / Chrome 扩展（生产 & dev）**：设置页「调试」tab 下「水晶蟹小屋 API 地址」（prod 构建调试 tab 整段 DCE，普通用户看不到）
  - **Tauri 桌面端 dev 模式**：同上，且 capabilities 已放通 `http://localhost:*` / `127.0.0.1:*` / `0.0.0.0:*`（含 http/https），可切到本地后端
  - **Tauri 桌面端 prod 构建**：UI 不可见、capabilities 不放通公网通配，地址固定为默认值
- 响应统一格式：`{ code: 0, data, message? }` 成功，`{ code: 4xxxxxxx, message, type?, payload? }` 错误（**不再有 `errno` 字段**）
- 鉴权：`Authorization: Bearer <token>`；密码修改后旧 token 通过 `session_key` 自动失效
- 角色枚举：`User=1`、`Admin=512`、`WebMaster=1024`，权限判定走位与运算 `(role & mask) === mask`
- 歌词管理改为 **数字 ID 寻址**（`POST /lyric/manage/:id`，`:id` 传字面量 `'new'` 表示创建）；上传字段名为 `content` 而非 v1 的 `lyric`
- 直播切片 `mid` 是 **字符串**（支持超大 UID）

### B 站 API 与 WBI 签名

- 所有 WBI 加密请求统一走 `buildBilibiliApiCall({ useWbi: true })`，**禁止手写签名逻辑**
- WBI 密钥在 `useBilibiliUserStore.getLoginUserInfo()` 时从 `nav` 接口的 `wbi_img` 提取并通过 `setWbiInfo()` 注入
- 长时间运行（> 30 分钟）需触发刷新：Chrome 扩展走 `chrome.alarms`，桌面端走前端 `setInterval`

### 路由

Hash Router，路径用短横线（`/live-slicers` / `/cloud-services`）。v1 旧路径（下划线）通过 `<Navigate replace>` 自动重定向到新路径，添加新路由时同步保留兼容项。

### 静态资源与素材目录

| 路径 | 用途 | 访问方式 | 跨平台可用 |
|---|---|---|---|
| `packages/web/public/` | Chrome 扩展 manifest 引用的 icon、需要绝对 URL 的运行期资源（如 `manifest.json` / `rules.json` / `logo16.png` 等） | 绝对路径 `/foo.png`，由 Vite publicDir 复制到产物 | ❌ 仅 Web/扩展。Tauri 端无对应 publicDir，引用会 404 |
| **`packages/web/src/assets/`** | **跨包公共图片素材**（组件内嵌图片、图标） | ES import：`import logoUrl from '@/assets/logo.png'` | ✅ Web/扩展 + Desktop（desktop `vite.config.ts` 中 alias `@` → `../web/src` 自动解析） |
| `packages/desktop/src-tauri/icons/` | Tauri 应用图标（dock / 任务栏 / 安装包） | 仅打包配置使用，不进 webview | ❌ 仅打包用 |

**新增跨平台图片资源时**：放到 `packages/web/src/assets/`，组件用 `import xxx from '@/assets/xxx.png'` 引入。Vite 编译期会处理为带 hash 的资源 URL，desktop 与 web/扩展同步可用。**禁止**通过 `<img src="/xxx.png">` 引用组件级图片——desktop 端无 publicDir 会 404。

## 代码规范与约定

### 命名


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LanceLRQ/shuoshuo-player](https://github.com/LanceLRQ/shuoshuo-player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
