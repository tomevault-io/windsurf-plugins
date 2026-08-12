---
trigger: always_on
description: 本地服务监控与快速启动控制台。**零依赖**：Python 3 标准库后端（单文件）+ 无构建原生前端。推荐双击 `总控台.app` 后台运行（不显示 Terminal/Dock）；`start.command` 保留为终端调试入口。
---

# 总控台 (Console)

本地服务监控与快速启动控制台。**零依赖**：Python 3 标准库后端（单文件）+ 无构建原生前端。推荐双击 `总控台.app` 后台运行（不显示 Terminal/Dock）；`start.command` 保留为终端调试入口。

## 结构

- `server.py` — 后端（单文件，仅标准库，Python 3.12）
- `static/index.html` / `static/app.js`（入口）/ `static/js/{core,launchpad,services,overlays,ports,widgets}.js`（原生 ES Modules，无构建）/ `static/icons.js` — 前端（原生，禁框架/CDN/构建）；`core.js` 承载工具/API/浮层/状态/主题注册，`launchpad.js` 卡片+拖拽+诊断+启动台 KPI/分区过滤，`services.js` 表格+监控 KPI 火花线，`overlays.js` 模态+抽屉，`ports.js` 端口归一化纯函数，`widgets.js` 右侧信息栏（实时动态/告警、TOP5、小贴士、快捷操作）与导航轨状态；模块间用 `window.__poll` 共享轮询入口
- 布局 v2：左侧 `.rail` 图标导航轨（启动台/服务监控视图切换 + 日志中心/设置中心弹层入口）+ 顶栏 + 内容/右侧信息栏双栏网格（≤1280px 侧栏下沉到底部、≤900px 导航轨隐藏）；结构样式集中在 `static/base.css` 末尾「布局 v2」段（主题令牌驱动），主题包负责视觉皮肤
- `static/themes/` — **单一主题**：当前仅内置 `ops`（指挥台，`DEFAULT_UI_THEME` 常量指定并在清单中固定排首位）。`{id}.css` 整包样式 + `{id}.json` 清单（`id/name/author/desc/colors[]`）的注册机制保留：`GET /api/state` 返回 `themes` 与 `uiTheme`；`POST /api/ui/theme {theme}` 校验 id 后落盘。产品不提供主题选择界面（已随多主题一并移除），深浅色切换仍保留。
- `static/fonts/GeistMono-Variable.woff2` — vendored 数据/代码字体；中文与正文使用 macOS 系统字体栈；`static/icons/*.svg` — Lucide 图标源文件（vendored）；`tools/gen_icons.py` — 由 svg 重新生成 `icons.js`（勿手改 icons.js）
- `static/assets/` — 品牌素材：`console-app-icon.png` 为 App Icon 主图，`brand-mark.png` 为顶栏标识；`favicon-32.png` / `favicon.ico` / `apple-touch-icon.png` 与 `.app` 内 `AppIcon.icns` 由 `tools/gen_brand_assets.py` 生成
- `~/Library/Application Support/总控台/config.json` — 用户配置；`icons/` 为应用图标。目录/ 文件权限分别为 0700/0600
- `~/Library/Logs/总控台/{appId}.log` — 应用启动日志；`console.log` 为 `.app` 启动日志
- `data/` — 旧版项目内数据，仅在新目标不存在的首次启动中复制迁移；保留不删除
- `start.command` — macOS 双击启动脚本（chmod +x）
- `总控台.app` — macOS 无终端窗口启动器（`LSUIElement` 后台应用；内部直接启动 `server.py`，输出写入 `~/Library/Logs/总控台/console.log`）

## 运行

`python3 server.py` → 绑定 `127.0.0.1`，端口从 **9600** 起尝试，被占则 +1（最多 10 个）。启动后自动打开浏览器。`/favicon.ico` 返回统一品牌图标。双击 `总控台.app` 会先识别同目录的现有总控台，可直接打开或安全重启，不需要用户输入命令，也不会出现 Terminal 窗口。

## API 契约（全部 JSON；icon 上传为原始字节）

### `GET /api/state` — 前端唯一轮询接口
```json
{
  "services": [{
    "key": "python3.12:8791", "instanceKey": "54252:8791",
    "pid": 54252, "name": "python3.12", "port": 8791,
    "cwd": "/Users/example/xx项目", "project": "xx项目", "cmd": "python3 app.py",
    "cpu": 0.3, "mem": 1.2, "uptimeSec": 7980,
    "group": "mine", "pinned": false, "hidden": false, "promoted": false,
    "appId": null, "appName": null,
    "origin": {"label": "Codex", "icon": "bot"}
  }],
  "watched": [{"pid": 1, "name": "ffmpeg", "cmd": "...", "cpu": 0.0, "mem": 0.5, "uptimeSec": 60, "keyword": "ffmpeg"}],
  "apps": [{
    "id": "a1b2c3d4", "name": "我的博客", "command": "python3 -m http.server 8080",
    "cwd": "/path", "port": 8080, "emoji": "🚀", "glyph": "rocket", "icon": "/icons/a1b2c3d4.png",
    "kind": "service", "attached": false,
    "running": true, "pid": 1234, "uptimeSec": 120,
    "listening": true, "portOccupied": false, "portOccupiedPid": null,
    "portConflict": false, "portConflictApps": [],
    "lastExit": {"status": "succeeded", "code": 0, "at": 1700000000, "startedAt": 1699999998750, "durationSec": 1.25},
    "health": {"status": "ok", "blocking": false, "issues": []},
    "favicon": "/icons/fav-a1b2c3d4.png",
    "ports": [8080],
    "listening": true, "portOccupied": false, "portOccupiedPid": null,
    "portOwner": null, "portConflict": false, "portConflictApps": []
  }],
  "watchedKeywords": ["ffmpeg"],
  "consolePort": 9600, "consolePid": 123, "consoleCwd": "/path/to/总控台",
  "version": "1.0.0", "schemaVersion": 1,
  "degraded": false, "degradedReasons": []
}
```
- `GET /api/health` — 不运行 `ps/lsof` 的轻量健康检查，返回 `status/version/schemaVersion/degraded/issues/config`
- `group`: `"mine"` | `"background"`；`icon`/`emoji`/`port`/`cwd`/`project`/`appId`/`appName`/`lastExit` 可为 `null`
- `lastExit`：最近一次退出结果。任务状态为 `succeeded`（exit 0）/`canceled`（脚本主动 exit 130）/`failed`（其他自然退出）/`stopped`（总控台中止，code=null）；旧数据可能只有 `code/at`，API 输出时会兼容推导但不改写磁盘。批处理启动时保留上一次完成历史，自然退出或中止后覆盖
- `health`：每次状态读取时只读检查配置，返回 `status: ok|error|unknown`、`blocking` 与 `issues[{kind,severity,title,detail,fix,action}]`。明确缺失的 cwd、脚本或运行时会阻止启动；复杂 Shell 命令无法静态判断时为 unknown，不阻止运行
- `kind`：`"service"`（长期服务，有端口语义）| `"task"`（批处理任务，强制 port=null，主按钮为「运行」）；旧数据缺省视为 `service`。启动台按 kind 分两个区渲染
- `running`：仅表示存在通过本次启动 token、进程组与当前用户三重校验的受控进程；不再以“配置端口有任意监听者”作为运行依据
- `attached`：用户从服务监控明确认领的外部服务身份。此类服务的监听子进程换 PID 后，可按配置端口 + 当前 UID + 真实 cwd 唯一重新关联；普通卡片仍不得仅凭端口自动认领
- 服务行的 `key` 保持 `name:port` 以兼容隐藏/置顶配置；`instanceKey` 使用 `pid:port` 区分同名同端口后来出现的新进程实例，前端发现与 DOM 对账均使用它
- `listening`：受控进程是否正在监听配置端口；`portOccupied`：该端口当前被不属于本卡片的进程占用；多张卡片允许保存同一个常见开发端口，`portConflict/portConflictApps` 仅为旧前端兼容字段并固定返回 `false/[]`；`legacyManaged`：是否通过旧版 PID+端口+UID+cwd 兼容身份识别
- `project`：cwd 最后一段目录名（用于区分同名进程）；`appId`/`appName`：该端口命中启动台应用时的关联信息
- 排除控制台自身进程；只返回当前用户的进程
- **进程溯源**：`origin` 沿 PPID 链（≤12 层）识别启动者——跳过壳/包管理器/运行时包装层与 launchd，优先匹配已知 AI 编程助手（codex/claude/kimi/gemini/aider/opencode 等）、`.app` 包（VS Code/Cursor/iTerm/Warp 等）、tmux/screen 与总控台 run-token 标记（「总控台」）；未识别的中间层先记为候选、有更优答案即覆盖，全部落空才以最近未识别进程命名；`label` 为展示名、`icon` 取 bot/code/terminal/package/rocket/server，仅用于展示，不影响启停判定

### 服务操作
- `POST /api/kill` `{pid, force?}` → `{ok}` / `{ok:false, error}`（force 用 SIGKILL；校验属当前用户）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laogou717/local-ops](https://github.com/laogou717/local-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
