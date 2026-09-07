---
trigger: always_on
description: > 本文件是 AI 助手理解本项目的入口指引。
---

# CLAUDE.md - CoC-kb 项目总览

> 本文件是 AI 助手理解本项目的入口指引。
> 最后更新: 2026-05-16

---

## 项目简介

**CoC-kb** 是一个围绕《克苏鲁的呼唤（Call of Cthulhu）第七版》TRPG 的综合项目，包含四大部分：

1. **知识库** — 从官方规则书系统提取的 CoC 7e 规则 wiki（298+ 页面）
2. **角色卡创建器** — 8 步引导式调查员角色创建工具
3. **角色卡追踪器** — 模拟纸质角色卡的调查员数据追踪工具
4. **向火独行** — 单人模组《向火独行》电子陪跑原型

---

## 目录结构

```
CoC-kb/
├── CLAUDE.md                          ← 本文件（项目总览）
├── README.md                          ← 项目介绍（面向人类读者）
├── knowledge-base/                    ← CoC 7e 规则知识库
│   ├── CLAUDE.md                      ← 知识库维护规范（必读）
│   ├── wiki/
│   │   ├── index.md                   ← 内容索引（查找页面的入口）
│   │   ├── log.md                     ← 操作日志（仅追加）
│   │   ├── concepts/                  ← 概念页面（69个：规则、机制、系统）
│   │   ├── entities/                  ← 实体页面（223个：生物、神祇、模组）
│   │   ├── synthesis/                 ← 合成分析（维护追踪、补全计划等）
│   │   ├── images/                    ← 实体配图（~190张，AI 生成）
│   │   └── sources/                   ← 来源摘要（6个规则书）
│   └── raw/                           ← 原始 PDF（只读，不可修改）
├── apps/
│   ├── coc_character_sheet/           ← 角色卡创建器（多文件 SPA）
│   ├── character-tracker/             ← 角色卡追踪器（多文件架构）
│   └── alone-against-the-flames-app/  ← 向火独行电子陪跑（原型阶段）
├── docs/                              ← 设计文档
└── .obsidian/                         ← Obsidian 配置
```

---

## 模块详解

### 1. knowledge-base/ — 规则知识库

- **主题**: 克苏鲁的呼唤第七版 TRPG 规则体系
- **规模**: 69 个概念页面 + 223 个实体页面 + 6 个来源摘要
- **来源书籍**: 40周年纪念版(470页)、调查员手册(162页)、怪物之锤两卷(468页)、入门套件第一卷与第三卷
- **维护规范**: 详见 `knowledge-base/CLAUDE.md`
- **架构模式**: 基于 Karpathy 的 LLM Wiki 模式，分为三层：
  1. `raw/` — 原始 PDF 规则书（只读，不可修改，不纳入版本控制）
  2. `wiki/` — 知识整理层（LLM 维护，Markdown 页面 + 交叉引用）
  3. `CLAUDE.md` — LLM 维护规范（Schema 配置 + 操作流程）
- **关键约定**:
  - 所有规则数据必须忠实于原始规则书，不可编造
  - `raw/` 目录只读，所有知识整理通过 wiki 层进行
  - 使用 `[[页面路径]]` 格式维护交叉引用
  - 操作日志 `log.md` 仅追加

### 2. apps/coc_character_sheet/ — 角色卡创建器

- **定位**: 8 步引导式调查员角色创建工具
- **技术栈**: 纯 HTML/CSS/JS，无框架，localStorage 持久化
- **支持时代**: 1920年代 / 现代
- **8 步流程**:
  1. 基本信息（姓名/性别/年龄/时代/头像）
  2. 属性生成（3D 骰子掷 8 项核心属性 + 幸运）
  3. 年龄调整（自动计算 EDU 成长/属性减值）
  4. 职业选择（数十种预定义职业 + 自定义职业）
  5. 技能分配（职业技能点 + 兴趣技能点）
  6. 背景故事（随机灵感表生成 + 手动编辑）
  7. 装备决定（按信用评级搜索物品/武器数据库）
  8. 完成导出（可编辑角色卡 + `.coc7` 文件导出）
- **文件结构**:
  ```
  js/
    data/          ← 游戏数据（skills/occupations/equipment/weapons/tables）
    render/        ← 渲染层（router + steps-1to4 + steps-5to8 + progress）
    state.js       ← 全局状态 + localStorage
    dice-physics.js ← 自研 3D 骰子物理引擎
    utils.js       ← 工具函数（骰子/属性计算/技能基础值）
    navigation.js  ← 导航逻辑 + 导出
    init.js        ← 启动入口
  css/             ← 样式（base.css + components.css）
  assets/avatars/  ← 20 张职业主题头像
  ```

### 3. apps/character-tracker/ — 角色卡追踪器

- **定位**: 模拟纸质角色卡正反面的调查员数据展示与追踪工具
- **技术栈**: 多文件 HTML/CSS/JS（无框架），dice-box 3D 骰子库（ES Module）
- **架构**: 模块化多文件架构，按功能拆分为 render/（渲染层）、data/（数据层）、dice-module.js（骰子模块）等
- **核心功能**:
  - 调查员基本信息 + 头像
  - 8 项属性（含半值/五分之一值）+ 骰子检定按钮
  - 资源追踪条（HP/Luck/MP/SAN + 状态勾选）
  - 50+ 技能表（三栏布局，含判定值 + 骰子按钮）
  - 武器表格 + 战斗摘要（DB/体格/闪避）
  - 背景故事（9 个字段）
  - 随身物品 + 资产面板
  - 快速参考规则（检定表 + 治疗规则）
  - 调查员同伴环形图（8 节点）
  - 3D 骰子检定系统（D4/D6/D8/D10/D12/D20/D100，支持骰子表达式）
- **数据存储**: localStorage 持久化，支持 .coc7 文件导入
- **测试**: 已移除（原有 Playwright E2E 测试和 Python 备选测试脚本已删除）
- **第三方依赖**: 集成 [dice-box](https://github.com/3dice/Dice-Box) 3D 骰子库（含 WebAssembly），支持 D4/D6/D8/D10/D12/D20/D100 骰子，附带 10 种视觉主题（含 CoC 专属主题 `coc/`）
- **文件结构**:
  ```
  index.html              ← HTML 结构 + 外部文件引用（~253 行）
  css/
    tracker.css           ← 全部样式（~2141 行）
  js/
    data/
      skills.js           ← 技能基础值数据 + 常量（SKILLS_DATA, SPECIALTY_MAP, ATTR_KEYS 等）
      default-character.js ← 默认示例角色数据（DEFAULT_DATA）
    utils.js              ← 工具函数（getSkillBase, calcSkillTotal, half, fifth）
    import.js             ← .coc7 导入/清除功能（decompressState, importCharacter）
    render/
      info.js             ← renderBasicInfo（基本信息 + 头像）
      attributes.js       ← renderAttributes（属性网格 + MOV）
      trackers.js         ← renderTrackers（HP/Luck/MP/SAN 资源条）
      skills.js           ← renderSkills（技能列表）
      weapons.js          ← renderWeapons（武器表格 + 战斗面板）
      backstory.js        ← renderBackstory（背景故事网格）
      equipment.js        ← renderEquipment（随身物品 + 资产）
      companions.js       ← renderCompanions（调查员同伴环形图）
    app.js                ← 主入口 IIFE（init + renderCharacter + bindDiceButtons）
    dice-module.js        ← 3D 骰子模块（ES Module）
  assets/                 ← 背景图 + 4 个角落装饰 SVG
  dice-box/               ← 3D 骰子引擎库 + 10 种主题资源
  ```

### 4. apps/alone-against-the-flames-app/ — 向火独行电子陪跑

- **定位**: 单人模组《向火独行（Alone Against the Flames）》的电子陪跑工具
- **技术栈**: 多文件 HTML/CSS/JS（ES Module），复用 character-tracker 的 DiceBox 3D 骰子库
- **当前状态**: 核心引擎完成，可完整游玩 270 条目模组；节点图导航已实装
- **架构**: engine/adapters/data/ui 四层代码边界
- **核心功能**（已实现）:
  - 三栏体验布局（左侧快速入口+进度、中央剧情主舞台、右侧角色状态+线索）
  - 模组引擎（module-engine.js）驱动节点式剧情流转
  - .coc7 角色卡导入（复用 character-adapter.js）
  - DiceBox 3D 骰子面板（dice-adapter.js），支持百分骰/奖励骰/惩罚骰
  - 技能/属性/派生值检定系统（自动识别检定难度和模式）
  - 检定结果驱动路径分支（成功/失败/大失败门控）
  - 对抗检定 & 多回合战斗系统（6 个战斗场景，弹出层 UI）
  - 效果系统：HP/SAN/MP/Luck 调整、技能成长、物品获得/失去、战斗触发
  - 检定门控效果：属性损失仅在检定失败后触发
  - 伤害阈值自动分支（伤害 vs maxHP/2 决定路径）
  - 效果通知 UI（effect pills）
  - 结局复盘页面（18 个结局，含路径/状态/里程碑/技能成长）
  - localStorage 游戏进度持久化
  - 路径时间线、线索线程、剧情记录面板
  - **节点图导航**（graph-map.js）：Canvas 力导向图，可视化 270 条目的跳转关系，支持平移/缩放/悬停预览，已访问节点高亮
- **待实现**:
  - 孤注一掷（Pushed Roll）
  - 幸运消耗（Spending Luck）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [digitalghost/CoC-kb](https://github.com/digitalghost/CoC-kb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
