---
trigger: always_on
description: We use pnpm, vite, vue 3 with vue-router and mdui 2.
---

We use pnpm, vite, vue 3 with vue-router and mdui 2.
Always use MDUI 2 for elements.

Use `slot` prop when needed or already written EVEN OUTDATED.

Avoid explaining what you're doing in code comments.

When unnecessary, don't create a bunch of functions to finish a simple task.

After completing all tasks in the request, execute `pnpm check` to check for ts errors and `pnpm pret:fix` to format the code when needed.

# mdui v2 Documentation

mdui is an front-end component library based on Material Design 3, using Web Components, supporting dynamic colors, dark mode, and lightweight efficiency.

## 开发指南

- [概述](https://www.mdui.org/zh-cn/docs/2/index.md): 让我们通过 mdui 的 CDN 和一个最简单的页面模板来开始使用 mdui。
- [安装](https://www.mdui.org/zh-cn/docs/2/getting-started/installation.md): 你可以选择通过 npm 安装 mdui，或者从 CDN 引入 mdui。推荐使用 npm 进行安装。
- [快速入门](https://www.mdui.org/zh-cn/docs/2/getting-started/usage.md): mdui 的组件都是标准的 Web Components 组件，你可以像使用 `<div>` 标签一样使用 mdui 组件。每个组件的文档中都详细描述了其完整的 API，包括属性、方法、事件、slot、CSS Part、CSS 自定义属性等。
- [TypeScript 支持](https://www.mdui.org/zh-cn/docs/2/getting-started/typescript-support.md): mdui 是用 TypeScript 开发的，因此对 TypeScript 提供了良好的支持。所有的 mdui 官方库都自带类型声明文件，可以直接使用。
- [IDE 支持](https://www.mdui.org/zh-cn/docs/2/getting-started/ide-support.md): mdui 专门为 VS Code 和 WebStorm 进行了优化，在这些 IDE 中可以获得代码自动完成、悬停提示等功能。
- [本地化](https://www.mdui.org/zh-cn/docs/2/getting-started/localization.md): mdui 内部默认使用英文，如果需要使用其他语言，则需要进行多语言配置。
- [常见问题](https://www.mdui.org/zh-cn/docs/2/getting-started/faq.md): 以下整理了一些 mdui 社区常见的问题和官方答复，在提问之前建议找找有没有类似的问题。

## AI 辅助开发

- [LLMs.txt](https://www.mdui.org/zh-cn/docs/2/ai/llms.md): mdui 提供了 `llms.txt` 与 `llms-full.txt`，用于为 LLM（大语言模型）提供准确、可引用的上下文，帮助 AI 更可靠地回答 mdui 相关问题。
- [MCP 服务](https://www.mdui.org/zh-cn/docs/2/ai/mcp.md): mdui 提供专用的 [MCP（Model Context Protocol）](https://modelcontextprotocol.io/) 服务器 `@mdui/mcp`，用于在本地为 AI 智能体提供组件、图标、CSS 变量和文档的查询能力。

## 样式

- [暗色模式](https://www.mdui.org/zh-cn/docs/2/styles/dark-mode.md): mdui 的所有组件都支持暗色模式，并且支持根据操作系统的设置自动切换主题。
- [动态配色](https://www.mdui.org/zh-cn/docs/2/styles/dynamic-color.md): mdui 提供了动态配色功能。只需提供一个颜色值，mdui 就能自动生成一套完整的配色方案。此外，mdui 还支持从指定的壁纸中提取主色调，并据此生成配色方案。
- [文章排版](https://www.mdui.org/zh-cn/docs/2/styles/prose.md): mdui 提供了 `mdui-prose` 和 `mdui-table` CSS 类，专门用于优化文章和表格的样式。
- [设计令牌](https://www.mdui.org/zh-cn/docs/2/styles/design-tokens.md): 设计令牌（Design Tokens）是一种用于管理设计系统的策略。

## 与框架集成

- [与 React 集成](https://www.mdui.org/zh-cn/docs/2/frameworks/react.md): 在 React 中使用 mdui 时，只需要按照 [安装](/zh-cn/docs/2/getting-started/installation#npm) 页面的步骤完成安装即可。
- [与 Vue 集成](https://www.mdui.org/zh-cn/docs/2/frameworks/vue.md): 在 Vue 中使用 mdui 时，首先需要按照 [安装](/zh-cn/docs/2/getting-started/installation#npm) 页面的指引完成安装，然后进行一些必要的配置。
- [与 Angular 集成](https://www.mdui.org/zh-cn/docs/2/frameworks/angular.md): 在 Angular 中使用 mdui 时，首先需要按照 [安装](/zh-cn/docs/2/getting-started/installation#npm) 页面的指引完成安装，然后进行一些必要的配置。
- [与其他框架集成](https://www.mdui.org/zh-cn/docs/2/frameworks/others.md): mdui 使用浏览器原生支持的 Web Components 开发，因此能在所有 Web 框架中使用。下面列举了在常用框架中使用 mdui 的方法。

## 组件

- [头像组件 Avatar](https://www.mdui.org/zh-cn/docs/2/components/avatar.md): 头像用于表示用户或事物，支持多种形式，包括图片、图标或字符等。
- [徽标组件 Badge](https://www.mdui.org/zh-cn/docs/2/components/badge.md): 徽标用于展示动态信息，如计数或状态指示。它可以包含文字或数字。
- [底部应用栏组件 BottomAppBar](https://www.mdui.org/zh-cn/docs/2/components/bottom-app-bar.md): 底部应用栏主要用于在移动端页面底部展示导航项和其他重要操作。
- [按钮组件 Button](https://www.mdui.org/zh-cn/docs/2/components/button.md): 按钮主要用于执行一些操作，例如发送邮件、分享文档或点赞评论等。
- [图标按钮组件 ButtonIcon](https://www.mdui.org/zh-cn/docs/2/components/button-icon.md): 图标按钮主要用于执行一些次要的操作。
- [卡片组件 Card](https://www.mdui.org/zh-cn/docs/2/components/card.md): 卡片是一个多功能组件，用于承载与单一主题相关的内容和操作。
- [复选框组件 Checkbox](https://www.mdui.org/zh-cn/docs/2/components/checkbox.md): 复选框允许用户从一组选项中选择一个或多个选项，或者切换单个选项的开/关状态。
- [纸片组件 Chip](https://www.mdui.org/zh-cn/docs/2/components/chip.md): 纸片组件用于辅助用户输入信息、进行选择、筛选内容或执行相关操作。
- [圆形进度指示器组件 CircularProgress](https://www.mdui.org/zh-cn/docs/2/components/circular-progress.md): 圆形进度指示器是一个用于显示任务进度的圆形组件，例如数据加载或表单提交等。
- [折叠面板组件 Collapse](https://www.mdui.org/zh-cn/docs/2/components/collapse.md): 折叠面板组件用于将复杂的内容区域进行分组和隐藏，以保持页面的整洁。
- [对话框组件 Dialog](https://www.mdui.org/zh-cn/docs/2/components/dialog.md): 对话框用于在用户的操作流程中提供重要提示。
- [分割线组件 Divider](https://www.mdui.org/zh-cn/docs/2/components/divider.md): 分隔线是一条细线，用于在列表和容器中对内容进行分组。
- [下拉组件 Dropdown](https://www.mdui.org/zh-cn/docs/2/components/dropdown.md): 下拉组件用于在一个弹出的控件中展示特定内容，通常与菜单组件一起使用。
- [浮动操作按钮组件 Fab](https://www.mdui.org/zh-cn/docs/2/components/fab.md): 浮动操作按钮（FAB）用于突出显示页面上的主要操作，它将关键操作置于易于访问的位置。
- [图标组件 Icon](https://www.mdui.org/zh-cn/docs/2/components/icon.md): 图标用于表示常见的操作。它支持 Material Icons 图标，也支持使用 SVG 图标。
- [布局组件 Layout](https://www.mdui.org/zh-cn/docs/2/components/layout.md): 布局组件提供了一个灵活的布局系统，用于创建复杂的页面布局。
- [线性进度指示器组件 LinearProgress](https://www.mdui.org/zh-cn/docs/2/components/linear-progress.md): 线性进度指示器是一种横向的指示器，用于向用户展示任务的执行进度，如数据加载或表单提交等。
- [列表组件 List](https://www.mdui.org/zh-cn/docs/2/components/list.md): 列表是一种垂直排列的索引，用于展示文本或图片，便于用户快速浏览和访问相关信息。
- [菜单组件 Menu](https://www.mdui.org/zh-cn/docs/2/components/menu.md): 菜单组件提供了一系列垂直排列的选项。当用户与按钮、或其他控件交互时，将显示菜单。
- [底部导航栏组件 NavigationBar](https://www.mdui.org/zh-cn/docs/2/components/navigation-bar.md): 导航栏用于在移动端页面中方便地在几个主要页面之间进行切换。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [realtvop/SaltNet](https://github.com/realtvop/SaltNet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
