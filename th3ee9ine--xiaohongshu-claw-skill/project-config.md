---
trigger: always_on
description: > **小红书笔记全流程 AI 创作 Skill** — 从选题到发布，一站式生成合规的图文笔记。
---

# xiaohongshu-claw-skill

> **小红书笔记全流程 AI 创作 Skill** — 从选题到发布，一站式生成合规的图文笔记。
>
> 当前版本：**v1.0.0**

[![OpenClaw](https://img.shields.io/badge/OpenClaw-Skill-brightgreen)](https://clawhub.ai)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 目录

1. [快速开始](#快速开始)
2. [完整流水线](#完整流水线)
3. [笔记 JSON 格式](#笔记-json-格式)
4. [Section 类型详解](#section-类型详解)
5. [违禁词检测](#违禁词检测)
6. [标题生成](#标题生成)
7. [质量评分](#质量评分)
8. [图片指令生成](#图片指令生成)
9. [模板系统](#模板系统)
10. [Python API 速查](#python-api-速查)
11. [脚本参考](#脚本参考)
12. [示例 JSON 说明](#示例-json-说明)
13. [文件结构](#文件结构)
14. [注意事项](#注意事项)

---

## 快速开始

```bash
# 1. 一键跑完整流水线（选题 → 写作 → 图片 → 渲染 → 合规检查）
python3 scripts/run_pipeline.py --topic "探店咖啡厅" --template food-explore

# 2. 仅检测违禁词
python3 scripts/check_banned_words.py --text "全网第一！顶级品质，医生推荐！"

# 3. 检测笔记 JSON 并给出替换建议
python3 scripts/check_banned_words.py examples/sample-food.json --suggest

# 4. 仅验证结构
python3 scripts/validate_note.py examples/sample-food.json

# 5. 生成标题建议 + 评分
python3 scripts/title_generator.py --topic "探店咖啡" --style casual --count 10 --score

# 6. 批量质量评分
python3 scripts/analytics.py examples/ --format table
```

---

## 完整流水线

```
选题关键词
    │
    ▼
collect_sources.py        # 抓取参考素材 / 搜索热词
    │
    ▼
note_lib.py               # 生成笔记 JSON（meta / sections / cover / cta）
    │
    ▼
banned_words_lib.py       # 自动违禁词扫描（嵌入 validate_note）
    │
    ▼
title_generator.py        # 生成候选标题 + CTR 评分
    │
    ▼
image_prompt_generator.py # 生成封面 & 内页图片 AI 提示词
    │
    ▼
render_note.py            # 渲染 HTML（选择模板）
    │
    ▼
validate_note.py          # 最终结构 + 违禁词双重校验
    │
    ▼
analytics.py              # 综合质量评分 + 改进建议
```

### run_pipeline.py 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--topic` | 笔记主题关键词（与 `--json` 二选一） | 必填 |
| `--json` | 已有笔记 JSON 文件路径（与 `--topic` 二选一） | — |
| `--template` | HTML 模板名称 | 根据 `--style` 自动选择 |
| `--style` | 写作风格 `casual/professional/cute/edgy` | `casual` |
| `--output` | 输出目录 | `./output` |
| `--no-images` | 跳过图片规划步骤 | `false` |
| `--force` | 有违禁词 ERROR 也继续运行 | `false` |
| `--strict` | 有 WARNING 也终止流程 | `false` |
| `--image-service` | 图片服务 `eachlabs/fal/openai/zhipu/generic` | `generic` |

### 风格 → 模板自动映射

| `--style` | 默认模板 |
|-----------|---------|
| `casual` | `lifestyle-review` |
| `professional` | `knowledge-share` |
| `cute` | `minimalist-card` |
| `edgy` | `outfit-inspo` |

### 输出文件

流水线运行后在 `--output` 目录生成：

| 文件 | 说明 |
|------|------|
| `note.json` | 笔记结构化数据 |
| `note.html` | HTML 预览文件 |
| `images-plan.json` | 配图提示词计划 |
| `pipeline-report.json` | 流水线报告（评分、违禁词、标题候选） |

---

## 笔记 JSON 格式

所有脚本围绕统一的 JSON 格式运作。以下是完整的字段定义。

### 顶层字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `template` | string | ✅ | HTML 模板名称，必须是 8 个模板之一 |
| `meta` | object | ✅ | 笔记元信息 |
| `cover` | object | — | 封面图配置 |
| `sections` | array | ✅ | 笔记正文段落数组（至少 1 个） |
| `cta` | string | — | 结尾行动号召文案（默认 `"你怎么看？评论区聊聊🙌"`） |

### meta 字段

| 字段 | 类型 | 必填 | 校验规则 | 说明 |
|------|------|------|---------|------|
| `title` | string | ✅ | **≤ 20 字**（超出报 error） | 笔记标题 |
| `tags` | array[string] | — | 建议 3-5 个，最多 10 个 | 话题标签 |
| `author` | string | — | 默认 `"39Claw"` | 作者名 |
| `date` | string | — | ISO 格式 `YYYY-MM-DD`，默认今天 | 发布日期 |
| `location` | string | — | — | 地点信息，如 `"上海·静安区"` |

### cover 字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `url` | string | 封面图 URL（有值才会渲染） |
| `caption` | string | 封面图说明文字 |
| `prompt` | string | AI 图片生成提示词 |

### 最小可用示例

```json
{
  "template": "lifestyle-review",
  "meta": {
    "title": "今日分享｜简单生活好物"
  },
  "sections": [
    {
      "type": "hook",
      "body": ["最近发现了几个超好用的生活好物，忍不住分享给大家！"]
    },
    {
      "type": "detail",
      "title": "好物推荐",
      "body": ["第一个是……", "第二个是……"]
    }
  ]
}
```

### 完整示例

```json
{
  "template": "food-explore",
  "meta": {
    "title": "上海宝藏火锅｜人均80吃到撑🔥",
    "tags": ["#上海美食", "#火锅推荐", "#探店"],
    "author": "39Claw",
    "date": "2026-03-24",
    "location": "上海·静安区"
  },
  "cover": {
    "prompt": "Steaming hot pot, red broth, close-up, food blog style, 3:4 vertical",
    "caption": "封面"
  },
  "sections": [
    {
      "type": "hook",
      "body": ["🔥 终于找到上海性价比最高的火锅！", "人均80，量大料足～"]
    },
    {
      "type": "detail",
      "title": "🍲 汤底 & 食材",
      "body": ["番茄牛骨汤底，自然酸甜。", "肥牛、毛肚必点🥩"],
      "image": {"url": "...", "caption": "菜品特写"}
    },
    {
      "type": "pros-cons",
      "pros": ["食材新鲜", "环境干净", "服务态度好"],
      "cons": ["晚高峰需等位", "停车位较少"]
    },
    {
      "type": "tips",
      "body": ["💡 工作日有优惠套餐", "📱 大众点评预约免排队"]
    },
    {
      "type": "rating",
      "items": [
        {"label": "口味", "score": "⭐⭐⭐⭐⭐"},
        {"label": "环境", "score": "⭐⭐⭐⭐"}
      ],
      "overall": "强烈推荐！"
    }
  ],
  "cta": "你们有什么宝藏火锅推荐吗？快来评论区分享🙏"
}
```

---

## Section 类型详解

`sections` 数组中的每个元素通过 `type` 字段指定渲染方式。共支持 **8 种类型**：

### hook — 开头钩子

> 前 3 行抓住注意力，决定用户是否继续阅读。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `type` | `"hook"` | ✅ | — |
| `body` | string 或 array[string] | ✅ | 钩子文案，建议 1-3 句 |

```json
{
  "type": "hook",
  "body": ["🔥 终于找到上海性价比最高的火锅！", "人均80，量大料足～"]
}
```

**校验提示**：`validate_note` 会检测是否存在 hook section，缺少时发出 warning。

---

### detail — 正文段落

> 笔记的核心内容段落，支持标题和内嵌配图。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `type` | `"detail"` | ✅ | — |
| `title` | string | — | 段落小标题（渲染为红色粗体） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [th3ee9ine/xiaohongshu-claw-skill](https://github.com/th3ee9ine/xiaohongshu-claw-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
