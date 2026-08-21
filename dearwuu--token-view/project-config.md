---
trigger: always_on
description: > ## 🔴 开发维护规范（超高优先级——任何改动前必读）
---

# AGENTS.md

> ## 🔴 开发维护规范（超高优先级——任何改动前必读）
>
> 以下每条都对应真实踩过的事故（2026-08 密集修复总结）。违反任何一条，
> 问题会以"时好时坏、改了复发"的形态回来。
>
> ### 一、坐标系铁律（本项目第一事故源）
> 1. CSS 逻辑像素 ↔ Win32 物理像素的换算**只用前端 `devicePixelRatio`**，
>    后端永远不自猜——`GetDpiForWindow/GetDpiForSystem` 与 Chromium dpr
>    可能差 1.25 倍（文本缩放/WebView2 raster 差异，实测 1.5 vs 1.875）。
> 2. pywebview 的 `window.x/y/width/height` 属性是**创建时快照**，不随
>    Win32 SetWindowPos 更新；读当前位置尺寸必须 `GetWindowRect`。
> 3. 持久化的 geometry 统一物理像素，消费处换算回 CSS；任何新持久化
>    数值必须注明单位（物理/css/秒/毫秒）。
> 4. 自动化测试脚本（PIL 截图/SetCursorPos/GetWindowRect）先
>    `SetProcessDpiAwareness(2)`，否则坐标系错位导致"测试误报"。
>
> ### 二、测量铁律（防反馈回路）
> 5. 测量内容尺寸前必须解除一切容器约束（height/maxHeight/overflow/
>    `min(--panel-width, 100vw)` 钳制/flex 拉伸），量完恢复；测量结果
>    不允许被当前窗口尺寸污染（"窗口越大量得越大"）。
> 6. intrinsic sizing 陷阱：`flex-basis:0`、`overflow:hidden+ellipsis`
>    的元素在 fit-content/max-content 测量下会塌缩成 0。
>
> ### 三、异步与事件铁律
> 7. 关键时序**禁用 `requestAnimationFrame`**（无焦点窗口被挂起永不返回），
>    用同步 reflow（`void el.offsetHeight`）或 setTimeout。
> 8. 悬浮窗按钮一律 `pointerdown`，不用 `click`（down/up 配对在窗口移动
>    时失效＝"点两三次才生效"且改了复发）。
> 9. 所有状态锁（防重入/dragging）必须有丢失兜底：超时解锁、
>    `e.buttons===0` 检测、try/finally。
> 10. WebView2 改窗口位置/尺寸后会**重派** mouseenter/mouseleave/mousemove，
>     事件驱动状态机必须容忍重派（去重/延迟/坐标判断）。
>
> ### 四、状态与配置铁律
> 11. 前端→后端**只提交用户编辑的字段，禁止整快照回传**（页面加载时的
>     token/凭证会被后台刷新，回传即覆盖）。后端防线：`api/providers.update`
>     拒绝凭证字段被空值覆盖。
> 12. 配置读-改-写必须持 `config.io_lock`（save 原子但读改写窗口期会丢
>     更新：token 轮换 vs 增删 provider 实踩）。
> 13. **"每次重存"而非"为空才存"**（异常退出的残留值会自我延续）。
> 14. 状态位保持正交：compact（密度）/topMode（布局）/dockMode（吸附）/
>     dockHidden（临时可见性）各司其职，函数只动自己职责内的位；
>     失败必须回滚到一致状态，不留"半状态"。
> 15. 退出/关闭时保存配置，必须先重新 `config.load()` 再改
>     （内存快照是启动时的，直接 save 会擦掉运行期全部变更）。
> 16. 共享凭证（kimi 个人/团队同一份 localStorage token）：刷新后轮转
>     必须**同步写给所有共享者 + 使用方 fetch 前 reload**，否则互踢 401。
>
> ### 五、外部依赖铁律
> 17. 端口/地址可被其他程序抢占（Electron 抢 9222 实踩）：连接后必须
>     验证目标身份（目标页面出现），失败给明确指引；`CDPHarness.find_page`
>     对 127.0.0.1/[::1] 自适应是参考实现。
> 18. 站点会改版：选择器/URL 关键字放宽匹配；凭证机制抽象成
>     extract/refresh 两段可替换。
> 19. **不凭单次现象下结论**（"Chrome 151 移除 origin 头"实为连到
>     Electron）；先定位根因再改代码。
>
> ### 六、变更纪律
> 20. **修 bug 不改样式**——用户认可的视觉样式（宽度/布局/grid）是冻结
>     资产，逻辑修复与样式调整必须分开，不得夹带。
> 21. 改窗口行为必须真机验证：SendInput 模拟点击 + 截图确认，
>     不能只靠读代码推断"应该好了"。

---

桌面悬浮窗，实时显示 coding plan 的 token 用量。**pywebview** 实现（HTML/CSS/JS +
Python 桥），支持**智谱 GLM Coding Plan（含团队版）**、**OpenCode Go**、**Kimi**、
**小米 MiMo**、**火山 Ark**。行业类似项目（Headroom / claude-statusbar /
claude-monitor）均不用 Qt，证明这条路线是对的。

**数据抓取不依赖浏览器常开**：各 provider 首次从调试 Chrome **一次性提取凭证**
（设置页「提取凭证」按钮），之后日常刷新走纯 HTTP 直连（已实测无反爬，见下文）。
CDP 仅用于凭证提取和直连失败时的兜底。

## 运行

- 启动：`python -X utf8 main.py`（Windows 控制台必须加 `-X utf8`，否则中文乱码）
- 依赖：`pywebview` / `requests` / `websocket-client` / `pillow` / `pystray`（requirements.txt）。
  本机已装；缺则跑 `pip install -r requirements.txt`
- 测试：`pytest tests/`（14 个用例，凭证判定/fetch 路由/解析，无 GUI）
- 打包：`pyinstaller TokenView.spec --clean --noconfirm` → `dist/TokenView.exe`（22MB）
- 验证：跑 `main.py` 看效果 + 看 `%APPDATA%/token_view/debug.log`

## 目录结构

```
.
├── main.py                    # pywebview 入口：创建窗口 + 启动
├── config.py                  # 配置 JSON 读写（pathlib + 原子写）
├── logger.py                  # 统一日志到 %APPDATA%/token_view/debug.log
├── requirements.txt
├── TokenView.spec             # PyInstaller 打包配置
│
├── providers/                 # 用量数据源（每个 provider 独立文件）
│   ├── base.py                #   BaseProvider / UsageData / UsageItem / fmt_tokens
│   ├── cdp.py                 #   CDPHarness + CDP 异常类（三个 provider 共用）
│   ├── zhipu.py               #   智谱 GLM（CDP/cookie/API key 三模式）
│   ├── kimi_team.py           #   Kimi 团队空间（refresh_token 直连，继承 KimiProvider）
│   ├── opencode.py            #   OpenCode Go（CDP 模式）
│   ├── mimo.py                #   小米 MiMo（CDP 模式）
│   └── __init__.py            #   build() 工厂 + 重导出
│
├── api/                       # pywebview js_api 桥层
│   ├── core.py                #   Api 类（编排下面所有模块）
│   ├── chrome.py              #   Chrome 查找 + CDP Chrome 启动
│   ├── screen.py              #   跨平台屏幕工作区 / Win32 HWND / DPI
│   ├── window.py              #   窗口几何（move/resize/置顶/auto-hide）
│   ├── providers.py           #   Provider JSON 配置 CRUD
│   ├── state.py               #   state.json 协议（Headroom 风格）
│   ├── settings.py            #   设置窗口 + 模式/刷新间隔
│   └── __init__.py            #   暴露 Api 类
│
├── web/                       # 前端（HTML/CSS/JS）
│   ├── index.html             #   主面板（8 方向 resize handles）
│   ├── app.js                 #   主逻辑：刷新 / 模式 / resize / auto-hide dock / 假透明度
│   ├── style.css              #   主题 + 顶部模式 + 假透明度 CSS 变量
│   └── settings.html          #   设置页
│
├── tests/                     # pytest 单元测试（27 个用例）
│   ├── conftest.py
│   ├── test_base.py           # 数据模型 + 工厂
│   ├── test_state.py          # state.json 协议
│   └── test_config.py         # 配置读写
│
├── docs/                      # 文档
│   ├── architecture.md        # 架构图 + 数据流
│   └── archive/               # 历史任务交接文档（不再反映当前实现）
│       ├── PROMPT-cdp-攻坚.md
│       └── README.md
│
├── README.md
└── AGENTS.md                  # 本文件
```

## 架构（big picture）

**渲染层**：pywebview 把 `web/index.html` 跑在 WebView2（Windows）/WKWebView（macOS）/
WebKitGTK（Linux）里。CSS 处理圆角/阴影/暗色/DPI/8 方向 resize，JS 调
`window.pywebview.api.xxx` 拿数据。

**桥层**：`api/core.py:Api` 注入到 `webview.create_window(js_api=...)`。Api 是无下划线

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DearWuu/token_view](https://github.com/DearWuu/token_view) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
