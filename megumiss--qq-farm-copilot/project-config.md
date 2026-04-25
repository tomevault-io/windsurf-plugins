---
trigger: always_on
description: 本文件定义本仓库内自动化/编码代理的工作约定。以当前代码实现为准。
---

# AGENTS

本文件定义本仓库内自动化/编码代理的工作约定。以当前代码实现为准。
- 一切修改以工作区最新内容为准。
- 除非显示说明否则所有回答交互均使用中文。
- 每次修改完后，使用项目 `.venv` 下的 `ruff format` 进行代码格式化。
- 出现问题时优先定位并修复根因，不要只靠代码推断原因，不要以“兜底/兼容分支”替代真实修复；临时兜底仅可作为明确标注的短期措施。
: 仅格式化 Python 文件，跳过 `json/md` 等非 Python 文件（避免改坏 JSON 语法与文档排版）。
: 推荐命令：`.\.venv\Scripts\python.exe -m ruff format core gui models tasks utils main.py private\main_window_core.py`

- `private/main_window_core.py` 与 `gui/main_window_core.pyd` 的关系与更新方式：
: `private/main_window_core.py` 是 Fluent GUI 私有源码；`gui/main_window_core.pyd` 是发布/默认加载的二进制产物。
: GUI 更新流程（必须遵守）：先改 `private/main_window_core.py`，验证通过后执行 `.\private\build_main_window_core_pyd.ps1` 重新编译并覆盖 `gui/main_window_core.pyd`。
: 首次编译先执行：`.\private\build_main_window_core_pyd.ps1 -InstallDeps`。


## 0. 当前状态

- 项目名：`QQ Farm Copilot`
- UI 结构：左侧截图预览 + 中间实例运行面板 + 最右侧竖向实例栏
- 实例纳管操作：`新增 / 删除 / 切换 / 克隆 / 重命名`
- 调度模式：`TaskExecutor` 单线程串行执行
- 任务配置：`%APPDATA%/QQFarmCopilot/instances/<instance_id>/configs/config.json -> tasks`（动态字典，包含持久化 `next_run`）
- 农场详情配置：`config.land.plots`（固定 24 格，元素结构：`{plot_id, level, maturity_countdown, need_upgrade, need_planting}`；`maturity_countdown` 为 `HH:MM:SS`，`need_upgrade` 表示地块是否可升级，`need_planting` 表示地块是否需要播种）与 `config.land.profile`（`level/gold/coupon/exp`，来源于等级同步 OCR）
- 好友黑名单配置：`config.tasks.friend.features.blacklist`（`list[str]`，在任务设置详情弹窗维护）
- 偷取统计开关：`config.tasks.friend.features.steal_stats`（默认 `false`；开启后仅在偷取动作后执行 OCR 统计，偷取速度会变慢）
- 高级配置：`config.safety.debug_log_enabled` 控制 Debug 日志输出
- 截图频率：`config.screenshot.capture_interval_seconds`（默认 `0.3` 秒；`0` 表示不限制最小截图间隔）
- 播种稳定超时：`config.planting.planting_stable_timeout_seconds`（默认 `3.0` 秒；用于背景树锚点稳定等待超时）
- 播种选种：`config.planting.warehouse_first` 默认开启；开启时优先按 `BgPatchNumberOCR` 在区域 `x:[50,480], y:[地块点击y+40, 地块点击y+80]` 识别最左数字块
- 活动作物跳过：`SEED_BTN_HEART_FRUIT`（爱心果）固定排除；`config.planting.skip_event_crops` 默认关闭，仅控制是否额外排除 `SEED_BTN_MUGWORT`（艾草）
- 等级同步：播种前执行等级 OCR；由 `config.planting.level_ocr_enabled` 控制，识别后回写 `config.planting.player_level`；统一 ROI 使用 `tasks/main.py` 内常量（不区分平台）
- 窗口选择：`config.window_select_rule` 仅保存匹配顺序（`auto` / `index:N`），不保存 `hwnd`
- 视觉按钮来源：`core/ui/assets.py`（由 `tools/button_extract.py` 生成）
- 版本来源：`utils/version.py::APP_VERSION`（Release 打包前由 `tools/write_version.py --tag <tag>` 自动写入）
- 更新检查：读取 GitHub `releases/latest`；启动后自动检查一次，之后每 `6` 小时检查一次；有更新时左侧“设置”图标显示红点

## 1. 核心架构与职责

- `core/engine/bot/engine.py`
: `BotEngine` 入口，组合 `bootstrap/executor/runtime/vision`。

- `core/instance/manager.py`
: 实例会话管理（实例增删改查、当前实例切换、元数据保存）。

- `core/engine/bot/runtime.py`
: 生命周期与会话控制（start/stop/pause/resume/run_once）、配置更新、可中断睡眠、坐标映射。

- `core/engine/bot/executor.py`
: 任务注册与调度桥接（自动发现 `_run_task_*`）。

- `core/engine/task/executor.py`
: 通用任务执行器（pending/waiting 队列、优先级排序、结果回写 next_run）。

- `tasks/*.py`
: 业务任务实现（`main/friend/share/reward/gift/sell/land_scan` 及子任务）。

- `core/ui/ui.py` + `core/base/module_base.py`
: 页面识别、导航、弹窗清理、`appear/appear_then_click` 等模板点击能力。

## 2. 调度语义（必须遵守）

### 2.1 任务来源

- 执行器自动发现 `BotExecutorMixin` 中所有 `_run_task_<name>`。
- 任务启停与参数从 `config.tasks[<name>]` 读取。

### 2.2 排序规则

- 仅执行 `enabled=true` 且 `next_run <= now` 的任务。
- `pending` 队列按 `priority` 升序排序（值越小优先级越高）。
- 同一时刻到期任务按 `priority` 串行执行，不并发。

### 2.3 触发类型

- `trigger=interval`：按 `interval_seconds`。
- `trigger=daily`：按 `daily_time` 计算距离下一次秒数。
- `trigger=interval` 额外受 `enabled_time_range`（`HH:MM:SS-HH:MM:SS`）限制；不在时间段则跳过本轮并延迟到下个时间段起点。
- `interval_seconds` / `failure_interval_seconds` 生效下限为 `executor.min_task_interval_seconds`（默认 `5` 秒）。
- `TaskResult.next_run_seconds` 若设置，会覆盖本次默认成功/失败间隔。
- 执行器每次计算后的 `next_run` 会回写 `config.tasks.<name>.next_run`。

### 2.4 失败语义

- `TaskResult.success=false` 时计入失败并使用 `failure_interval_seconds`（除非 next_run_seconds 覆盖）。
- 不要新增会影响调度推进的“业务阻断标记”。

### 2.5 多实例边界（必须遵守）

- 启动/暂停/停止/立即执行逻辑保持在中间实例面板，禁止新增“全局实例总控启停”。
- 实例纳管仅包含：`新增 / 删除 / 切换 / 克隆 / 重命名`。
- 删除、重命名运行中实例必须拒绝并提示先停止该实例。

## 3. 常用方法速查

## 3.1 Runtime/Bot 常用

- `_is_cancel_requested(session_id=None) -> bool`
: 判断当前会话或执行器是否停止。

- `_sleep_interruptible(seconds, session_id=None) -> bool`
: 可中断睡眠，返回 `False` 表示被取消。

- `_prepare_window() -> rect | None`（vision）
: 刷新窗口、激活、更新 `action_executor/device` 窗口矩形。

- `_clear_screen(rect, session_id=None)`
: 连续点击 `GOTO_MAIN` 兜底回主。

- `resolve_live_click_point(x, y) -> (x, y)`
: 逻辑坐标映射到当前截图坐标系（考虑 nonclient 裁剪偏移）。

- `device.click_minitouch(x, y, desc=...) -> bool`
: 统一通过 `ActionExecutor` 执行点击动作。

- `_capture_frame(rect, save=False) -> (cv_img, pil_img)`
: 截图并推送 GUI 预览。

- `_capture_and_detect(...)`
: 当前只负责截图返回，模板检测由业务侧按需调用 detector。

## 3.2 UI/模板点击常用

- `ui.ui_get_current_page(...)`
: 页面识别（未知页会尝试回主+清弹窗）。

- `ui.ui_goto(page)` / `ui.ui_ensure(page)`
: 页面导航与确保到达。

- `ui.ui_additional()`
: 统一弹窗处理入口（等级、奖励、公告等）。

- `appear(button, offset=(30,30), threshold=0.8, static=False)`
: 仅判断出现。

- `appear_then_click(..., interval=1, ...)`
: 出现后点击；**interval 最低保持 1**。

- `appear_then_click_any([...], interval=1, ...)`
: 依次尝试多个按钮。

- UI 页面跳转开发注意
: 在跳转链路里，点击只代表“发起动作”，不代表“页面已切换”；点击后应继续循环并复检页面状态，确认到达目标页后再结束当前分支。

## 3.3 TaskExecutor 常用

- `task_call(task_name, force_call=True)`
: 立即将任务置为可执行。

- `task_delay(task_name, seconds=..., target_time=...)`
: 推迟任务下一次执行。

- `update_task(name, **kwargs)`
: 热更新任务参数（enabled/priority/interval 等）。

## 4. 业务任务逻辑（当前实现）

- `main`
: 主流程任务，按功能开关顺序执行收获维护、扩建、播种、升级等动作。

- `main` 在自动播种前会先尝试等级 OCR（可由 `config.planting.level_ocr_enabled` 关闭）。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [megumiss/qq-farm-copilot](https://github.com/megumiss/qq-farm-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
