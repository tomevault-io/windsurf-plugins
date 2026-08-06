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
: `core/ui/assets.py` 由 `tools/button_extract.py` 自动生成，不要对其执行 `ruff format`。
: 推荐命令：`.\.venv\Scripts\python.exe -m ruff format core gui models tasks utils main.py`

- `gui/main_window_core.py` 是 Fluent GUI 主窗口源码，也是运行与发布打包的默认加载模块；不再维护 `gui/main_window_core.pyd` 二进制产物。


## 0. 当前状态

- 项目名：`QQ Farm Copilot`
- UI 结构：左侧截图预览 + 中间实例运行面板 + 最右侧竖向实例栏
- 实例纳管操作：`新增 / 删除 / 切换 / 克隆 / 重命名`
- 调度模式：`TaskExecutor` 单线程串行执行
- 任务配置：`%APPDATA%/QQFarmCopilot/instances/<instance_id>/configs/config.json -> tasks`（动态字典，包含持久化 `next_run`）
- 配置清理：启动加载配置时会按 `configs/config.template.json` 自动清理 `tasks.<name>.features` 中模板未定义的历史字段（并写回配置）
- 农场详情配置：`config.land.plots`（固定 24 格，元素结构：`{plot_id, level, maturity_countdown, countdown_sync_time, need_upgrade, need_planting}`；`level` 支持 `unbuilt|normal|red|black|gold|amethyst`；`maturity_countdown` 为 `HH:MM:SS`，`countdown_sync_time` 为该地块倒计时采样时间 `YYYY-MM-DD HH:MM:SS`，`need_upgrade` 表示地块是否可升级，`need_planting` 表示地块是否需要播种）及 `config.land.profile`（`level/gold/coupon/exp`，来源于等级同步 OCR）
- 好友黑名单配置：`config.tasks.friend.features.blacklist`（`list[str]`，在任务设置详情弹窗维护）
- 数据统计开关：`config.tasks.friend.features.steal_stats`（默认 `false`；开启后仅在偷取动作后执行 OCR 统计，偷取速度会变慢）
- 好友偷菜限制：`config.tasks.friend.features.steal_enabled_time_range`（默认 `00:00:00-23:59:59`）与 `config.tasks.friend.features.steal_limit_count`（默认 `0`，表示不限）
- 好友帮忙限制：`config.tasks.friend.features.help_enabled_time_range`（默认 `00:00:00-23:59:59`）与 `config.tasks.friend.features.help_limit_count`（默认 `0`，表示不限）
- 护主犬帮忙过滤：`config.tasks.friend.features.help_only_guard_dog`（默认 `false`；开启后仅在好友详情匹配到 `icon_xxxx.gif` 时执行帮忙）
- 数据统计落盘：`%APPDATA%/QQFarmCopilot/instances/<instance_id>/stats/daily_action_stats.csv`（按天累计 `harvest/operation/friend_steal/friend_help`）
- 定时重启任务：`config.tasks.restart`（默认关闭；`trigger=interval`，默认 `interval_seconds=14400`；重启等待时间使用实例级 `config.window_restart_delay_seconds`，默认 `5` 秒）
- 活动任务：`config.tasks.event`（默认关闭；`trigger=daily`，默认 `daily_times=["10:01"]`；`features.activity_name/resources/use_coupon/end_time` 记录当期活动配置与结束时间）
- 定时收获任务：`config.tasks.timed_harvest`（默认开启；`trigger=daily`，默认 `daily_times=["00:00"]`；`features.aggregation_seconds` 默认 `60` 秒；`features.priority_window_seconds` 默认 `120` 秒；依赖地块巡查结果生成后续执行点）
- 高级配置：`config.safety.debug_log_enabled` 控制 Debug 日志输出；`config.safety.stuck_seconds`（默认 `60` 秒）与 `config.safety.stuck_long_wait_seconds`（默认 `120` 秒）控制无有效点击的卡死判定阈值
- 异常恢复配置：`config.recovery`（`task_restart_attempts/task_retry_delay_seconds/window_launch_wait_timeout_seconds/startup_retry_step_sleep_seconds/startup_stabilize_timeout_seconds/prefer_repair_before_restart`）；`prefer_repair_before_restart` 为 `true` 时，页面未知/卡死/截图/窗口异常会先尝试一键修复（最多 3 次），失败后 fall back 到重启任务
- 通知配置：`config.notification`（`exception_notify_enabled/win_toast_enabled/onepush_config`；仅在触发人工接管停机时发送异常通知）
- 全局日志保留：`%APPDATA%/QQFarmCopilot/app_settings.json -> logging.retention_days`（单位天，默认 `7`；启动与全局设置变更时清理过期 `.log`）
- 截图频率：`config.screenshot.capture_interval_seconds`（默认 `0.3` 秒；`0` 表示不限制最小截图间隔）
- 播种稳定超时：`config.planting.planting_stable_timeout_seconds`（默认 `3.0` 秒；用于背景树锚点稳定等待超时）
- 土地滑动次数：`config.planting.land_swipe_right_times`（默认 `4`）与 `config.planting.land_swipe_left_times`（默认 `6`）；地块巡查与土地升级共用，滑动坐标仍使用代码内静态坐标
- 播种选种：`config.planting.warehouse_first` 默认开启；开启时优先按 `BgPatchNumberOCR` 在区域 `x:[50,480], y:[地块点击y+35, 地块点击y+115]` 识别最左数字块
- 活动作物跳过：`SEED_BTN_HEART_FRUIT`（爱心果）、`SEED_BTN_HAHA_PUMPKIN`（哈哈南瓜）、`SEED_BTN_COLORED_GLAZE_LOTUS`（琉璃莲）与 `SEED_BTN_STARWHISPER_FLOWER`（星语花）固定排除；`config.planting.skip_event_crops` 默认关闭，若与 `warehouse_first` 同时开启则按关闭仓库优先处理
- 等级同步：播种前执行等级 OCR；由 `config.planting.level_ocr_enabled` 控制，识别后回写 `config.planting.player_level`；统一 ROI 使用 `tasks/main.py` 内常量（不区分平台）
- 小程序快捷方式：`config.window_shortcut_path` 保存桌面快捷方式路径（`.lnk`，在设置面板“窗口关键词”上方选择）；`config.window_shortcut_launch_delay_seconds`（默认 `3` 秒）控制快捷方式拉起后到窗口初始化之间的等待时间
- 窗口选择：`config.window_select_rule` 仅保存匹配顺序（`auto` / `index:N`），不保存 `hwnd`
- 窗口定位：`config.planting.window_screen_index` 保存目标屏幕序号（与显示器查询序号一致；`0` 表示默认主屏）；`config.planting.window_position` 保存该屏幕内的位置锚点（左中/居中/右中/四角）；`config.planting.virtual_desktop_index` 保存目标虚拟桌面序号（`0` 表示不移动，`1+` 表示目标桌面）
- 视觉按钮来源：`core/ui/assets.py`（由 `tools/button_extract.py` 生成）
- 版本来源：`utils/version.py::APP_VERSION`（Release 打包前由 `tools/write_version.py --tag <tag>` 自动写入）
- 更新检查：读取 GitHub `releases/latest`；启动后自动检查一次，之后每 `6` 小时检查一次；有更新时左侧“设置”图标显示红点

## 1. 核心架构与职责

- `core/engine/bot/engine.py`
: `BotEngine` 入口，组合 `bootstrap/executor/runtime/vision`。

- `core/instance/manager.py`
: 实例会话管理（实例增删改查、当前实例切换、元数据保存）。

- `core/engine/bot/runtime.py`
: 生命周期与会话控制（start/stop/pause/resume/run_once）、配置更新、可中断睡眠、坐标映射；并负责启动阶段异常收敛与恢复。

- `core/engine/bot/executor.py`
: 任务注册与调度桥接（自动发现 `_run_task_*`），并作为任务异常恢复主入口（NIKKE 风格单层 `try/except` 直分支）。

- `core/engine/task/executor.py`
: 通用任务执行器（pending/waiting 队列、按固定任务顺序调度、结果回写 next_run）。

- `tasks/*.py`
: 业务任务实现（`main/friend/share/reward/gift/event/sell/land_scan/timed_harvest` 及子任务；`restart` 入口在 `executor.py`）。

- `core/ui/ui.py` + `core/base/module_base.py`
: 页面识别、导航、弹窗清理、`appear/appear_then_click` 等模板点击能力。

## 2. 调度语义（必须遵守）

### 2.1 任务来源


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rainnight6/qq-farm-copilot](https://github.com/rainnight6/qq-farm-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
