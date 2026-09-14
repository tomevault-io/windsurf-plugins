---
trigger: always_on
description: AyuGram Desktop 是 Telegram Desktop 的 fork。本文档面向协作者和 AI 助手，说明项目结构、编码规范和构建流程。内容以本代码库的实际状态为准。
---

# AyuGram Desktop — 项目规范与架构

AyuGram Desktop 是 Telegram Desktop 的 fork。本文档面向协作者和 AI 助手，说明项目结构、编码规范和构建流程。内容以本代码库的实际状态为准。

---

## 项目定位

在 Telegram Desktop 之上叠加五类定制能力：

- **幽灵模式**：控制已读回执、在线状态、输入状态的发送时机
- **消息留档**：把已删除消息与编辑历史保存在本地
- **正则过滤**：按正则表达式隐藏消息（含隐藏已拉黑用户的消息）
- **文本处理**：中英文之间自动加空格、过滤异常组合字符
- **界面定制**：宽消息倍率、头像圆角、开关样式等

技术栈：C++20、Qt 5.15.19（静态编译，源码见 prebuild 的 `qt_5.15.19` 阶段）、rpl 响应式库、CMake 3.16+、nlohmann::json。

---

## 上游项目结构

改代码前先确认文件应该放在哪里。本节说明各目录的职责划分。

### 仓库骨架

```
AyuGramDesktop/
├── Telegram/                # 全部产品源码
│   ├── SourceFiles/         # 主源码（41 个顶层目录，见下）
│   ├── CMakeLists.txt       # 源文件在此逐个登记（ayu/ 同样逐文件列出）
│   ├── cmake/               # 平台与依赖的 CMake 模块
│   ├── codegen/             # 样式、emoji、TL scheme 的代码生成器
│   ├── Resources/           # 图标、音频、翻译等资源
│   ├── ThirdParty/          # 外部工具（msys2、gyp 等，由 prebuild 安装）
│   ├── lib_ui/              # 界面基础库（fork 的子模块，AyuGram 有改动）
│   ├── lib_tl/              # TL scheme 解析（fork 的子模块，改过 codegen）
│   ├── lib_base / lib_crl / lib_rpl / lib_storage / ...  # 其余 desktop-app 子模块，一般不改
│   └── 其余文件             # 均为上游原样
├── scripts/                 # prebuild.py / build.py（见构建章节）
└── .github/upstream.json    # 与上游的分叉基线记录
```

**子模块约定**：`lib_ui`、`lib_tl`、`codegen`、`cmake` 是 fork，改动必须记入 `.github/upstream.json` 对应条目的 `customised_paths`。其余 `lib_*` 视为只读依赖。

### SourceFiles 分层

按职责分四层理解（目录名沿用 tdesktop 原名，改动上游文件需要登记）：

**进程与生命周期**

| 目录 | 职责 |
|---|---|
| `main` | Account / Session / Domain 生命周期，多账号 |
| `core` | `Core::App()` 全局单例，启动与退出流程，沙箱与更新调度 |
| `platform` | Windows / macOS / Linux 的平台差异（通知、托盘、字体） |
| `_other` | 打包、更新器、开机自启等安装周边 |

**数据层**

| 目录 | 职责 |
|---|---|
| `mtproto` | MTProto 协议：数据中心列表、加密连接、授权状态 |
| `api` | 主动请求的语义化封装（`ApiWrap` 一族），调用 MTProto 但不处理协议细节 |
| `data` | 内存数据仓库：`PeerData` / `UserData` / `HistoryItem` / `Session`，全部在主线程 |
| `storage` | 本地持久化：缓存、数据库、加密的 tdata |
| `tde2e` | 端到端加密库 |

**聊天界面层**

| 目录 | 职责 |
|---|---|
| `window` | 主窗口、会话导航、窗口控制器 |
| `history` | 消息列表渲染：气泡，`HistoryItem`（数据）与 `Element`（视图）分离 |
| `dialogs` | 左侧会话列表 |
| `chat_helpers` | 输入框、剪贴板、机器人交互辅助 |
| `media` | 媒体查看器与播放器（view / player 子目录） |
| `overview` | 聊天内媒体总览标签页 |
| `layout` | 媒体网格布局基类（overview / photo 共用） |
| `editor` | 图片与视频编辑 |
| `iv` | 即时预览页面 |
| `boxes` | 通用弹窗（editors / pickers / confirm） |
| `info` | 右侧信息面板（个人资料 / 媒体 / 管理员） |
| `calls` | 通话 |
| `settings` | 官方设置页 |
| `intro` | 登录引导 |
| `profile` / `statistics` / `payments` / `passport` / `export` / `support` / `poll` / `inline_bots` / `webauthn` | 各自独立的领域功能 |

**支撑层**

| 目录 | 职责 |
|---|---|
| `ui` | 通用控件与样式应用层（基础控件在 lib_ui） |
| `lang` | 翻译加载（`tr::lng_*`） |
| `countries` / `ffmpeg` / `menu` / `codegen` / `test(s)` | 国家码、FFmpeg 封装、菜单、内嵌生成、测试 |

### ayu/ 定制层

AyuGram 新增的代码集中在 `Telegram/SourceFiles/ayu/`：

```
ayu/
├── ayu_infra.cpp              # 初始化入口（翻译/数据库/界面/工作线程/翻译器/调试服务端）
├── ayu_settings.{h,cpp}       # 全部设置项（rpl::variable + JSON 序列化）
├── ayu_state.{h,cpp}          # 跨组件的运行时状态
├── data/                      # SQLite 留档库与上层封装
├── features/                  # 业务功能，一个功能一个子目录
│   ├── auto_space/            # 中英文之间自动加空格
│   ├── filters/               # 正则过滤与隐藏（含幽灵拉黑名单）
│   ├── forward/ message_shot/ streamer_mode/ translator/
├── debug/                     # 调试服务端（仅 _DEBUG 编译），commands/ 一个领域一个文件
├── ui/                        # ayu 的控件与设置页
├── utils/                     # Session / Peer 转换、远程配置
└── libs/sqlite/               # 内嵌 SQLite
```

**改动位置对照表**：

| 要做的事 | 放在哪里 | 登记要求 |
|---|---|---|
| 新增 AyuGram 功能 | `ayu/features/<名称>/`，并在 `Telegram/CMakeLists.txt` 的 `ayugram_files` 逐文件添加 | — |
| 新增设置项 | `ayu_settings.{h,cpp}`：成员、`to_json`、`from_json` 三处同步 | — |
| 设置项的界面 | `ayu/ui/settings/`，入口注册在 `settings_main.cpp` | — |
| 需要持久化的数据 | `ayu/data/` | — |
| 新增调试指令 | `ayu/debug/commands/<领域>_commands.cpp` + CMake 登记 | 必须包在 `#ifdef _DEBUG` 里 |
| 修改上游行为（渲染、菜单等） | 直接改上游文件 | 记入 `.github/upstream.json` |
| 界面基础控件改动 | `Telegram/lib_ui/` | fork 仓库与 upstream.json 两处都要 |

---

## 编码规范

### 命名

| 元素 | 规则 | 上游实例 |
|---|---|---|
| 局部变量 | 小驼峰，优先 `const auto` | `dataName`、`phone`、`flags` |
| 成员变量 | 小驼峰加 `_` 前缀 | `_lastseen`、`_peerGiftsCount` |
| 常量 | `k` 前缀加大驼峰 | `kWideIdsTag` |
| 类与结构体 | 大驼峰 | `HistoryItem` |
| 函数 | 小驼峰（ayu 代码必须遵守；上游风格混杂，不做统一改造） | `processUser` |
| 命名空间 | 大驼峰或匿名 | `AyuInfra`、`namespace { ... }` |
| 文件名 | 小写加下划线，前缀与所属领域一致 | `data_user.cpp`、`history_item.cpp` |

### 注释

- 简体中文，说清意图即可，不使用行话和缩略语
- **最多两行**，一行能说清就写一行
- 只写意图、约束、边界条件，不复述代码本身在做什么
- 单行注释用 `//`，函数注释写在声明或定义上方

```cpp
// 构造本地假会话绕过登录，不写入磁盘，重启后消失。
[[nodiscard]] Result FakeSession(const QStringList &args);
```

### 优先使用卫语句

先排除异常情况提前返回，正常流程靠左对齐，避免深层嵌套。上游代码普遍是这个形态：

```cpp
// 正确
void Process(const TextWithEntities &text) {
	if (text.empty()) {
		return;
	}
	const auto entity = text.entities.front();
	// ...正常逻辑，无嵌套
}

// 错误：把正常流程包进 else
void Process(const TextWithEntities &text) {
	if (!text.empty()) {
		const auto entity = text.entities.front();
		// ...正常逻辑多包了一层
	}
}
```

### 不要过度防御

- 上游核心代码（`data_session.cpp`、`history.cpp` 等）完全没有 `try/catch`——**异常不是这个项目的错误处理方式**，不要引入
- 用 `Expects()` / `Ensures()`（GSL）表达契约，上游大量使用；前置条件由调用方保证时，被调方断言即可，不需要双向判空
- 只在真正可能失败的地方校验；写任何兜底分支前先确认这个分支现实中会走到
- 本地无法处理的错误交给上层（返回空值或错误码），不要直接忽略

### 响应式（rpl）

- 所有设置项都是 `rpl::variable<T>`：一次性读取用 `current()`（返回引用），订阅变化用 `value()`（返回数据流）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kindness-Kismet/AyuGramDesktop-Plus](https://github.com/Kindness-Kismet/AyuGramDesktop-Plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
