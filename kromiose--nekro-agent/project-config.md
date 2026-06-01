---
trigger: always_on
description: 主题系统采用分层架构设计，将主题定义与使用完全分离，支持多主题扩展。
---

# 前端主题系统开发指南

## 主题系统架构

主题系统采用分层架构设计，将主题定义与使用完全分离，支持多主题扩展。

### 核心文件结构

- `palette.ts` - 调色板定义，包含所有主题的原始颜色配置
- `themeConfig.ts` - 主题配置与管理，提供主题获取和切换的核心功能
- `variants.ts` - 组件样式变体，定义各种UI组件的通用样式变体
- `ThemeProvider.tsx` - 主题提供者组件，为整个应用提供主题
- `ThemeToggleButton.tsx` - 主题切换按钮组件，用于切换主题模式
- `index.tsx` - 主题入口文件，统一导出所有主题相关组件和功能

## 基本原则

- **严禁硬编码颜色值**：禁止在组件中使用硬编码的颜色值，必须从主题系统中获取
- **禁止使用主题判断条件**：不要使用 `theme.palette.mode === 'dark'` 这类条件判断，应使用动态属性
- **支持多主题模式**：系统支持亮色、暗色和跟随系统三种模式，未来可扩展为更多自定义主题
- **组件样式变体化**：使用预定义的样式变体，而非在每个组件中重复定义样式
- **调色板与使用分离**：调色板定义与样式使用完全分离，便于主题扩展和统一管理
- **持续更新**：当实现发生变化或者追加新的重要知识时，持续更新本文档的内容保持其最新且可用

## 主题使用指南

### 获取主题配置

通过主题配置管理函数获取当前主题配置：

```tsx
import { getCurrentThemeMode, getCurrentPalette, getCurrentUIElements } from '../../theme/themeConfig';

// 获取当前主题模式 ('light' | 'dark')
const mode = getCurrentThemeMode();

// 获取当前主题的调色板
const palette = getCurrentPalette();

// 获取当前主题的UI元素配置
const uiElements = getCurrentUIElements();

// 根据路径获取当前主题的特定配置项
import { getThemeToken } from '../../theme/themeConfig';
const palette = getThemeToken('palette');
const ui = getThemeToken('ui');
```

### 使用组件级样式

对于特定组件的样式，使用预定义的组件级样式：

```tsx
import { LOG_TABLE_STYLES, LOGIN_PAGE_STYLES } from '../../theme/themeConfig';

// 在组件中使用
<Box sx={{
  backgroundColor: LOG_TABLE_STYLES.SEVERITY.INFO.backgroundColor,
  color: LOG_TABLE_STYLES.SEVERITY.INFO.color
}} />

// 登录页使用
<Paper sx={{
  boxShadow: LOGIN_PAGE_STYLES.SHADOW.CARD,
  background: LOGIN_PAGE_STYLES.CARD,
  border: LOGIN_PAGE_STYLES.BORDER
}} />
```

### 使用样式变体

样式变体是预定义的样式配置，用于快速应用常见样式：

```tsx
import { 
  CARD_VARIANTS, 
  BUTTON_VARIANTS, 
  INPUT_VARIANTS, 
  SCROLLBAR_VARIANTS
} from '../../theme/variants';

// 卡片变体
<Card sx={CARD_VARIANTS.default.styles} />
<Card sx={CARD_VARIANTS.flat.styles} />
<Card sx={CARD_VARIANTS.transparent.styles} />

// 按钮变体
<Button sx={BUTTON_VARIANTS.primary.styles} />
<Button sx={BUTTON_VARIANTS.secondary.styles} />

// 输入框变体
<TextField sx={INPUT_VARIANTS.default.styles} />

// 滚动条样式
<Box sx={SCROLLBAR_VARIANTS.default.styles} />
<Box sx={SCROLLBAR_VARIANTS.thin.styles} />
```

### 使用主题切换按钮

在应用中添加主题切换按钮：

```tsx
import { ThemeToggleButton } from '../../theme';

// 使用默认大小
<ThemeToggleButton />

// 指定大小
<ThemeToggleButton size="small" />
```

## 扩展主题系统

### 添加新的主题

要添加新的主题，需要在 `palette.ts` 中定义新的主题配置：

```tsx
// 在 palette.ts 文件中添加新主题
export const themes: Record<ThemeKeys, ThemeConfig> = {
  light: { /* 现有浅色主题 */ },
  dark: { /* 现有暗色主题 */ },
  myCustomTheme: {
    palette: {
      // 定义调色板
      primary: {
        main: '#3F51B5',
        light: '#3F51B5',
        dark: '#3F51B5',
        lighter: '#7986CB',
        darker: '#303F9F',
        highlight: '#8C9EFF',
      },
      // 其他颜色...
    },
    ui: {
      // 定义UI元素样式
      background: {
        main: '#FAFAFA',
        // 其他背景样式...
      },
      // 其他UI样式...
    }
  }
};
```

### 添加新的组件样式

为特定组件添加专用主题样式：

```tsx
// 在 themeConfig.ts 文件中添加
export const NEW_COMPONENT_STYLES = {
  // 背景样式
  get BACKGROUND() {
    const { background } = getCurrentUIElements();
    const mode = getCurrentThemeMode();
    return {
      light: `linear-gradient(...)`,
      dark: `linear-gradient(...)`
    }[mode];
  },
  // 其他样式...
};
```

### 添加新的样式变体

为组件添加新的样式变体：

```tsx
// 在 variants.ts 文件中添加
export const MY_COMPONENT_VARIANTS = {
  // 自定义变体
  custom: {
    get styles(): SxProps<Theme> {
      const { shadow, border, background } = getThemeToken('ui');
      return {
        background: background.card,
        boxShadow: shadow.card,
        // 其他样式...
      };
    },
  },
};
```

## 主题切换原理

主题切换基于 Zustand 状态管理和 React Context：

1. `useColorMode` 存储当前主题模式（light/dark/system）
2. `ThemeProvider` 根据当前模式创建并提供 MUI 主题
3. 所有样式通过动态 getter 实时获取当前主题的样式

主题切换顺序为：浅色 -> 暗色 -> 系统 -> 浅色

## 最佳实践

1. 使用组件样式变体代替内联样式
2. 使用预定义的主题配置代替硬编码值
3. 所有新增页面和组件必须支持浅色和暗色两种主题
4. 通过扩展现有变体创建新样式，而非从头开始
5. 为特殊页面创建专用样式对象，便于集中管理
6. 确保透明度和阴影在两种主题下都有良好表现
7. 测试所有组件在主题切换时的平滑过渡

## 页面体验与信息架构约束

以下规则适用于工作区页面、配置页、设置页以及其他带有编辑与切换行为的中后台页面。

### 交互可供性优先于解释性文案

- 不要通过“点击可查看”“可点击打开”“点击进入”这类说明性文字提示用户哪里可以交互
- 可点击元素应通过悬浮态、按压反馈、指针样式、层级变化、图标暗示和动效来自然表达
- 浮层、菜单和列表中的可点击项，默认应具备清晰的 hover 态和点击命中区，而不是额外附加解释文案
- 若用户需要靠阅读说明才知道某处可以点击，优先判定为样式与信息层级设计失败，而不是文案缺失
- 说明文案只用于补充业务语义，不用于弥补交互设计本身的可理解性不足

### 1. 页面不应做成“表单堆叠”

- 禁止把多个大号输入框或配置块直接从上到下平铺成“后台表单墙”
- 优先使用“总览层 + 聚焦编辑层”的结构，让用户先理解信息分区，再进入具体编辑
- 当页面存在多个语义区域时，顶部应先提供清晰的模块入口卡片、分段导航或摘要区，而不是直接暴露全部编辑器
- 模块切换后的主要内容区域应尽量连续，不要被无关信息区块打断

### 2. 信息按“用户目的”分组，而不是按实现过程分组

- 页面分区名称必须面向最终用户的任务目标，不要暴露设计过程、实现历史、迁移状态或内部建模术语
- 禁止在页面主文案中出现“升级为”“不再只是”“编排器”“系统改造”等研发过程描述
- 如果必须保留技术术语，应将其降级为副标题、附注或悬浮说明，不应作为页面的主心智
- 页面标题、分区标题、按钮文案应优先表达“用户现在能做什么”，而不是“系统内部如何工作”

### 3. 文案保持专业、简洁、可直接操作

- 面向最终用户的页面，默认使用简短、专业、明确的文案，避免教程式长段说明
- 主区域只保留必要的短说明；补充解释、注意事项、概念说明优先放入 Tooltip、帮助图标或次级说明区
- 如无必要，不要额外添加标题、引导语、说明段落或标签文字来解释一个本应通过结构和控件关系就能自解释的操作区
- 模糊表述必须改成明确表述，例如“后续可继续更新”应改为“可由 CC 自动更新”
- 不要随意引入新概念。如果用户已接受既有术语体系，应保持一致，不得擅自重命名为另一套说法

### 4. 切换控件必须紧凑且一致

- 所有具有“二选一 / 多选一”性质的切换控件，一律优先使用 `ButtonGroup`、Tabs 或 Segmented Control 风格，不要并排摆放多个独立按钮
- 切换控件应尽量内聚到对应标题行、工具条或局部控制区，避免单独占据一整行造成空间浪费
- 只读视图与编辑视图、预览与源码、不同对象配置之间的切换，应使用统一的控件形态和交互位置
- 展开/收起类卡片，点击整个卡片主体即可触发展开时，不应只允许点击一个小图标
- 页面级或模块级主导航使用 `Tabs`；工具条内的筛选、视图切换、预览/源码、局部模式切换使用 `SegmentedControl`；不要把两者混成同一种视觉层级
- 新增局部切换控件时，优先复用公共 `SegmentedControl` 或其主题变体，不要在页面里重复手写 `ToggleButtonGroup` / `ButtonGroup` 的局部 `sx`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KroMiose/nekro-agent](https://github.com/KroMiose/nekro-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
