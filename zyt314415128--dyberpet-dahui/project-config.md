---
trigger: always_on
description: 将家里养的猫变成"数字生命"——一只运行在 Windows 桌面上的模拟养成桌面宠物。
---

# 桌面宠物 — 数字猫打灰

## 项目概述
将家里养的猫变成"数字生命"——一只运行在 Windows 桌面上的模拟养成桌面宠物。

## 核心设计决策

### 产品形态
- **类型**：模拟养成型桌面宠物（非对话伴侣、非纯观赏）
- **互动**：全面养成——饥饿值、心情值、亲密度、喂食、玩耍、铲屎
- **行为**：脚本化随机行为（站立、行走等概率触发）
- **语言交互**：猫不说话，不接入对话/语音功能

### 视觉风格
- **风格**：Disney/皮克斯风格，偏真实感，有猫毛细节，非简洁卡通风
- **形式**：2D 精灵图动画（Sprite Sheet）
- **素材管线**：AI 视频生成 → 色度键抠图（绿幕去背）→ 等比缩放部署

### 技术栈
- **壳/框架**：DyberPet（Python + PySide6），作为桌面渲染和养成系统底座
- **数据存储**：JSON（`dyberpet/data/` 下的 `pet_data.json`、`settings.json`、`act_data.json`、`task_data.json`）
- **目标平台**：Windows（唯一）

### 开发方式
- AI 辅助开发（Claude Code 写代码，用户做决策和测试）
- 参照 DyberPet 成熟项目的架构，在其基础上改造

## 目录结构约定
```
桌面宠物/
├── CLAUDE.md                  # 本文件 — 项目规范
├── .gitignore
├── dyberpet/                  # DyberPet 壳代码（上游 v0.8.5）
│   ├── run_DyberPet.py        #   启动入口 + 信号总线
│   ├── DyberPet/              #   PySide6 主逻辑
│   │   ├── DyberPet.py        #     PetWidget 主窗口 + 鼠标交互
│   │   ├── modules.py         #     三线程 Worker（Animation/Interaction/Scheduler）
│   │   ├── settings.py        #     全局状态 + 配置持久化
│   │   ├── conf.py            #     PetConfig/ActData/PetData/ItemData/TaskData
│   │   ├── utils.py           #     工具函数 + SubPet_Manager
│   │   ├── Accessory.py       #     饰品/掉落物/子宠物系统
│   │   ├── Notification.py    #     通知（Toaster）+ 气泡（BubbleText）
│   │   ├── bubbleManager.py   #     气泡行为逻辑层
│   │   ├── Dashboard/         #     仪表盘（状态/背包/商店/任务）
│   │   └── DyberSettings/     #     设置面板（基本设置/存档/角色卡/物品MOD）
│   ├── res/role/打灰/          #   奶牛猫打灰（→ assets/configs/ 是配置源）
│   └── res/role/花椒/          #   白猫花椒（像素占位，仅 stand 动画，待正式素材替换）
├── assets/                    # 素材与配置（唯一编辑源）
│   └── configs/               #   pet_conf.json + act_conf.json
├── tools/                     # 素材处理工具
│   ├── frame_extractor.py     #   视频帧提取（视频→PNG 序列）
│   ├── deploy_sprites.py      #   精灵图部署（色度键抠图 + resize 到 640×360）
│   ├── copy_configs.py        #   配置文件批量部署
│   ├── adjust_floor.py        #   地面位置可视化调节器
│   ├── cut_pixel_sprites.py   #   像素精灵图表格切割为单独帧
│   ├── map_pixel_frames.py    #   像素帧映射到项目动画命名格式
│   ├── cat-size-preview.html  #   猫尺寸可视化预览
│   └── frame-layout.html      #   1920×1080 参考帧布局
├── data/                      # 运行时数据（gitignore）
└── docs/                      # 设计文档
    ├── design.md              #   完整设计规格
    ├── decisions/             #   架构决策记录
    └── superpowers/           #   AI 工作产物
        ├── specs/             #     设计规格
        └── plans/             #     实现计划
```

## 命名约定
- Python 文件：snake_case
- 类名：PascalCase
- 配置 JSON：camelCase（与 DyberPet 保持一致）
- 精灵图文件：snake_case，动作名_帧数.png（如 walk_8f.png）
- **配置同步**：`assets/configs/` 是角色配置的**唯一编辑源**。修改后运行 `python tools/copy_configs.py` 部署到 `dyberpet/res/role/打灰/`。不要直接改 dyberpet 下的配置

## 开发原则
- 先在 DyberPet 原版跑通，再开始改
- 每次改完必须验证：猫能显示、能互动、数据能存
- 素材替换和代码改造分开做，不要混在一起提交
- **deploy_sprites.py 不覆盖已有 act_conf.json 配置**：只补充新动作 key，已有的 `frame_start`/`frame_end`/`move_phases`/`frame_refresh` 等设置必须保留不动

## 架构概览

### 三线程 + 信号驱动

应用采用三个 `QThread` Worker 并行运行，通过 Qt 信号通信：

| Worker | 职责 | 关键信号 |
|--------|------|---------|
| `Animation_worker` | 随机待机动画、HP/FV 概率系统、半屏方向过滤 | `sig_setimg_anim`, `sig_move_anim`, `sig_start_sleep` |
| `Interaction_worker` | 交互动画（拖拽/落地/睡觉/启动/喂食/摸头） | `sig_setimg_inter`, `sig_bounce_move`, `sig_act_finished` |
| `Scheduler_worker` | HP/FV 衰减、番茄钟/专注计时、物品掉落、自然唤醒 | `sig_change_hp`, `sig_change_fv`, `sig_wake_sche` |

**状态协调**：`PetWidget` 持有三个 Worker 引用。交互发生时暂停 Animation_worker，Interaction_worker 完成后调用 `resume_animation()` 恢复。

### 启动流程
`run_DyberPet.py` → `DyberPetApp` → `PetWidget`（`_init_ui` → `_init_widget` → `init_conf` → `runAnimation` → `runInteraction` → `runScheduler`）→ `DPNote` → `DPAccessory` → `__connectSignalToSlot`

### 数据持久化层（conf.py）

| 类 | 存储文件 | 职责 |
|----|---------|------|
| `PetData` | `data/pet_data.json` | HP、FV、coins、items、陪伴天数 |
| `ActData` | `data/act_data.json` | 动画解锁状态、概率、播放列表 |
| `TaskData` | `data/task_data.json` | 每日任务历史、目标、连续天数 |
| `settings` | `data/settings.json` | 用户偏好（重力/音量/缩放/语言等） |
| `ItemData` | 运行时加载 | 物品配置（扫描 `res/items/` + `res/pet/` + `res/role/{name}/items/`） |
| `PetConfig` | 运行时加载 | 角色配置（从 `pet_conf.json` + `act_conf.json` 构建） |

**懒加载**：`LazyAct` 类延迟加载精灵图，启动时只加载 stand 第一帧 + `fallasleep_wake` 全量，其余动作在后台线程 100ms 后逐个加载。

### 仪表盘系统（Dashboard）
`DashboardMainWindow(FluentWindow)`：状态（HP/FV/Buff）、背包（消耗品/收藏品）、商店（购买/出售，折旧率 75%）、任务（番茄钟/专注计时）。背包是中心枢纽。

### 物品系统
`res/items/Default/` 包含 `items_config.json`（21 个物品）。按 fv_lock 分 Tier 0-5，高层食物更强力。Buff 类型：`HP_stop`（冻结饥饿衰减）、`coin`（周期金币）。掉落概率：基础 8%，每级 FV +2%，上限 25%。

### 喂食动画系统
每个食物有专属喂食动画（`feed_{物品名}`）.
- **动画播放规则**：喂食动画播放期间，继续喂食不打断当前动画，也不补播；只实时产生效果（HP/FV 增加）
- **无专属动画的食物**：播放 stand 作为 fallback
- **素材管线**：绿幕视频 → frame_extractor.py 提帧 → sam2_mask.py 抠图 → deploy_sprites.py 部署
- **配置**：`act_conf.json` 中添加 `feed_{物品名}` 配置，`deploy_sprites.py` 的 `SLEEPActions` 列表中注册

### 金币系统
获取：点击猫（高斯分布）、物品掉落、任务奖励（单任务 200，五任务 1500）。消耗：商店购买。出售：折旧 75%。

### HP/FV 衰减与降级
- **HP 衰减**：每分钟 -1。Tiers：`[0, 20, 70, 100]` → Starving/Hungry/Normal/Energetic
- **FV 衰减**：HP tier 0 时每分钟 -5，其余不衰减
- **FV 降级**：FV 降到 0 后继续扣减，触发 `_level_down` 降至上一级满值（如 lv4→lv3 满值 120）
- **FV 升级**：每分钟 +1，满值后自动升级

### 通知系统
`DyberToaster`：右下角弹窗，合并同类，5 秒消失。`BubbleText`：猫头顶气泡，countdown 类型鼠标靠近淡出。`BubbleManager` 按 HP tier 调度气泡（tier 0 → `fv_drop`/`hp_zero`/`feed_required`；tier 1-2 → `hp_low`/`feed_required`）。隐身模式下所有气泡静默。

## 鼠标交互规则

### 状态机概览
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zyt314415128/dyberpet-dahui](https://github.com/zyt314415128/dyberpet-dahui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->
