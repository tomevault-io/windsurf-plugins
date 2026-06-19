---
trigger: always_on
description: Transform academic papers into conversational Chinese articles in Qiaomu's style. Use when user provides arXiv URL/ID with keywords "解读论文", "论文解读", "理解paper", "读paper", or "乔木风格". Runs fully automatically.
---


# 乔木论文解读

## 概述

将学术论文自动转化为乔木风格的深度解读文章。**全自动执行**，无需用户中途确认。

**核心特点**：
- 对话式语言，像和朋友聊天
- 关键术语用引用块（>）解释，每出现新术语立刻加
- 生活化类比帮助理解，每个核心方法后必须紧跟一个类比
- 真实论文图表嵌入文章（从 LaTeX 源码精确提取）
- AI 生成纸雕水彩封面 + 《纽约客》风格配图（需配置图片生成服务）
- 写作风格规范已内嵌，无需外部依赖

---

## 配置（首次使用必读）

### 配置方式：.env 文件

在 skill 目录下创建 `.env` 文件（已加入 `.gitignore`，不会被发布）：

```bash
# ~/.claude/skills/qiaomu-paper-interpreter/.env

PAPER_OUTPUT_DIR=~/Papers/papers
PAPER_READING_DIR=~/Papers/reading
OBSIDIAN_VAULT=
IMAGE_PROVIDER=skip
IMAGE_GENERATOR_SCRIPT=
```

**参考模板**：skill 目录下的 `.env.example` 包含所有可用变量及说明。

**变量说明**：

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `PAPER_OUTPUT_DIR` | `~/Papers/papers` | 论文工作目录根路径 |
| `PAPER_READING_DIR` | `~/Papers/reading` | 最终文章存放目录 |
| `OBSIDIAN_VAULT` | 空 | Obsidian vault 名称，空则跳过自动打开 |
| `IMAGE_PROVIDER` | `skip` | `skip`（跳过配图）/ `jimeng` / `openai` |
| `IMAGE_GENERATOR_SCRIPT` | 空 | 图片生成脚本路径，空则用内置默认 |

系统环境变量优先级高于 `.env` 文件，适合 CI/CD 或多项目场景。

### 配置读取逻辑（每次执行时运行）

```python
import os
from pathlib import Path

SKILL_DIR = Path("~/.agents/skills/qiaomu-paper-interpreter").expanduser()

# 1. 读取 .env 文件（如果存在）
env_file = SKILL_DIR / ".env"
if env_file.exists():
    for line in env_file.read_text().splitlines():
        line = line.strip()
        if line and not line.startswith("#") and "=" in line:
            k, v = line.split("=", 1)
            # 系统环境变量优先，.env 不覆盖已设置的值
            os.environ.setdefault(k.strip(), v.strip())

# 2. 读取最终配置（含默认值兜底）
OUTPUT_DIR    = Path(os.environ.get("PAPER_OUTPUT_DIR",    "~/Papers/papers")).expanduser()
READING_DIR   = Path(os.environ.get("PAPER_READING_DIR",   "~/Papers/reading")).expanduser()
OBSIDIAN_VAULT = os.environ.get("OBSIDIAN_VAULT", "")
IMAGE_PROVIDER = os.environ.get("IMAGE_PROVIDER", "skip")   # skip | jimeng | openai
IMAGE_GENERATOR_SCRIPT = os.environ.get("IMAGE_GENERATOR_SCRIPT", "")

# 图片生成脚本默认路径
if not IMAGE_GENERATOR_SCRIPT and IMAGE_PROVIDER != "skip":
    IMAGE_GENERATOR_SCRIPT = str(
        Path("~/.agents/skills/qiaomu-image-generator/scripts/generate.py").expanduser()
    )
```

---

## 执行流程（4步）

**顺序**：步骤A → 步骤B → 步骤C → 步骤D

**执行原则**：全程自动，用 TodoWrite 显示进度，静默修复质量问题。

### 初始化 Todo

```python
TodoWrite([
    {"content": "A. 提取论文内容 + 并发转换图片", "status": "in_progress"},
    {"content": "B. 生成乔木风格解读文章", "status": "pending"},
    {"content": "C. 生成 AI 配图（封面 + 纽约客）", "status": "pending"},
    {"content": "D. 保存发布", "status": "pending"},
])
```

---

## 步骤A：提取论文内容 + 并发转换图片

**目标**：一次完成 LaTeX 提取、元数据解析、图片并发转换、图表清单生成。

### A1. 确定 arxiv_id

支持输入格式：

| 输入 | 处理方式 |
|------|---------|
| `https://arxiv.org/abs/2605.03269` | 直接提取 ID |
| `https://arxiv.org/pdf/2605.03269` | 直接提取 ID（和 abs 等价） |
| `https://arxiv.org/pdf/2605.03269v2` | 提取 ID，保留版本号 |
| `https://huggingface.co/papers/2605.03269` | 先 WebFetch 页面找 arXiv 链接，再提取 ID |
| `2605.03269` | 直接作为 ID 使用 |

**内部流程**：extract_tex.py 拿到 ID 后，从 `https://arxiv.org/e-print/{id}` 下载 LaTeX 源码 tar.gz，自动解压，找 main.tex，提取结构化内容和图片。

**如果论文没有 LaTeX 源码（PDF-only）**：
`e-print` 返回 PDF 而不是 tar.gz，extract_tex.py 会返回 `has_source: false`。此时自动切换到 markitdown fallback：
```bash
markitdown "https://arxiv.org/pdf/{arxiv_id}" -o "{paper_dir}/extracted_text.md"
```
fallback 情况下无法提取真实图表，figure_list.md 标注"LaTeX 源码不可用"，文章中用文字描述代替图片引用。

### A2. 并发启动：extract_tex.py + arXiv API（含断点续跑）

**断点续跑**：如果 `extract_result.json` 已存在且有效，直接跳过下载，从 A3 继续。适用于崩溃重试场景。

```python
import subprocess, threading, urllib.request, re, time, json
from pathlib import Path

# ── 断点续跑检测 ──
# 检查是否有已完成的 paper_id 目录（paper_id 此时尚未确定，用 arxiv_id 查找）
existing = list(OUTPUT_DIR.glob(f"*{arxiv_id.replace('.', '_')}*"))
_resume_dir = next((d for d in existing
                    if (d / "extract_result.json").exists()
                    and (d / "extract_result.json").stat().st_size > 100), None)

if _resume_dir:
    paper_dir = _resume_dir
    result_file = paper_dir / "extract_result.json"
    print(f"⚡ 断点续跑：跳过下载，使用 {paper_dir.name}")
    arxiv_meta = {}   # 仍需 API 获取日期，下面会并发请求
    _skip_extract = True
else:
    paper_dir = OUTPUT_DIR / f"tmp_{int(time.time())}"
    paper_dir.mkdir(parents=True, exist_ok=True)
    result_file = paper_dir / "extract_result.json"
    _skip_extract = False

# ── 线程1：并发请求 arXiv API（⚠️ 发布日期严禁猜测，只用此处返回值）──
arxiv_meta = {}
def fetch_arxiv_meta():
    try:
        xml = urllib.request.urlopen(
            f"http://export.arxiv.org/api/query?id_list={arxiv_id}", timeout=10
        ).read().decode()
        m = re.search(r'<published>(.*?)</published>', xml)
        arxiv_meta["published_date"] = m.group(1)[:10] if m else ""
        # ⚠️ 必须从 <entry> 内提取，xml 第一个 <title> 是 feed 标题，不是论文标题
        entries = re.findall(r'<entry>(.*?)</entry>', xml, re.DOTALL)
        entry = entries[0] if entries else ""
        t = re.search(r'<title>(.*?)</title>', entry, re.DOTALL)
        arxiv_meta["title"] = t.group(1).strip().replace('\n', ' ') if t else ""
        arxiv_meta["authors"] = re.findall(r'<name>(.*?)</name>', entry)
    except Exception:
        arxiv_meta["published_date"] = ""

meta_thread = threading.Thread(target=fetch_arxiv_meta, daemon=True)
meta_thread.start()

# ── 线程2（主线程）：运行 extract_tex.py，120s 超时 ──
if not _skip_extract:
    try:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joeseesun/qiaomu-paper-interpreter](https://github.com/joeseesun/qiaomu-paper-interpreter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
