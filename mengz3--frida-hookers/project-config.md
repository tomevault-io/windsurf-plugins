---
trigger: always_on
description: - 项目名：`Frida-Hookers`
---

# AGENTS.md

## 速览

- 项目名：`Frida-Hookers`
- 主要入口：`app_gui.py`
- 次入口：`hookers.py`
- 当前产品形态：**GUI-first**；CLI 仍可用，但最近的用户可见行为都以 GUI 为准
- 核心共享状态：`core/models.py` 中的 `HookerContext`
- 主要目录分工：
  - `core/`：设备、工作区、会话、RPC、APK 扫描等服务层
  - `ui/`：GUI 组装、控制器、错误展示、日志面板、线程切回与后台 worker
  - `tests/`：本地轻量回归测试
  - `workspaces/`：按包名隔离的运行时工作区，不是框架源码
  - `hookers/js/`：GUI 当前实际使用的内置 Frida 脚本、RPC 资产、参数化脚本模板、清理 warp；也包含 `bypass_frida_svc_detect.js`、`replace_dlsym_get_pthread_create.js` 这类无单独快捷按钮的专项内置脚本
  - `mobile-deploy/`：设备侧二进制与本地辅助工具

## 文档编码约定

- 项目文档默认统一使用 **UTF-8（无 BOM）**
- `.editorconfig` 已显式约定：
  - `charset = utf-8`
  - `end_of_line = lf`
- `*.md` 同样按 UTF-8 保存，不单独使用 GBK / ANSI / UTF-16
- 如果 shell 里看到中文乱码，不要先假设文件已损坏；优先用 Python 按 UTF-8 读取确认，再判断是否只是终端显示编码问题

## 只看少数文件时，优先看这些

1. `app_gui.py`
   - GUI 运行时总装配入口
   - 构建 `HookerContext` 与各个 service
2. `ui/main_window.py`
   - 主窗口 UI 构建与公共 UI helper
3. `ui/composition.py`
   - controller / presenter 的实例化与 signal wiring
4. `core/models.py`
   - 理解共享状态、当前 App、当前会话、工作区目录的最好入口
5. `ui/quick_hook_actions.py`
   - GUI 快捷脚本按钮的共享配置表入口
   - 普通快捷脚本主要看这里；参数化按钮还要联动 `ui/hook_runtime.py` 与 `ui/composition.py`
6. `core/device_service.py`
   - 设备准备、Frida server 生命周期、App 运行态 façade
7. `ui/terminal_console.py`
   - GUI 右侧内嵌 CLI 终端控制器
   - 命令解析、历史记录、CLI 模式切换、实时补全、Tab 补全应用、REPL transcript 风格回显、App 级 CLI 分发
8. `ui/cli_terminal_view.py`
   - 黑色终端本体输入控件
   - 负责活动输入行、prompt 保护、历史区只读保护、命令提交信号、Tab/上下键转发

如果再多看几个文件，优先看：

- `core/session_service.py`：attach/spawn、脚本加载、会话生命周期
- `core/workspace_service.py`：工作区初始化、本地 APK、内置脚本清理、运行时脚本落地
- `core/rpc_service.py`：RPC 检索与 Hook 脚本生成
- `ui/error_presenter.py`：GUI 统一错误展示策略
- `ui/app_workflow.py`：准备环境、选 App、初始化工作区
- `ui/hook_runtime.py`：开始注入、停止 Hook、停止 server、重启 App、参数化快捷脚本弹窗
- `ui/log_panel.py`：右侧日志渲染、过滤、搜索、落盘
- `ui/terminal_console.py`：右侧 CLI 终端命令层、补全、历史、CLI 模式切换与命令分发
- `ui/cli_terminal_view.py`：黑色终端本体、活动输入行、prompt 保护、历史区只读保护与输入事件转发
- `ui/ui_thread_dispatcher.py`：把 worker / Frida 后台线程回调安全切回 GUI 主线程
- `ui/workers/*.py`：GUI 后台动作执行器；区分设备准备、Hook 启动、工作区初始化与通用一次性动作

## 当前架构模型

### 共享状态

- `HookerContext` 是唯一共享运行时上下文
- service 通过同一份 context 协作
- controller 尽量不直接互相 import 调用，而是通过最小能力注入协作

### `core/` 分层

- `device_service.py`
  - 当前已经是 **façade + 3 个内部职责组件**：
    - `_DeviceBridge`：ADB/su、远端文件、forward
    - `_FridaServerManager`：Frida device、server 生命周期、端口、probe、日志
    - `_AppRuntimeInspector`：App 元数据、前台状态、PID、应用列表、radar.dex
  - 外部世界仍只通过 `DeviceService` 调用
- `workspace_service.py`
  - 按包名创建/补齐工作区
  - 准备本地 APK
  - 管理工作区脚本、旧内置脚本清理、解密输出与参数化运行时脚本落地
- `session_service.py`
  - attach / spawn / script load / resume / stop / detached 处理
  - 注入宿主侧 `Hookers` 结构化日志桥
- `rpc_service.py`
  - 复用 `hookers/js/rpc.js` 做高频对象/页面/组件查询
  - 生成 Hook 脚本
- `apk_scan_service.py`
  - 本地 `ApkCheckPack.exe` APK 扫描
  - 只负责校验输入路径并执行外部 exe，返回 stdout/stderr/returncode
- `errors.py`
  - 结构化错误模型
  - 统一转成 GUI 可消费的 `UiErrorPayload`

### `ui/` 分层

- `main_window.py`
  - 负责 UI 构建
  - 持有公共 UI helper：`set_busy()`、`set_status_text()`、脚本列表相关 helper、`toggle_log_focus_mode()`、`closeEvent()`
  - 负责快捷按钮分组布局、专项按钮 tooltip、当前 Attach / Spawn 模式徽章
- `quick_hook_actions.py`
  - 快捷脚本按钮的配置驱动入口
  - 按钮文案、脚本名、日志模板、tooltip、功能分组等基础配置都在这里收口
  - 参数化按钮也会在这里登记，但仍需要 `hook_runtime.py` / `composition.py` 的专门分支
- `composition.py`
  - 负责 `MainWindow` 的 controller / presenter 装配
  - 负责 signal wiring
  - 不承载业务逻辑
- `error_presenter.py`
  - 统一消费 `UiErrorPayload`
  - 负责 warning / critical / 状态栏 / 日志 / 弹窗展示策略
- `app_workflow.py`
  - 准备环境并刷新 App
  - 前台 App 自动选中
  - 工作区初始化
- `hook_runtime.py`
  - attach / spawn 开始注入
  - 停止 Hook
  - 停止 Frida Server
  - 重启 App
  - detached 后状态恢复
  - 为特定快捷脚本提供参数弹窗与运行时脚本生成
- `rpc_tools.py`
  - RPC 工具动作、结果展示、结果弹窗
- `terminal_console.py`
  - 右侧 GUI 内嵌 CLI 终端命令层
  - 当前支持 CLI 模式切换、当前 App 命令模式、顶层 App 选择辅助命令、上下键历史、实时补全、Tab 补全应用与 REPL transcript 风格回显
  - 终端查询结果统一写入现有日志面板，不单独维护第二套输出缓冲
- `cli_terminal_view.py`
  - 黑色终端本体控件
  - 负责活动输入行、prompt 保护、历史区只读保护、命令提交信号、Tab/上下键转发与日志追加时的 prompt 保留
- `apk_scan.py`
  - 左侧 APK 扫描流程
- `log_panel.py`
  - 右侧日志缓冲、过滤、搜索、增量渲染、落盘
- `ui_messages.py`
  - 高频 GUI 文案
- `controller_types.py`
  - GUI 侧协议、typed payload、callback alias
  - 把 MainWindow / controller / service 之间的依赖边界显式类型化
- `ui_thread_dispatcher.py`
  - 用 Qt Signal 把任意 Python 回调安全切回 owner 所在线程
  - 避免 worker、Frida 回调线程直接操作 Qt 控件
- `workers/action_worker.py`
  - 通用一次性后台动作执行器
  - 被 RPC、APK 扫描、停止 Hook、重启 App 等流程复用
- `workers/device_worker.py`
  - 后台执行 connect / start_frida_server / deploy_radar_dex / refresh_applications
- `workers/hook_worker.py`
  - 后台执行 attach / spawn 启动链路
  - 启动失败时负责兜底 stop_active_session 清理
- `workers/workspace_worker.py`
  - 后台执行 prepare_app_context + ensure_workspace
  - 不强制把目标 App 拉到前台

## 当前行为真相（非常重要）

- GUI 是当前主要产品表面；当代码与 README 不一致时，以代码为准
- Attach **只会** attach 到已经运行的 App，不会主动拉起目标 App
- 选中 App 不会自动初始化工作区
- 只有点击 `初始化工作目录并刷新列表` 才会真正：
  - 生成工作区目录
  - 准备本地 APK
- GUI 启动时默认脚本根目录是 `hookers/js`；选中 App 后会切到 `workspaces/<package>/js`
- GUI 快捷按钮默认从 `hookers/js` 读取内置脚本；但参数化脚本会先在 `workspaces/<package>/js` 生成运行时副本再启动
- 左侧脚本列表在未选 App 时指向 `hookers/js`；选中 App 后默认指向 `workspaces/<package>/js`，但用户仍可手动切换脚本根目录
- `workspaces/<package>/js` 中已有脚本在重复初始化工作目录时会被保留；当前初始化流程不会主动清理或删除原有脚本，只会补齐辅助资源、刷新本地 APK，并把 `hookers/js` 里的内置脚本复制到工作区，文件名前缀为 `内置-`
- `准备环境并刷新 App` 后，GUI 会尝试识别当前前台 App；如果该包名存在于刷新后的应用列表里，则自动选中
- 如果没有可识别的前台 App，则保持 App 选择为空
- 当前固定使用单一设备侧 server：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mengz3/Frida-Hookers](https://github.com/Mengz3/Frida-Hookers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
