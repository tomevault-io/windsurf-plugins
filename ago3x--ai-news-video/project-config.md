---
trigger: always_on
description: |
---


# AI新闻短视频生成器

## 架构分工

```
┌─────────────────────────────────────┐
│            Agent 负责               │
│  1. 搜索新闻 / 理解主题              │
│  2. 撰写详细文案（article.md）       │
│  3. 据文案生成口播脚本（script.json） │
│  4. 生成抖音/小红书标题与发布文案     │
│  5. 根据口播编写 N 个 HTML 页面      │
└──────────────┬──────────────────────┘
               │ 详细文案 + 口播文稿 + HTML
               ▼
┌─────────────────────────────────────┐
│         Python 脚本负责              │
│  6. TTS生成配音                     │
│  7. 按静音点切分配音                 │
│  8. Playwright录制HTML              │
│  9. 音视频合并 + 拼接输出            │
│ 10. README 写入详细文案与社媒文案     │
└─────────────────────────────────────┘
```

---

## Agent 工作流程

### Step 1：搜索/理解新闻

用搜索工具获取该主题的最新信息，整理出：
- 核心事件（一句话）
- 关键数据（数字、价格、参数等）
- 主要特性/变化（3~4条）
- 对比信息（新旧/竞品对比）
- 意义/影响

> 也可直接调用辅助脚本批量抓取：`python scripts/search_news.py "搜索关键词"`

---

### Step 1.5：图片采集（搜索完成后立即执行）

在写 `article.md` 之前，对主要信息来源页面执行图片采集，将素材保存到 `output/{slug}/images/`。

#### 采集方式

**方式一：全页截图**

用 Playwright 对目标 URL（新闻原文、GitHub 仓库主页等）截取全屏快照，保存为 `images/page-screenshot.png`：

```python
from playwright.sync_api import sync_playwright
from pathlib import Path

def capture_page(url: str, out_path: Path):
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page(viewport={"width": 1280, "height": 800})
        page.goto(url, wait_until="networkidle", timeout=30000)
        page.screenshot(path=str(out_path), full_page=False)
        browser.close()

# 保存为：output/{slug}/images/page-screenshot.png
```

**方式二：提取页面图片**

从页面中筛选有实质内容的 `<img>`，过滤掉图标（宽高 < 100px）、SVG、data URI，下载前 3～5 张到 `images/img-001.jpg` 等：

```python
import urllib.request
from pathlib import Path

def extract_images(page, out_dir: Path, max_count: int = 5):
    imgs = page.query_selector_all('img')
    saved = 0
    for img in imgs:
        src = img.get_attribute('src') or ''
        w = int(img.get_attribute('width') or 0)
        h = int(img.get_attribute('height') or 0)
        if src.startswith('data:') or src.endswith('.svg'):
            continue
        if w < 100 or h < 100:
            continue
        ext = 'jpg' if ('jpg' in src or 'jpeg' in src) else 'png'
        dest = out_dir / f"img-{saved+1:03d}.{ext}"
        try:
            urllib.request.urlretrieve(src, dest)
            saved += 1
        except Exception:
            continue
        if saved >= max_count:
            break
```

#### 失败处理

- 若目标页面截图失败（超时、反爬、登录墙），跳过截图，不中断后续流程
- 若图片提取为 0，后续 HTML 改用纯文字模板，不使用 `image-caption` / `split-image`
- 采集失败时在日志中注明：`[IMAGES] 采集失败，降级为纯文字模板`

#### 输出结构

```
output/{slug}/images/
  ├── page-screenshot.png    # 全页截图（可选）
  ├── img-001.jpg            # 提取图片 1
  ├── img-002.jpg            # 提取图片 2
  └── ...
```

---

### Step 2：撰写详细文案（先于口播）

在写口播、定镜头之前，先把素材写成**一篇完整、可独立阅读的长文案**，保存为：

`output/{slug}/article.md`

**目的**：把搜索到的信息沉淀为结构化正文，作为口播与画面的唯一事实来源，避免口播与详述脱节。

**建议结构**（可按题材增删）：

| 区块 | 内容 |
|------|------|
| 标题 | 一句话概括事件 |
| 导语 | 2~4 句：谁、做了什么、为何重要 |
| 背景 | 必要的前情与行业语境 |
| 核心信息 | 分点列出：功能、数据、价格、时间等（带出处或「据公开信息」） |
| 对比 / 影响 | 与竞品、旧版或市场预期的差异 |
| 小结 | 对普通观众的一句话 takeaway |

**写作要求**：

- 书面语为主，信息密度高于口播；数字、专有名词写全，便于后续压缩成口播句。
- 不写分镜、不写「镜头1/镜头2」——本步只做「稿」，不做「演」。
- 篇幅建议 **400～900 字**（题材复杂可到 1200 字），过短则口播会缺料。
- 若不便单独维护文件，可将同等正文放入 `script.json` 的 `detailedCopy` 字段（Markdown 字符串）；**二者至少其一**，且以 `article.md` 为准（同时存在时 README 优先收录文件）。

#### 视觉主题选择

写完 `article.md` 后，根据文案的**主题性质、情感基调和目标受众**，自主判断最合适的视觉主题，写入 `script.json` 的 `style` 字段。

| 主题值 | 适合的内容感觉 |
|--------|--------------|
| `apple` | 科技感、产品发布、冷静专业、AI/大模型类；默认选项 |
| `cyber` | 极客/黑客感、开源社区、GitHub项目、编程工具、赛博朋克调性 |
| `media` | 严肃资讯、财经市场、政策法规、社会事件、有温度的新闻报道 |
| `light` | 轻松生活、消费测评、小红书风格、面向大众的科普内容 |

**判断方式**：不做关键词匹配，基于对文案整体调性的理解作出判断。用户可在命令行加 `--style <主题>` 覆盖此选择。

---

### Step 3：根据详细文案生成口播文稿

**输入**：`article.md`（及 Step 1 的要点笔记）。**禁止**跳过详细文案直接写口播。

根据详细文案决定镜头数量，以「每段讲清一个点、节奏不拖沓」为准；口播中的事实、数字必须与 `article.md` 一致，不可编造。

---

#### 第一步：先定镜头数量

在写任何文案之前，先用以下决策树确定本次需要几个镜头：

```
新闻信息量评估
│
├── 只有 1 个核心事件，无需对比、无复杂背景
│   → 3 个镜头（钩子 + 核心信息 + 共鸣）
│
├── 有 1~2 个核心亮点 + 简单背景或价格信息
│   → 4 个镜头（钩子 + 亮点 + 数据/价格 + 共鸣）
│
├── 有 3+ 个维度（功能 + 对比 + 背景 + 影响）
│   → 5 个镜头（钩子 + 背景 + 亮点 + 对比 + 共鸣）
│
└── 深度报道：有时间线、多方角色、复杂影响
    → 6 个镜头（可拆分亮点或背景为 2 个镜头）
```

**硬性约束**：
- **最少 3 个**：钩子 + 至少一个信息镜头 + 共鸣，再少就撑不起节奏
- **最多 6 个**：单条视频总时长不超过 60 秒，超过观众会划走
- **不要凑数**：内容只够 3 个镜头，绝不硬拆成 4 个；内容可以讲 5 个点，不要压缩成 3 个

---

#### ⚠️ 文案核心原则：讲故事，不写说明书

**禁止** 这样写（产品说明书风格）：
> "GPT-5带来四大升级：推理能力提升10倍、多模态全面增强、响应速度快3倍、上下文扩展到100万token！"

**应该** 这样写（有钩子、有冲突、有情绪）：
> "等了整整一年，发布会结束那一刻，所有人都沉默了——不是因为太震撼，是因为看到了那个价格。"

两者信息量相同，但后者让人停住、想看下去。

**对照组 2 — 财经资讯**

> ❌ 坏（数据罗列）：A股今日三大指数全线下跌，上证综指跌0.8%，深证成指跌1.2%，创业板指跌1.5%，成交量较昨日萎缩。
>
> ✅ 好（情绪带入）：今天开盘，很多人的手还没到键盘上，账户就已经绿了——而且是那种越看越绿的绿。

**对照组 3 — GitHub项目发布**

> ❌ 坏（功能列举）：该项目支持多模态输入、RAG检索增强、工具调用、流式输出，star数已达10k，支持Docker一键部署。
>
> ✅ 好（场景代入）：昨晚，一个 GitHub 项目的 star 数，在 12 小时内从两千涨到了一万——没有发布会，没有公司背书，只有一个人写的 README。

---

#### 故事骨架（四个叙事环节，≠ 四个镜头）

> ⚠️ 这里的①②③④是**叙事功能**，不是镜头数量。3 个镜头的视频同样覆盖四个环节，中间两个环节可以合并在一个镜头里讲。

| 叙事环节 | 功能 | 镜头映射规则 |
|---------|------|------------|
| **① 钩子** | 制造悬念 / 反差 / 震惊，让观众停住 | 永远是第 1 个镜头，独占一个镜头 |
| **② 冲突** | 揭示矛盾、落差、意外，让观众代入 | 简单新闻可并入③；内容足够时独立一个镜头 |
| **③ 爆料** | 核心数据 / 事实集中引爆 | 数据多时可拆成 2 个镜头（功能 + 价格），数据少时与②合并 |
| **④ 共鸣** | 与观众产生连接，引导互动 | 永远是最后一个镜头，独占一个镜头 |

**示例映射**：
- 3 镜头：① → ②③合并 → ④
- 4 镜头：① → ② → ③ → ④
- 5 镜头：① → ② → ③功能 → ③价格 → ④
- 6 镜头：① → ②背景 → ③功能 → ③数据 → ③对比 → ④


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ago3x/ai-news-video](https://github.com/ago3x/ai-news-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
