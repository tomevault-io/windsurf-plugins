---
trigger: always_on
description: 公众号文章排版样式规范与装饰性卡片模板。提供完整的排版参数（字号、行高、字间距、颜色）和可复用的装饰性卡片 HTML 代码。当用户需要：(1) 创建或编辑公众号文章，(2) 了解公众号排版规范，(3) 使用装饰性卡片样式，(4) 生成公众号格式的 HTML 内容时使用此技能。
---


# 公众号文章排版样式

## 概述

本技能提供公众号文章的完整排版规范和装饰性卡片模板，基于对南信大海科院公众号文章的深度分析。包含字号、行高、字间距、颜色等核心参数，以及信息卡片、标题标签、引用块等装饰性元素的完整 HTML 代码。

## 快速参考

### 核心排版参数

| 参数 | 正文 | 图注 | 标题 | 强调 |
|------|------|------|------|------|
| 字号 | 15px | 14px | 16-18px | 15px |
| 行高 | 1.8 | 1.6 | 1.8 | 1.8 |
| 字间距 | 1.6px | 0px | 1px | 1.6px |
| 对齐 | justify | center | justify | justify |
| 颜色 | rgb(62,62,62) | rgb(62,62,62) | rgb(62,62,62) | rgb(62,62,62) |
| 首行缩进 | 2.2133em | 0em | - | - |

### 颜色规范

| 用途 | 颜色值 | 说明 |
|------|--------|------|
| 正文 | rgb(62, 62, 62) | 深灰色，非纯黑 |
| 主色调-蓝 | rgb(33, 150, 243) | 强调、标签背景 |
| 主色调-青蓝 | rgb(95, 156, 239) | 装饰、边框 |
| 主色调-浅蓝 | rgb(120, 185, 226) | 分隔线、装饰 |
| 深蓝-1 | rgb(22, 93, 155) | 边框、重点 |
| 深蓝-2 | rgb(16, 76, 117) | 边框 |
| 浅蓝背景-1 | rgb(199, 225, 247) | 卡片背景 |
| 浅蓝背景-2 | rgb(241, 245, 255) | 卡片背景 |
| 浅蓝背景-3 | rgb(244, 250, 255) | 卡片背景 |
| 浅蓝背景-4 | rgb(245, 249, 252) | 卡片背景 |
| 浅蓝背景-5 | rgb(249, 253, 254) | 卡片背景 |
| 浅蓝背景-6 | rgb(242, 247, 252) | 卡片背景 |
| 浅蓝背景-7 | rgb(225, 240, 253) | 卡片背景 |
| 白色 | rgb(255, 255, 255) | 文字、背景 |
| 浅灰 | rgb(245, 245, 245) | 背景 |
| 边框灰 | rgb(189, 203, 212) | 虚线边框 |
| 绿色-1 | rgb(75, 143, 120) | 装饰点 |
| 绿色背景 | rgb(236, 249, 243) | 卡片背景 |
| 青色-1 | rgb(43, 158, 228) | 渐变起始 |
| 青色-2 | rgb(0, 210, 192) | 渐变结束 |
| 青色-3 | rgb(121, 181, 229) | 渐变起始 |
| 青色-4 | rgb(72, 211, 208) | 渐变结束 |

## 正文排版

### 标准正文段落

```html
<section style="text-align: justify; font-size: 15px; line-height: 1.8; letter-spacing: 1.6px; box-sizing: border-box;">
    <p style="text-indent: 2.2133em; white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
        <span leaf="">这里是正文内容，首行缩进。</span>
    </p>
</section>
```

### 强调文字

```html
<strong style="box-sizing: border-box;">
    <span leaf="">强调内容</span>
</strong>
```

### 特殊首行缩进

```html
<!-- 2.2em 缩进（部分文章使用） -->
<p style="text-indent: 2.2em; ...">
```

## 分隔线装饰

### 1. 双横线+圆点分隔线

适用于：章节分隔、视觉装饰

```html
<section style="text-align: left; justify-content: flex-start; display: flex; flex-flow: row; margin: 10px 0px; box-sizing: border-box;">
    <section style="display: inline-block; vertical-align: middle; width: auto; flex: 100 100 0%; height: auto; padding: 0px 10px 0px 0px; align-self: center; box-sizing: border-box;">
        <section style="margin: 5px 0px; box-sizing: border-box;">
            <section style="background-color: rgb(120, 185, 226); height: 1px; box-sizing: border-box;">
                <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: middle; width: auto; align-self: center; flex: 0 0 auto; min-width: 5%; max-width: 100%; height: auto; padding: 0px; line-height: 0; box-sizing: border-box;">
        <section style="text-align: center; box-sizing: border-box;">
            <section style="display: inline-block; width: 16px; height: 16px; vertical-align: top; overflow: hidden; border-width: 0px; border-radius: 100%; border-style: none; border-color: rgb(62, 62, 62); background-color: rgba(120, 185, 226, 0.2); box-sizing: border-box;">
            </section>
        </section>
        <section style="text-align: center; margin: -6px 0px 0px; box-sizing: border-box;">
            <section style="display: inline-block; width: 18px; height: 18px; vertical-align: top; overflow: hidden; border-width: 0px; border-radius: 100%; border-style: none; border-color: rgb(62, 62, 62); background-color: rgba(120, 185, 226, 0.2); box-sizing: border-box;">
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: middle; width: auto; align-self: center; padding: 0px 0px 0px 10px; flex: 100 100 0%; height: auto; box-sizing: border-box;">
        <section style="margin: 5px 0px; box-sizing: border-box;">
            <section style="background-color: rgb(120, 185, 226); height: 1px; box-sizing: border-box;">
                <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
            </section>
        </section>
    </section>
</section>
```

### 2. 左右对称短横线

适用于：标题上方装饰

```html
<section style="text-align: left; justify-content: flex-start; display: flex; flex-flow: row; margin: 10px 0px 0px; box-sizing: border-box;">
    <section style="display: inline-block; vertical-align: top; width: 50%; align-self: flex-start; flex: 0 0 auto; height: auto; line-height: 0; padding: 0px 0px 0px 4px; box-sizing: border-box;">
        <section style="display: flex; width: 100%; flex-flow: column; box-sizing: border-box;">
            <section style="z-index: 1; box-sizing: border-box;">
                <section style="margin: 0px; box-sizing: border-box;">
                    <section style="display: inline-block; width: 20px; height: 4px; vertical-align: top; overflow: hidden; background-color: rgb(33, 150, 243); margin: 0px; box-sizing: border-box;">
                        <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
                    </section>
                </section>
            </section>
        </section>
    </section>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makabaka11/wechat-article-style.skill](https://github.com/makabaka11/wechat-article-style.skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
