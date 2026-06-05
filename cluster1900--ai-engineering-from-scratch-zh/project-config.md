---
trigger: always_on
description: 供接触本 repo 的贡献者和 AI agents 使用的操作手册。打开 PR 前请先阅读。
---

# AGENTS.md

供接触本 repo 的贡献者和 AI agents 使用的操作手册。打开 PR 前请先阅读。

本 repo 是一套课程，而不是 SaaS app。课程就是产品。下面每条规则都是为了让 435 节课长期保持一致。

---

## 理念

435 节课。20 个 phase。每个算法都先从原始数学出发构建，然后才导入任何 framework。你会在 Python、TypeScript、Rust 或 Julia 中手写 backprop、Tokenizer、Attention 机制和 agent loop。然后你再用生产级 library 运行同一操作，让 framework 不再是黑箱。“Build It / Use It” 的分离是主线。每节课都会交付一个可复用 artifact，可接入你的日常工作流。

---

## Repo layout

```
phases/
  NN-phase-slug/
    NN-lesson-slug/
      docs/en.md              # 课程讲解
      code/                   # 实现 + tests
      quiz.json               # 6 个问题
      outputs/                # 可复用 artifact（skill / prompt / agent / MCP server）
README.md                     # 公开入口；课程数量自动同步
ROADMAP.md                    # phase/lesson 状态
glossary/terms.md             # 规范术语定义
site/
  build.js                    # 解析 README + ROADMAP + glossary -> data.js
  data.js                     # 生成文件；main push 时由 CI 重建
scripts/                      # 自动化
.github/workflows/
  curriculum.yml              # invariant + 自动同步 workflow
```

---

## 硬性规则

1. **每个 lesson directory 一个 commit。** 永远不要把多节课合并到一个 commit。一个 10-lesson PR 应该有 10 个 commits。
2. **Conventional commit subjects** ≤72 chars：`feat(phase-NN/MM): <slug>`。Body 解释为什么，而不是做了什么。
3. 图示**只允许 Mermaid 或 SVG**。不要使用 ASCII / Unicode box-drawing。
4. **每个 fenced code block 都需要 language tag。** 按需使用 `text`、`json`、`python`、`typescript`、`rust`、`julia`、`bash`、`console`、`mermaid`、`yaml`。
5. **只允许原创实现。** 不要在 docs、code comments 或 commit text 中引用外部 curriculum repos。当 RFCs、official specs 和 academic papers 是权威来源时，可以引用它们。
6. **Dependency allowlist**（见下方 `Dependencies`）。Stdlib-first。
7. **永远不要 commit 生成文件**：`catalog.json` 已 gitignored，`site/data.js` 由 CI 重建，`package-lock.json` 永不追踪。

---

## Dependencies

| Language   | Allowed                                                                  |
|------------|--------------------------------------------------------------------------|
| Python     | `numpy`, `torch`, `h5py`, `zstandard`, `safetensors`, stdlib              |
| TypeScript | `hono`, `zod`, `ws`（仅在需要 WebSockets 时）, `@hono/node-server`, Node 20+ stdlib |
| Rust       | 仅 stdlib（单文件 `rustc --edition 2021`）                          |
| Julia      | `Random`, `Statistics`, `LinearAlgebra`, `Printf`（Julia stdlib）          |

如果某个 finding 建议使用被禁止的 dep，请跳过，并给出原因：“stays stdlib-first for educational clarity.”

---

## Lesson contract

### docs/en.md frontmatter

```markdown
# <Title>

> <One-line hook>

**Type:** <Learn | Build | Reference>
**Languages:** <comma-list matching the main.* files in code/>
**Prerequisites:** <comma-list of upstream lessons, or "None">
**Time:** ~<estimate in minutes>

## Learning Objectives
- <4-6 bullet points starting with a verb>
```

`**Languages:**` 字段必须与 `code/` 中带有 `main.*` 文件的 languages 匹配。

### quiz.json schema

```json
{
  "lesson": "<dir-slug>",
  "title": "<Lesson Title>",
  "questions": [
    {"stage": "pre",   "question": "...", "options": ["a","b","c","d"], "correct": 0, "explanation": ""},
    {"stage": "check", "question": "...", "options": ["a","b","c","d"], "correct": 1, "explanation": ""},
    {"stage": "check", "question": "...", "options": ["a","b","c","d"], "correct": 2, "explanation": ""},
    {"stage": "check", "question": "...", "options": ["a","b","c","d"], "correct": 1, "explanation": ""},
    {"stage": "post",  "question": "...", "options": ["a","b","c","d"], "correct": 3, "explanation": ""},
    {"stage": "post",  "question": "...", "options": ["a","b","c","d"], "correct": 0, "explanation": ""}
  ]
}
```

必须正好 6 个问题：1 个 pre + 3 个 check + 2 个 post。`correct` 使用从零开始的索引。site renderer 只理解这种形状，旧版 `q/choices/answer` schemas 会静默崩溃。

### code/

- 使用该 language 的规范命令端到端运行，并以 0 退出。
- Demo 必须自行终止。不要有无限 stdin loops，也不要因为缺少 API keys 而 hang。
- 4-6 行 header comment，引用本 lesson 的 `docs/en.md` 路径，以及任何 spec 或 RFC 来源。

### code/tests/

- 至少 5 个 unit tests。
- 通过该 language 的 stdlib runner 运行（`python3 -m unittest discover`、`npx tsx --test`，Rust/Julia inline）。

---

## Per-PR validation

Push 前在本地运行：

```bash
python3 scripts/audit_lessons.py
python3 scripts/check_readme_counts.py        # advisory — CI fixes on merge

# For each lesson touched:
cd phases/NN-phase/MM-lesson/code
python3 main.py && python3 -m unittest discover tests -v   # or the lang equivalent
```

CI gates（`.github/workflows/curriculum.yml`）：

| Job                              | Trigger      | Behavior                                              |
|----------------------------------|--------------|-------------------------------------------------------|
| `audit`                          | push + PR    | 运行 `audit_lessons.py`。Blocking。                    |
| `readme-counts-sync`（仅 main） | push to main | 重建 catalog + 自动修复 README counts。         |
| `site-rebuild`（仅 main）       | push to main | 重新运行 `node site/build.js`，commit `site/data.js`。 |
| `readme-counts-drift`            | PR           | 仅 advisory，main 会在 merge 后自我修复。             |

---

## Automation contract

**CI 自动处理，请不要在你的 PR 中触碰：**

| Surface              | Bot                            | When                |
|----------------------|--------------------------------|---------------------|
| `catalog.json`       | 按需重建（gitignored） | 每个 CI job        |
| `README.md` counts   | `readme-counts-sync`           | push to main 时     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cluster1900/ai-engineering-from-scratch-zh](https://github.com/cluster1900/ai-engineering-from-scratch-zh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
