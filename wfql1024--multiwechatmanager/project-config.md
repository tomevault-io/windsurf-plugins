---
trigger: always_on
description: > 当前阶段: 见 MEMORY/FACTS.MD
---

# AGENTS.md — JhiFengMultiChat（极峰多聊）

> 最后更新: 2026-08-16
> 当前阶段: 见 MEMORY/FACTS.MD
> 记忆系统: MEMORY/（MEMORY.md + DECISIONS.MD/TODOS.MD/FACTS.MD/DEV_LOGS.MD）

---

## 一、项目身份

- **项目名**: JhiFengMultiChat（极峰多聊）
- **定位**: Windows 桌面端多平台聊天软件管理器（Java 17 重写版）
- **功能**: 微信/企业微信/QQ/TIM/钉钉/飞书多账号管理 — 多开、防撤回、一键登录、窗口切换
- **Python 旧版参考**: `legacy_python/`（不在版本控制中，仅供数据结构/业务逻辑参考）
- **工作目录**: `D:\SpaceDev\MyProj\JhiFengMultiChat`
- **远端**: `main` 分支

---

## 二、技术栈

| 层面 | 选型 |
|------|------|
| JDK | 17 LTS (`D:\SpaceDev\softwareDev\SDKs\Java\jdk-17.0.2`) |
| 构建 | Gradle 8.8 (Kotlin DSL) |
| UI | JavaFX 17, `StageStyle.TRANSPARENT` |
| 渲染 | WebView 内嵌 HTML/CSS/JS |
| JSON | Jackson 2.16 |
| 日志 | SLF4J 2.0.9 + Logback 1.4.14 + jul-to-slf4j 桥 |
| HTTP | `java.net.http.HttpClient`（JDK 内置） |
| 加密 | `javax.crypto.Cipher`（AES/CBC/PKCS5Padding） |
| 异步 | `ExecutorService` → `Platform.runLater` → `executeScript` |
| 事件 | 自建 EventBus（`core` 包，观察者模式，数据更新↔UI刷新解耦） |
| 图标 | MCP 服务器 `mcp-universal-icons` + `icons-mcp`（`.mcp.json`） |
| JNA | 5.14.0 (`jna` + `jna-platform`)，用于 Windows API 调用 |
| 测试 | JUnit 5.10.2 + Mockito 5.10.0 |

---

## 三、项目结构、数据存储与配置架构

详见 [`docs/project_structure.md`](docs/project_structure.md)。

---

## 六、页面架构

### 全局侧栏 (`#nav-sidebar`)
平台列表（动态渲染）+ 底部统计/设置入口。由 `main.js` 渲染，`MainWindow.injectJsBridge()` 触发加载。

### 主页面 (`#page-main`)
唯一主内容区，无内嵌左栏。元素查询使用作用域隔离：`querySelector('#page-main #' + id) || document.getElementById(id)`。

### 已废弃
`#page-manage` → `.old/`，`manage.js` → `.old/manage.js`。迁移详情见 `MEMORY/DECISIONS.MD`。

---

## 七、运行命令

```bash
gradle run --no-daemon --args="--dev"      # 开发运行
gradle compileJava --no-daemon             # 仅编译
gradle build --no-daemon                   # 完整构建
.\scripts\analyze.bat                      # 依赖分析
.\scripts\package-exe.bat                  # EXE 打包 (jlink + jpackage)
```

---

## 八、待推进

见 `MEMORY/TODOS.MD`：

- 登录页面、统计页面 — 占位未实现
- 数据库层（SQLite + DAO）— 未开始
- 平台图标提取 — ExtractIconExW 完善
- 二进制补丁引擎 — 从 Python 迁移，待评估合法性
- SSL 握手 — 打包版 handshake_failure
- LoggerUtils.java — 待移植
- Handle 操作 — JNA 仿照 pywinhandle.py 重写约 300-500 行
- 登录状态 UI 接入 — 数据已自动维护落库（EventBus 流A），界面未展示
- 其余账号操作的事件驱动迁移 — 删除/批量/头像/快捷键等按 EventBus 流B 增量迁移

---

## 九、近期重大变更

### page-main 复制迁移（2026-07-04~05）

- **决策**: 从 page-manage 完整复制 DOM+JS 到 page-main 页面，`main.js` 复制自 `manage.js` 后做少量作用域修正
- **原因**: 逐段理解和改写容易出错，完整复制再逐步调试更可靠；且保持功能一致性
- **关键修改**:
  - `getEl(id)` 函数改为优先在 `#page-main` 内查找，后备到全局 `document.getElementById`
  - 移除了原管理页的"登录"和"管理"按钮，侧栏精简为平台列表 + 统计 + 设置入口
  - `manage.js` → `.old/manage.js` 归档保留历史
  - HTML 中删除了旧 `#page-manage` 区块

### 头像显示功能重构（2026-07-30）

- **背景**: 账号列表头像不显示，需将 Java 版本调整至与旧版 Python 一致的行为，同时适配用户可设置的自定义数据目录
- **新增**: `AvatarUtils.java` — 核心逻辑封装，获取顺序：本地文件 `{userDir}/{sw}/{acc}/{acc}.jpg` → URL 下载（以 `/0` 结尾）→ SVG 文字头像回退
- **路径使用**: `ConfigManager.getInstance().getUserDataPath()` 支持用户设置的数据目录
- **JsBridge** 中 `getAccountGroupData()` 改单行调用 `AvatarUtils.getAvatarDataUrl()`
- **CSS 调整**: `.manage-account-avatar` `border-radius` 从 `50%` → `6px`（圆角矩形）
- **SVG 文字头像**: 深灰背景 `#555` + 白色首字母，圆角矩形 `rx=6`

详细迁移过程及经验教训见 `MEMORY/DEV_LOGS.MD`。

### 账号列表来源接入磁盘扫描（2026-07-31）

- **背景**: 账号列表仅读 `SwAccData.json` 持久化记录，磁盘真实存在的账号目录/共存 exe 未记录时不显示
- **新增**: `JsBridge.getSwExistedAccounts(swId)` — 用 `SwConfigProvider.newAccessor()` 构建 `SwInfoFuncCore`，调用 `getSwAllAccountsExisted(swId, null)`（磁盘扫描：数据目录子目录 − 排除目录 + 共存 exe）
- **main.js** `loadAccountData` 改为：以磁盘扫描结果为账号来源，按账号 ID 与 `getSwDetailData`（SwAccData.json）详情合并；磁盘已删的残留记录不再显示
- **bridge.js** 新增 `getSwExistedAccounts` 包装

### 头像显示接入（2026-07-31）

- **背景**: 账户列表直接渲染原始 `avatar_url`，未走 AvatarUtils 管道，头像常不显示
- **新增**: `JsBridge.getAccAvatarAsync(swId, accountId, cbId)` — 后台线程先调 `AccInfoFuncCore.getAvatarFromCache`（截图缓存恢复），再用 `getAccAvatarFromFile`（→ `AvatarUtils.getAvatarDataUrl`：本地文件 → URL 下载(`/0` 结尾) → SVG 回退）
- **main.js**: 头像优先级 `avatar_data > avatar_url > 占位符`；`requestAccountAvatar` 异步加载 + `updateAccountAvatarCell` 逐行更新（SVG 回退不降级替换已有真实图片）
- **坑**: `_handleAsync` 第三参必须为字符串，推送 JSON 需双编码（见教训 #45）

### 日志目录修复与"日志"设置项（2026-07-31）

- **logback.xml**: `${jfmultichat.logdir:-logs}` 加默认回退，消除 `jfmultichat.logdir_IS_UNDEFINED` 目录
- **Launcher.java**: `jfmultichat.logdir` 设为 main 首行语句（任何 Logger 前）
- **设置页新增"日志"子项**: 打开日志目录 + 预留"上传日志"按钮；日志目录固定 `AppPaths.getLogsDir()`（`%APPDATA%\JhiFengMultiChat\{ver}\{Dev?}UserFiles\logs`，按 Dev/Prod 模式自动切换，不随用户配置）

### 账号展示名显示（2026-08-01，commit `c2561a4`）

- 账号列表"昵称"列头改为空白（讨论中用"展示名"列代表），`data-sort` 改 `display_name`
- `JsBridge.getSwDetailData` 每账号注入 `display_name` = `AccInfoFuncCore.getAccOriginDisplayName`（remark → nickname → alias → 账号 ID）
- `main.js`：展示名列/头像首字符/排序均改用展示名（前端兜底 `nickname`/`id`）

### SwAccData 账号自动填充（2026-08-01，commit `c2561a4`）

- `AccInfoFuncCore.syncSwAccAccounts(sw, accountIds)` — 遍历磁盘扫描账号列表，SwAccData 缺失节点自动补空
- `JsBridge.getSwExistedAccounts` 取得列表后调用（进入平台页即触发）

### EventBus 事件机制（2026-08-01）

- **新增 `core` 包**: `EventBus` 单例（subscribe/publish）+ `event/PlatformEnteredEvent`、`event/AccountDataChangedEvent`
- **流A 平台进入自动维护**: `selectPlatformInternal` 顶部 → `notifyPlatformEntered(swId)` → `PlatformEnteredEvent` → 后台执行登录态维护（`AccInfoFuncCore.resolvePidAccountMap` → `associateCoexistAccounts` → `updateAccLoginData`，拆分自原 god-method `getSwAccountsLoginStatus`，原方法保留为门面）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wfql1024/MultiWeChatManager](https://github.com/wfql1024/MultiWeChatManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
