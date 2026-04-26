---
trigger: always_on
description: 本文件用于指导自动化编码代理在本仓库内的工作方式与约定。
---

# Codex Agents Guide

本文件用于指导自动化编码代理在本仓库内的工作方式与约定。

## 项目概览
- 项目名称：哈耶克的文明：市场经济
- 技术栈：React 19 + Vite + Tailwind CSS
- 主要入口：`src/main.jsx`, `src/App.jsx`
- 游戏与设计文档：`/ai_reports`

## 常用命令
```bash
npm install
npm run dev
npm run build
npm run lint
npm run preview
```

## 目录结构与职责
- `src/components`: UI 组件（含 `common`, `layout`, `game`, `panels`, `modals`, `tabs`），统一导出在 `src/components/index.js`
- `src/logic`: 核心玩法与规则逻辑（`economy`, `population`, `stability`, `buildings`, `diplomacy` 等），统一导出在 `src/logic/index.js`
- `src/utils`: 通用工具与计算（数值计算、格式化、资源处理等）
- `src/hooks`: 自定义 Hooks
- `src/assets`: 静态资源
- `src/workers`: Web Worker 相关代码
- `src/config`: 配置与常量

## 代码约定
- 优先遵循现有结构：玩法逻辑放在 `src/logic`，通用计算放在 `src/utils`。
- UI 以组件化拆分，尽量保持组件单一职责。
- 使用 Tailwind CSS class 作为主要样式组织方式；除非必要，避免使用 `style` 内联样式。
- JS/JSX 使用 4 空格缩进与分号结尾；导出以 `export const`/`export function` 为主。
- 注释多为中文，保留现有叙述风格；复杂逻辑前可用简短说明。
- 组件文件多为 `.jsx`，命名与导入风格保持现有一致（组件名首字母大写、工具函数驼峰）。

## src 逐文件概览
### 根目录
- `src/main.jsx`: 应用入口与根挂载点。
- `src/App.jsx`: 根组件与核心页面结构。
- `src/App.css`: App 级样式（若存在特殊覆盖）。
- `src/index.css`: 全局样式与 Tailwind 基础配置扩展。
- `src/App.test.jsx`: 基础测试占位/示例（当前未启用测试脚本）。

### hooks
- `src/hooks/index.js`: hooks 统一导出。
- `src/hooks/cheatCodes.js`: 作弊码相关逻辑。
- `src/hooks/useAchievements.js`: 成就系统状态与触发逻辑。
- `src/hooks/useDevicePerformance.js`: 设备性能检测与降级策略。
- `src/hooks/useEpicTheme.js`: 史诗主题/视觉风格切换。
- `src/hooks/useGameActions.js`: 游戏操作封装与快捷调用。
- `src/hooks/useGameLoop.js`: 主循环与 tick 逻辑驱动。
- `src/hooks/useGameState.js`: 全局游戏状态管理。
- `src/hooks/useNumberAnimation.js`: 数值滚动/动画工具。
- `src/hooks/useSimulationWorker.js`: 连接 Web Worker 的模拟计算。
- `src/hooks/useSound.js`: 音效管理与播放控制。
- `src/hooks/useViewportHeight.js`: 视口高度适配（移动端）。

### utils
- `src/utils/assetPath.js`: 资源路径拼装与兼容处理。
- `src/utils/buildingUpgradeUtils.js`: 建筑升级计算辅助。
- `src/utils/calendar.js`: 时间与日历相关工具。
- `src/utils/debugFlags.js`: 调试标志与开关。
- `src/utils/diplomaticUtils.js`: 外交相关工具函数。
- `src/utils/economy.js`: 经济相关通用计算。
- `src/utils/effectFormatter.js`: 效果/修正说明文本格式化。
- `src/utils/eventEffectFilter.js`: 事件效果筛选与过滤。
- `src/utils/foreignTrade.js`: 对外贸易辅助逻辑。
- `src/utils/imageRegistry.js`: 图片资源索引与注册。
- `src/utils/livingStandard.js`: 生活水平评分、等级与消费/解锁算法。
- `src/utils/resources.js`: 资源定义辅助与解锁判断。
- `src/utils/soundManager.js`: 音效资源与播放管理。
- `src/utils/styleOptimizer.js`: 样式/动画的性能优化工具。

### components
- `src/components/index.js`: 组件统一导出入口。

#### components/common
- `src/components/common/AchievementToast.jsx`: 成就弹出提示。
- `src/components/common/BattleNotification.jsx`: 战斗提示通知。
- `src/components/common/CompactCard.jsx`: 紧凑型卡片组件。
- `src/components/common/DynamicEffects.jsx`: 通用动态特效集合。
- `src/components/common/EnhancedCards.jsx`: 扩展卡片组件集合。
- `src/components/common/EpicDecorations.jsx`: 史诗风装饰元素。
- `src/components/common/MotionComponents.jsx`: 动画/动效组件封装。
- `src/components/common/SimpleLineChart.jsx`: 简易折线图组件。
- `src/components/common/UIComponents.jsx`: 通用 UI 元件集合。
- `src/components/common/UnifiedUI.jsx`: 统一风格 UI 组件集合。

#### components/game
- `src/components/game/CityMap.jsx`: 城市地图视图。

#### components/layout
- `src/components/layout/BottomNav.jsx`: 底部导航栏。
- `src/components/layout/EraBackground.jsx`: 时代背景/场景背景。
- `src/components/layout/GameControls.jsx`: 游戏控制区按钮/快捷入口。
- `src/components/layout/StatusBar.jsx`: 顶部状态栏。

#### components/modals
- `src/components/modals/AchievementsModal.jsx`: 成就详情弹窗。
- `src/components/modals/AnnualFestivalModal.jsx`: 年度节庆弹窗。
- `src/components/modals/BattleResultModal.jsx`: 战斗结算弹窗。
- `src/components/modals/CardDetailModal.jsx`: 卡片详情弹窗。
- `src/components/modals/DeclareWarModal.jsx`: 宣战弹窗。
- `src/components/modals/DifficultySelectionModal.jsx`: 难度选择弹窗。
- `src/components/modals/EventDetail.jsx`: 事件详情展示。
- `src/components/modals/PopulationDetailModal.jsx`: 人口详情弹窗。
- `src/components/modals/ResourceDetailModal.jsx`: 资源详情弹窗。
- `src/components/modals/SaveSlotModal.jsx`: 存档位选择/管理弹窗。
- `src/components/modals/SaveTransferModal.jsx`: 存档导入导出弹窗。
- `src/components/modals/StratumDetailModal.jsx`: 阶层详情弹窗。
- `src/components/modals/TradeRoutesModal.jsx`: 贸易路线弹窗。
- `src/components/modals/TutorialModal.jsx`: 新手引导弹窗。
- `src/components/modals/WikiModal.jsx`: 游戏百科/说明弹窗。

#### components/panels
- `src/components/panels/CoalitionPanel.jsx`: 执政联盟面板。
- `src/components/panels/DecreeDetailSheet.jsx`: 法令详情底部面板。
- `src/components/panels/DemandsList.jsx`: 诉求列表组件。
- `src/components/panels/DissatisfactionAnalysis.jsx`: 不满来源分析面板。
- `src/components/panels/EmpireScene.jsx`: 帝国场景展示面板。
- `src/components/panels/LogPanel.jsx`: 日志/事件记录面板。
- `src/components/panels/ResourcePanel.jsx`: 资源概览面板。
- `src/components/panels/SettingsPanel.jsx`: 设置面板。
- `src/components/panels/StrategicActionButton.jsx`: 策略行动按钮组件。
- `src/components/panels/StrategicActionCard.jsx`: 策略行动卡片组件。
- `src/components/panels/StrataPanel.jsx`: 阶层列表面板。
- `src/components/panels/StratumDetailSheet.jsx`: 阶层详情底部面板。
- `src/components/panels/TechDetailSheet.jsx`: 科技详情底部面板。
- `src/components/panels/UnitDetailSheet.jsx`: 单位/军队详情底部面板。

#### components/tabs
- `src/components/tabs/BottomSheet.jsx`: 底部抽屉容器。
- `src/components/tabs/BuildTab.jsx`: 建筑/发展页签。
- `src/components/tabs/BuildingDetails.jsx`: 建筑详情展示。
- `src/components/tabs/BuildingUpgradePanel.jsx`: 建筑升级面板。
- `src/components/tabs/DiplomacyTab.jsx`: 外交页签。
- `src/components/tabs/MilitaryTab.jsx`: 军事页签。
- `src/components/tabs/OverviewTab.jsx`: 总览页签。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HkingAuditore/civ-game](https://github.com/HkingAuditore/civ-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
