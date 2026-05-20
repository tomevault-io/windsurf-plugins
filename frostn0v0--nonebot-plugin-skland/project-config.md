---
trigger: always_on
description: 本文档为 AI 编程代理（如 GitHub Copilot、Claude 等）提供项目上下文和开发指南。
---

# AGENTS.md

本文档为 AI 编程代理（如 GitHub Copilot、Claude 等）提供项目上下文和开发指南。

## 项目概述

**nonebot-plugin-skland** 是一个基于 NoneBot2 框架的 Python 插件，用于通过森空岛（Skland）API 查询鹰角网络旗下游戏的数据，目前支持：

- **明日方舟（Arknights）**：角色信息卡片、签到、肉鸽战绩、抽卡记录
- **终末地（Endfield）**：角色信息卡片、签到、抽卡记录

## 技术栈

- **Python**: >=3.10
- **框架**: NoneBot2 + nonebot-plugin-alconna（命令解析）
- **数据库**: SQLAlchemy ORM（通过 nonebot-plugin-orm）
- **渲染**: nonebot-plugin-htmlrender（Jinja2 模板 + Tailwind CSS）
- **定时任务**: nonebot-plugin-apscheduler
- **HTTP 客户端**: httpx

## 目录结构

```text
nonebot_plugin_skland/
├── __init__.py          # 插件入口，命令处理器注册
├── matcher.py           # Alconna 命令定义和快捷指令
├── tasks.py             # 定时任务（每日签到）
├── config.py            # 配置管理（含 gacha_render_max、ef_gacha_render_max 等）
├── data_source.py       # 游戏数据加载与管理（卡池数据）
├── model.py             # SQLAlchemy ORM 模型（SkUser, Character, GachaRecord）
├── db_handler.py        # 数据库操作函数
├── api/
│   ├── __init__.py      # API 模块导出
│   ├── request.py       # SklandAPI - 森空岛数据接口（签到、角色信息、抽卡等）
│   └── login.py         # SklandLoginAPI - 登录认证接口
├── commands/            # 命令处理器模块
│   ├── __init__.py      # 模块导出
│   ├── card.py          # 明日方舟角色卡片查询处理
│   ├── bind.py          # 绑定和二维码登录
│   ├── char.py          # 角色信息更新
│   ├── sync.py          # 资源同步
│   ├── rogue.py         # 肉鸽战绩查询
│   ├── gacha.py         # 明日方舟抽卡记录查询和导入
│   ├── arksign.py       # 明日方舟签到
│   └── endfield/        # 终末地命令处理子包
│       ├── __init__.py  # 导出所有 handler
│       ├── card.py      # 终末地角色卡片查询
│       ├── sign.py      # 终末地签到
│       ├── gacha.py     # 终末地抽卡记录查询（支持 -u 更新、分页渲染）
│       └── utils.py     # 终末地工具函数（签到结果格式化等）
├── schemas/             # Pydantic 数据模型
│   ├── __init__.py      # 模块导出（向后兼容）
│   ├── binding.py       # 绑定角色数据
│   ├── cred.py          # 认证凭据
│   ├── arknights/       # 明日方舟数据模型
│   │   ├── __init__.py  # 导出所有模型
│   │   ├── card.py      # 角色卡片（ArkCard）
│   │   ├── sign.py      # 签到响应（ArkSignResponse）
│   │   ├── game_data.py # 游戏数据（CharTable, GachaDetails）
│   │   ├── gacha/       # 抽卡相关模型
│   │   │   ├── base.py      # 基础类型（GachaInfo, GachaPull 等）
│   │   │   ├── pool.py      # 卡池（GachaPool）
│   │   │   └── statistics.py # 统计（GroupedGachaRecord）
│   │   ├── rogue/       # 肉鸽相关模型
│   │   │   ├── data.py      # 基础数据（Topics, RogueData）
│   │   │   ├── career.py    # 生涯统计（RogueCareer）
│   │   │   └── history.py   # 历史记录（RogueHistory）
│   │   └── models/      # 详细数据模型
│   │       ├── status.py    # 状态信息
│   │       ├── chars.py     # 角色信息
│   │       ├── building.py  # 基建信息
│   │       └── ...          # 其他模型
│   └── endfield/        # 终末地数据模型
│       ├── __init__.py  # 导出所有模型
│       ├── card.py      # 角色卡片（EndfieldCard）
│       ├── sign.py      # 签到响应（EndfieldSignResponse）
│       └── gacha/       # 终末地抽卡相关模型
│           ├── __init__.py
│           ├── base.py      # 基础类型（EndfieldPoolType, EfCharGachaInfo, EfWeaponGachaInfo, EfGachaContentPool 等）
│           ├── pool.py      # 卡池信息（EfGachaPoolInfo，含保底与武库配额计算）
│           └── statistics.py # 分组统计（EfGroupedGachaRecord，含 flat_pools/max_category_pool_count/get_visible_pool_ids）
├── render.py            # HTML 渲染函数
├── filters.py           # Jinja2 模板过滤器
├── utils.py             # 工具函数（Token刷新装饰器、资源下载、抽卡记录分组等）
├── download.py          # 游戏资源下载器
├── exception.py         # 自定义异常类
├── hook.py              # 启动/关闭钩子
├── extras.py            # 帮助菜单数据
├── migrations/          # 数据库迁移脚本
└── resources/
    ├── fonts/           # 字体文件
    ├── images/          # 图片资源
    │   ├── endfield/    # 终末地图片资源（职业图标、属性图标、进阶阶段等）
    │   └── gacha/       # 抽卡记录装饰图片
    └── templates/       # Jinja2 HTML 模板 + CSS
        ├── ark_card.html.jinja2
        ├── endfield_card.html.jinja2
        ├── gacha.html.jinja2          # 明日方舟抽卡记录模板
        ├── gacha_macros.html.jinja2
        ├── ef_gacha.html.jinja2       # 终末地抽卡记录模板
        ├── ef_gacha_macros.html.jinja2
        ├── rogue.html.jinja2
        ├── rogue_info.html.jinja2
        ├── rogue_macros.html.jinja2
        ├── endfield_macros.html.jinja2
        ├── macros.html.jinja2
        └── clue.html.jinja2           # 线索看板模板
```

## 核心模块说明

### 命令系统

命令定义和处理器分布在以下模块中：

- **`matcher.py`**: Alconna 命令定义和快捷指令

```python
skland_command = Alconna(
    "skland",
    Subcommand("bind", ...),      # 绑定账号
    Subcommand("qrcode", ...),    # 扫码绑定
    Subcommand("arksign", ...),   # 明日方舟签到
    Subcommand("efsign", ...),    # 终末地签到
    Subcommand("rogue", ...),     # 肉鸽战绩
    Subcommand("gacha", ...),     # 明日方舟抽卡记录
    Subcommand("efcard", ...),    # 终末地角色面板
    Subcommand("efgacha", ...),   # 终末地抽卡记录（-u 更新, -b/-l 分页）
    Subcommand("rginfo", ...),    # 单局肉鸽战绩详情
    # ...
)
skland = on_alconna(skland_command, ...)
```

- **`__init__.py`**: 命令处理器注册，使用 `@skland.assign("subcommand.action")` 装饰器
- **`commands/`**: 按功能划分的命令处理逻辑

### 游戏数据管理 (`data_source.py`)

管理游戏卡池数据的加载与更新：

- **`GachaTableData`** — 明日方舟卡池数据管理（版本检查、下载、加载），数据源为 [ArknightsGameResource](https://github.com/yuanyan3060/ArknightsGameResource) 和 PRTS Wiki
- **`EfGachaPoolTableData`** — 终末地卡池数据管理，数据源为 [EndfieldGachaPoolTable](https://github.com/FrostN0v0/EndfieldGachaPoolTable)

全局实例 `gacha_table_data` 和 `ef_gacha_pool_data` 在 `hook.py` 启动时加载。

### 定时任务 (`tasks.py`)

定时任务从 `__init__.py` 中提取，独立管理：

```python
@scheduler.scheduled_job("cron", hour=0, minute=15, id="daily_arksign")
async def run_daily_arksign():

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FrostN0v0/nonebot-plugin-skland](https://github.com/FrostN0v0/nonebot-plugin-skland) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
