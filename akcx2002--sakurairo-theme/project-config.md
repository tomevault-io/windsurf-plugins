---
trigger: always_on
description: A comprehensive skill for using the Sakurairo WordPress theme features. Use this skill when creating articles, building pages, using shortcodes, setting up galleries/exhibitions, configuring homepage components, working with ChatGPT/AI features, managing Bilibili/Steam/QQ integrations, customizing theme options, or writing content with theme-specific features. Triggers on tasks involving Sakurairo content creation, page building, shortcode usage, theme functionality, and visual customization.
---


# Sakurairo Theme - 功能使用技能

## 概述

本技能提供使用 Sakurairo WordPress 主题各项功能的完整指导。Sakurairo 是一款基于 Sakura V3 Series 重构开发的 WordPress 主题，具有 AI 辅助阅读、多服务 API 集成（B站、Steam、QQ、Bangumi）、丰富的页面构建能力、灵活的首页组件系统、多样的文章展示样式等特性。本技能聚焦于**内容创作者和网站管理者**的日常使用场景。

---

## 一、文章写作与内容展示

### 1.1 文章 AI 摘要（AI Excerpt）

主题集成了 ChatGPT AI 辅助阅读功能，可为文章自动生成摘要。

**启用方式：**
- 在后台主题选项 → ChatGPT/AIGC 设置中配置 API Key
- 发表或更新文章时，通过 REST API 触发摘要生成：
  - 管理员调用 `GET /wp-json/sakura/v1/chatgpt?post_id={文章ID}`
- 生成的摘要存储在文章自定义字段 `ai_summon_excerpt` 中
- 在前台单页文章顶部以蓝色提示框显示，带有 `AI Excerpt` 标签

**使用场景：** 为长篇文章自动生成精炼摘要，提升读者阅读体验。

### 1.2 AI 名词注释（AI Annotations）

自动识别文章中复杂或专业术语，并生成浮动注释。

**启用方式：**
- 管理员调用 `GET /wp-json/sakura/v1/chatgpt/annotate?post_id={文章ID}`
- 注释数据存储在 `iro_chatgpt_annotations` 自定义字段中
- 文章中匹配的关键词会自动显示为带下划线的可点击注释

**使用场景：** 技术文章、科普文章中为专业术语添加即时解释。

### 1.3 文章目录（Table of Contents）

自动根据文章中的标题标签（h1~h6）生成目录。

**启用方式：** 主题选项 → 文章设置 → 开启「自动生成文章目录」

**行为说明：**
- 文章含有一个或多个标题标签时自动显示目录
- 目录在桌面端侧边栏浮动显示
- 移动端通过 `layouts/mo_toc_menu.php` 显示为折叠菜单

### 1.4 文章字数统计与阅读时间

自动统计文章字数并估算阅读时长。

**功能：**
- 支持 CJK（中日韩）字符和英文单词混合统计（详见 `inc/word-stat.php`）
- 阅读时间按每分钟 220 字估算
- 显示在文章 meta 区域，格式如："约 5 分钟"、"不到 1 分钟"

### 1.5 文章许可协议

每篇文章可单独设置或继承全局知识共享许可协议。

**支持协议：**
- `cc-by-nc-sa`（默认，署名-非商业性使用-相同方式共享 4.0）
- `cc0`（公共领域贡献）
- 可在文章编辑页面单独设置 `license` 自定义字段覆盖全局设置

### 1.6 文章内容样式

主题提供两种文章内容渲染风格，可在主题选项中切换：
- **Sakura 风格**（`css/content-style/sakura.css`）- 柔和圆润
- **GitHub 风格**（`css/content-style/github.css`）- 简洁清晰

### 1.7 代码高亮

文章中的代码块自动应用语法高亮（通过 Highlight.js）。

Gutenberg 编辑器中的代码块可设置 `language` 属性，自动添加 `language-xxx` CSS 类。

---

## 二、短代码（Shortcodes）

### 2.1 使用说明

在文章或页面编辑器中直接插入以下短代码即可调用相应功能。

### 2.2 音乐播放器（APlayer / Meting）

```
[ap layer]
```
通过 Meting API 集成网易云音乐/QQ音乐，在文章中嵌入音乐播放器。

**后台配置：** 主题选项 → 社交/集成 → Meting API 设置

### 2.3 Bilibili 视频

```
[bilibili bvid="BV1xx411c7mD"]
```

嵌入 Bilibili 视频到文章中。支持通过 BVID 或 AVID 引用视频。

### 2.4 画廊/相册

```
[gallery]
```
在文章中嵌入图片画廊。支持从媒体库选择图片，也可以通过 REST API 获取图库数据。

### 2.5 其他短代码

更多短代码样式定义在 `css/shortcodes.css` 中，包括各类图文混排布局。

### 2.6 Gutenberg 编辑器块

主题提供了自定义 Gutenberg 块（`inc/blocks/`），在古腾堡编辑器中可用：

- 添加 Font Awesome 图标
- 自定义布局块
- 代码块语言标记支持（自动为代码块添加 `language-xxx` CSS 类）

---

## 三、页面创建与特殊页面类型

### 3.1 首页组件系统

首页不是固定模板，而是由**组件排序系统**驱动的。在主题选项中可调整以下组件的顺序：

| 组件 | 功能 |
|---|---|
| `static_page` | 在首页嵌入一个静态页面内容 |
| `exhibition` | 展示区/精选区（热门文章、特色内容） |
| `primary` | 主文章列表（博客流） |

**配置路径：** 主题选项 → 首页设置 → 首页组件排序

### 3.2 归档页面

主题提供归档信息 REST API 端点（`/wp-json/sakura/v1/archive_info`），用于展示文章归档统计。

通过创建页面并选择归档模板来使用。

### 3.3 友情链接页面

主题启用了 WordPress 内置的链接管理器（通过 `pre_option_link_manager_enabled` 过滤器）。

**友情链接状态检测：**
- 后台新增「链接状态」管理页面（`inc/link-status.php`）
- 可手动检测单个或所有友情链接的可用性
- 记录 HTTP 状态码、失败次数、错误信息

### 3.4 展示区（Exhibition）

首页展示区组件可显示多种站点统计信息方块，在主题选项中配置：

| 功能方块 | 说明 |
|---|---|
| 文章数 | 显示站点文章总数 |
| 评论数 | 显示评论总数 |
| 访客数 | 显示访问量 |
| 链接数 | 显示友情链接数量 |
| 运行时长 | 显示站点已运行时间 |

**配置路径：** 主题选项 → 首页设置 → 展示区组件

### 3.5 分类/标签图片

主题支持为分类和标签添加自定义图片（`inc/categories-images.php`）。

**设置方式：**
- 编辑分类/标签时出现「分类/标签图像」字段
- 支持上传自定义图片
- 前台分类/标签页面显示对应图片

---

## 四、首页与封面设置

### 4.1 封面区域

首页顶部封面支持多种模式：

| 模式 | 设置选项 |
|---|---|
| 视频封面 | 上传/链接视频文件，支持直播流、循环播放 |
| 图片封面 | 设置静态封面图片 |
| 全屏模式 | 封面占满整个视口高度 |
| 非全屏模式 | 封面固定高度 |

**配置路径：** 主题选项 → 首页封面

### 4.2 个人头像

支持使用本地图片作为头像，防止外源 CDN 抽风问题。
- 设置项：`personal_avatar`（在主题选项中上传）
- 兜底：使用 WordPress 默认 Gravatar

### 4.3 文本 Logo

支持自定义文本 Logo：
```php
// 设置示例
text_logo = [
    'color' => '#ff6496',
    'size'  => '24',
    'font'  => 'Noto Serif SC'
]
```

---

## 五、社交媒体与外部服务集成

### 5.1 Bilibili 集成

| 功能 | 说明 |
|---|---|
| 视频嵌入 | 通过 BVID 嵌入 B 站视频 |
| 追番列表 | 展示用户的 Bilibili 追番数据 |
| 收藏夹 | 展示用户的 Bilibili 视频收藏 |
| 电影 | 展示用户的 Bilibili 电影数据 |

**后台配置：** 主题选项 → 社交/集成 → Bilibili 设置
**缓存：** 数据通过 WordPress Transients 缓存，可在「缓存设置」页面管理。

### 5.2 Steam 游戏库

展示用户的 Steam 游戏库信息。
- 需要 Steam API Key
- 数据通过 Transients 缓存

### 5.3 QQ 集成

- 通过 QQ 号获取头像和用户信息
- 常用于评论区的用户头像显示（通过 `sakura/v1/qqinfo` API 端点）

### 5.4 Bangumi 追番

支持 Bangumi.tv 和 Bilibili 两种追番数据源。

### 5.5 MyAnimeList

支持获取用户在 MyAnimeList 上的动漫列表。

### 5.6 音乐播放器（APlayer）

通过 Meting API 集成网易云音乐、QQ 音乐等平台的歌曲播放。

### 5.7 图片上传服务

评论和文章中上传图片支持多种后端服务：

| 服务 | API 类型 |
|---|---|
| Imgur | 需要 Client ID |
| SM.MS | 需要 API Key |
| Chevereto | 需要 API Key |
| Lsky Pro | 需要 API Token |

---

## 六、评论系统

### 6.1 功能特性

- AJAX 无刷新提交评论
- 评论排序可配置（正序/倒序）
- 验证码支持（CAPTCHA / Cloudflare Turnstile / Vaptcha）
- QQ 头像自动获取
- Gravatar 代理支持（可配置自定义代理地址）

### 6.2 验证码配置

在主题选项中选择验证码服务：
- **CAPTCHA**（内置，通过 `sakura/v1/captcha/create` API 生成）
- **Cloudflare Turnstile**（隐私友好的替代方案）
- **Vaptcha**（支持语音/图片验证）

---

## 七、导航与交互体验

### 7.1 导航菜单

主题注册了一个导航菜单位置（`primary`），在后台 → 外观 → 菜单中配置。

**特性：**
- 支持多级下拉菜单
- 文本 Logo 显示（可在主题选项配置颜色、大小、字体）
- 移动端汉堡菜单

### 7.2 Pjax 无刷新导航

主题支持 Pjax 无刷新页面跳转（`poi_pjax` 选项），实现：
- 页面切换无闪烁
- 音乐播放不会中断
- 过渡动画平滑

开启后，需要给某些链接添加 `data-no-pjax` 属性来排除（如登录/退出链接）。

### 7.3 实时搜索（Live Search）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AKCX2002/sakurairo-theme](https://github.com/AKCX2002/sakurairo-theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
