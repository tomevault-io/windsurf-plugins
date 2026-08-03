---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

QQ Farm Vision Bot — 基于 OpenCV 视觉识别的 QQ 经典农场（QQ/微信小程序）自动化工具。纯本地运行，不依赖游戏接口。当前是 2.0.x 多实例版本，支持后台运行、Web 控制、老板键、地块巡查、跨实例偷菜/捣乱、自动选择账号和 GitHub Actions 发布。

**技术栈**: Python 3.10+, PyQt6, OpenCV, MSS, PyAutoGUI, Pydantic, loguru, FastAPI (可选 Web 面板)

## Commands

```bash
# 安装依赖
pip install -r requirements.txt

# 启动程序
python main.py

# 模板采集（首次使用必须）
python tools/template_collector.py

# 种子图片批量导入
python tools/import_seeds.py

# 构建 EXE
pyinstaller build.spec
```

无 pytest 测试套件。测试脚本为独立脚本，需运行中的游戏窗口：
- `test_template_categories.py` — 列出已加载模板
- `test_land_count.py` — 土地数量检测
- `test_empty_land_detection.py` — 空地检测
- `test_plant_capture.py` — 播种流程测试

**热键**: F9 暂停/恢复，F10 停止，F11 老板键（隐藏窗口）。鼠标移到左上角可紧急停止（pyautogui FAILSAFE）。

## Architecture

### 数据流

```
截屏 (mss) → OpenCV 多尺度模板匹配 → 场景识别状态机 → 策略决策 → ActionExecutor (前台 pyautogui / 后台 PostMessageW) → 循环
```

### 多实例架构

程序支持同时管理多个游戏窗口（多开）。每个实例拥有独立的配置、日志、截图目录。

- **InstanceManager** (`core/instance_manager.py`): 管理所有实例的元数据、配置和会话，存储在 `instances/profiles.json`
- **InstanceSession**: 封装单个实例的元数据（id/name/state）+ 路径 + 配置
- **实例目录结构**: `instances/{id}/configs/config.json`, `instances/{id}/logs/`, `instances/{id}/screenshots/`
- **MainWindow** 维护 `dict[str, BotEngine]` 映射（`_engines`），每个实例一个 BotEngine
- **InstanceSidebar** (`gui/widgets/instance_sidebar.py`): 实例列表 UI，支持右键菜单（新增/克隆/重命名/删除）
- **BotEngine** 构造函数接受 `instance_id` 和 `cross_bus` 参数，用于日志/截图目录隔离和跨实例通讯

### 四层架构

```
┌─────────────────────────────────────────────┐
│  GUI 层 (PyQt6) + Web 层 (FastAPI, 可选)    │
│  main_window.py / widgets/                   │
│  InstanceSidebar (多实例切换)                 │
│  web/server.py (截图预览、启停控制)          │
├─────────────────────────────────────────────┤
│  行为决策层 (core/strategies/)               │
│  popup → harvest → maintain → plant →        │
│  expand → task → friend → gift →             │
│  targeted_steal / targeted_prank             │
├─────────────────────────────────────────────┤
│  图像识别层                                  │
│  cv_detector.py (模板匹配)                   │
│  scene_detector.py (场景识别)                │
├─────────────────────────────────────────────┤
│  窗口控制层 + 操作执行层                     │
│  window_manager.py + screen_capture.py       │
│  action_executor.py (前台/后台, RunMode)     │
└─────────────────────────────────────────────┘
```

### 主控编排: BotEngine (core/bot_engine.py)

- **BotEngine** (QObject): 初始化各层组件、管理配置、连接 Qt 信号。每个实例一个 BotEngine
- **BotWorker** (QThread): 在独立线程执行 farm/friend/test_fertilize 任务
- **TaskScheduler** (`core/task_scheduler.py`, QObject + QTimer): 定时触发农场检查和好友巡查，含窗口存活监控
- **TaskExecutor** (`core/task_executor.py`): 基于优先级的异步任务调度器，支持 interval/daily 触发、时段过滤、失败重试、热更新。通过 `TaskScheduleItemConfig` 配置每个任务
- **CrossInstanceBus** (`core/cross_instance_bus.py`): 跨实例偷菜/捣乱消息总线，负责 StealAlert / PrankAlert 去重和派发
- **状态总览快照**: 状态总览只展示运行状态、任务队列和下次执行时间；操作次数统计与每日动作统计落盘已移除
- 策略按优先级注册在 `self._strategies` 列表中
- 主循环 `check_farm()` 最多 50 轮，3 轮空闲自动退出，每轮 sleep 0.3s
- 静默时段: `core/silent_hours.py` 支持跨午夜时段（如 22:00-06:00），静默期间不执行操作
- 快速模板过滤: `SCENE_TEMPLATES` / `LAND_TEMPLATES` / `MAINTAIN_TEMPLATES` 常量只加载场景判断所需的模板子集，避免加载全部 70+ 模板

### 策略模式 (core/strategies/)

所有策略继承 `BaseStrategy`，共享 `cv_detector`、`action_executor`、`_capture_fn`。策略在 `__init__.py` 中统一导出。

**BaseStrategy** 提供:
- `click(x, y, desc)` — 构造 Action 并通过 action_executor 执行
- `find_by_name()`, `find_by_prefix_first()`, `find_any()` — 检测结果查询
- `stopped` 属性 — 检查停止/暂停信号，所有操作前必须检查

| 优先级 | 策略文件 | 类名 | 职责 |
|--------|----------|------|------|
| P-1 | `popup.py` | PopupStrategy | 关闭弹窗/商店/商城/返回主界面 + 升级检测 |
| P0 | `harvest.py` | HarvestStrategy | 一键收获 + 自动出售 |
| P1 | `maintain.py` | MaintainStrategy | 除草/除虫/浇水 |
| P2 | `plant.py` | PlantStrategy | 播种（动态翻页、手动指定次级回退） + 购买种子（OCR/价格/仓库格复查） + 施肥 |
| P3 | `expand.py` | ExpandStrategy | 扩建土地 |
| P3.5 | `task.py` | TaskStrategy | 领取任务奖励 / 出售果实 |
| P4 | `friend.py` | FriendStrategy | 帮忙除草/浇水/除虫/偷菜 |
| P5 | `gift.py` | GiftStrategy | SVIP 礼包 / 商城免费领取 / 邮件领取 |
| — | `targeted_steal.py` | TargetedStealStrategy | 定点偷菜（大小号通讯触发） |
| — | `targeted_prank.py` | TargetedPrankStrategy | 定点捣乱（大小号通讯触发） |

### 页面导航 (core/ui/navigator.py)

**Navigator**: 基于 BFS 的页面导航器，通过 `ui_ensure(target_page)` 自动从当前页面跳转到目标页面。依赖 `core/ui/page.py` 中的 `Page` 定义和 `ALL_PAGES` 页面图。需要注入 `capture_fn`、`click_fn`、`stopped_fn` 三个回调。

### 地块巡查 (tasks/land_scan.py)

**LandScanTask**: 两阶段滑动扫描（左滑 + 右滑）+ 逐块点击 OCR 采集。利用 `utils/land_grid.py` 的 `LandCell` / `get_lands_from_land_anchor` 计算地块网格坐标，`utils/ocr_utils.py` 的 `OCRTool` 识别成熟倒计时文字，支持普通/红/黑/金/紫晶土地等级。扫描结果供大小号通讯判断即将成熟的地块。

重要边界：地块巡查必须保持只读，不应隐式触发一键务农、主农场巡查、repair 或 restart。升级图标通过 `icon_land_upgrade.gif` 局部 ROI 检测并回写 `need_upgrade`。

### 大小号通讯 (core/cross_instance_bus.py)

**CrossInstanceBus**: 全局单例内存消息总线（线程安全），所有 BotEngine 共享。支持成熟偷菜通知和好友捣乱通知。

偷菜数据流：
```
Instance A 地块巡查 → 检测成熟倒计时 < N 秒 → post_alert(StealAlert)
→ Instance B TaskExecutor.poll_alerts() → 动态创建 TargetedStealTask → 定点偷菜
```
- **StealAlert**: 包含 source_instance_id、friend_name、plot_ids、earliest_maturity_seconds
- 内置去重（4 分钟内同一配对不重复发送）
- 配置在 `CrossInstanceConfig` 中：send_alerts / accept_steal / partners

捣乱数据流：
```
Instance A 检测可捣乱好友 → post_prank_alert(PrankAlert)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LuckyTiger12138/QQ_Farm](https://github.com/LuckyTiger12138/QQ_Farm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
