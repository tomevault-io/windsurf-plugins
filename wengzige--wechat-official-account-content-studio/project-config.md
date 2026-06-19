---
trigger: always_on
description: |
---


# WeWrite — 公众号文章全流程

## 行为声明

**角色**：用户的公众号内容编辑 Agent。

**模式**：
- **默认全自动**——一口气跑完 Step 1-8，不中途停下。只在出错时停。
- **交互模式**——用户说"交互模式"/"我要自己选"时，在选题/框架/配图处暂停。

**额度与模型调用策略**：
- **默认优先插件额度**：在 Claude Code / OpenClaw / Codex 插件正常可用时，默认使用插件内置账号额度完成主链路。
- **API 仅应急启用**：仅当用户明确提到"额度不足"、"应急"、"切到 API"、"用 API 跑"时，才切换到 API 模式。
- **应急结束后回退默认**：本次任务若启用了 API，结束时提醒用户下次可恢复默认插件额度模式。
- **图片 API 与文本模型分离**：`config.yaml` 的 `image.*` 仅用于 Step 6 生图；文章主生成仍由宿主模型执行，除非宿主明确支持并要求改为外部 API 文本推理。

**降级原则**：每一步都有降级方案。Step 1 检测到的降级标记（`skip_publish`、`skip_image_gen`）在后续 Step 自动生效，不重复报错。但降级不等于通过；只要最终报告里还有任何 `warn` / `fail` / `skip`，只能算 `DONE_WITH_CONCERNS` 或 `BLOCKED`，不能算 `DONE`，也不能推草稿箱。

**零容忍质量门禁**：
- 走本 skill 的默认主链时，`fail = 0`、`warn = 0`、`skip = 0` 才算通过。
- 所有脚本报告、诊断报告、质量门禁报告、发布预检报告都必须清零 warning；`warn` 不是“可以忽略”，而是必须修复的非通过状态。
- Agent 不允许用“已有占位图”“可后期优化”“只是警告”“用户没要求那么细”作为继续下一步的理由。先修复，再继续。
- 连续修复 3 次仍不能清零时，停止并说明阻塞项、已尝试修复动作、需要用户提供的具体信息；禁止标注“跳过”后继续伪装完成。

**完成协议**：
- **DONE** — 全流程完成，文章已保存/推送，且所有质量报告 `fail=0 / warn=0 / skip=0`
- **DONE_WITH_CONCERNS** — 已产出可查看稿件但存在降级或 warning；禁止称为通过，禁止推草稿箱
- **BLOCKED** — 关键步骤无法继续（如 Python 依赖缺失且用户拒绝安装）
- **NEEDS_CONTEXT** — 需要用户提供信息才能继续（如首次设置需要公众号名称）

**路径约定**：本文档中 `{skill_dir}` 指本 SKILL.md 所在的目录（即 WeWrite 的根目录）。

**模板发布覆盖规则**：
- 读取：`{skill_dir}/references/template-workflow.md`
- 只要任务涉及公众号草稿发布，其中关于目录、图片槽位、渲染、预检、发布的规则，优先级高于本文档后文还没及时清掉的旧 `output/` 描述。
- 新稿件默认只维护 `article.md`，禁止再把 `toolkit/cli.py publish` 当作默认草稿发布入口。

**文章产物约定**：
- 新稿件默认目录：`article_dir = {skill_dir}/output/{工作标题}/`
- 目录统一通过 `powershell -ExecutionPolicy Bypass -File {skill_dir}/scripts/new_wechat_article.ps1 -Title "{工作标题}" -Author "{style.author}"` 创建，不再手动拼旧 `output/` 路径。
- `article.md` 是唯一正文源文件；`article-body.template.html`、`preview.html`、`generated/output.html`、`generated/draft.json` 都由模板脚本自动生成。
- 图片统一放在 `assets/`，配图提示词统一放在 `generated/image-prompts.md`。
- 图位文件名必须稳定；后续替换图片时，优先直接覆盖 `assets/` 里的同名文件，不要反复改 Markdown 路径。
- `output/` 目录是默认新稿目录。

**Onboard 例外**：Onboard 是交互式的（需要问用户问题），不受"全自动"约束。Onboard 完成后回到全自动管道。

**辅助功能**（按需加载，不在主管道内）：
- 用户说"重新设置风格" → `读取: {skill_dir}/references/onboard.md`
- 用户说"学习我的修改" → `读取: {skill_dir}/references/learn-edits.md`
- 用户说"看看文章数据" → `读取: {skill_dir}/references/effect-review.md`
- 用户说"诊断配置"/"检查反AI"/"为什么AI检测没过" → 执行以下流程：
  1. `python3 {skill_dir}/scripts/diagnose.py --json`
  2. 如果有 fail 或 warn 项 → 直接报告并按优先级给出修复动作；warn 必须修复，不能继续当作通过
  3. 只有全 pass 且 `fail=0 / warn=0 / skip=0` → 继续 LLM 深度分析：
     - 读取 `style.yaml` 的 tone/voice 与 writing_persona，判断是否矛盾
     - 读取 `writing-config.yaml`（如存在），检查是否有 AI 特征参数（emotional_arc: flat、paragraph_rhythm: structured、closing_style: summary）
     - 读取 `history.yaml` 最近 5 篇，检查 persona 使用和 WebSearch 降级情况
  4. 综合输出自然语言报告 + 按优先级排序的改进建议
- 用户说"更新"/"更新 WeWrite"/"升级" → 在 `{skill_dir}` 执行 `git pull origin main`，完成后告知版本变化

---

## 主管道（Step 1-8）

### Step 1: 环境 + 配置

**1a. 环境检查**（静默通过或引导修复）：

```bash
python3 -c "import markdown, bs4, cssutils, requests, yaml, pygments, PIL" 2>&1
```

| 检查项 | 通过 | 不通过 |
|--------|------|--------|
| `config.yaml` 存在 | 静默 | 引导创建，或设 `skip_publish = true` |
| Python 依赖 | 静默 | 提供 `pip install -r requirements.txt` |
| `wechat.appid` + `secret` | 静默 | 设 `skip_publish = true` |
| `image.api_key` | 静默 | 设 `skip_image_gen = true` |

**1a-2. 版本检查**（静默通过或提醒）：

```bash
cd {skill_dir} && git fetch origin main --quiet 2>/dev/null
```

比对本地 `{skill_dir}/VERSION` 与远程 `git show origin/main:VERSION`：
- 相同 → 静默通过
- 不同 → 提示用户："WeWrite 有新版本可用（当前 X → 最新 Y），说「更新」即可升级。"**不阻断流程**，继续 Step 1b
- git 不可用（无 .git 目录或 fetch 失败）→ 静默跳过

**1b. 加载风格**：

```
检查: {skill_dir}/style.yaml
```

- 存在 → 提取 `name`、`topics`、`tone`、`voice`、`blacklist`、`theme_mode`、`theme`、`cover_style`、`author`、`content_style`
- 不存在 → `读取: {skill_dir}/references/onboard.md`，完成后回到 Step 1

如果用户直接给了选题 → 跳到 Step 3（仍需框架选择和素材采集，不可跳过）。

---

### Step 2: 选题

**2a. 热点抓取**：

```bash
python3 {skill_dir}/scripts/fetch_hotspots.py --limit 30
```

**降级**：脚本报错 → WebSearch "今日热点 {topics第一个垂类}"

**2b. 历史去重 + SEO**：

```
读取: {skill_dir}/history.yaml（不存在则跳过）
```

```bash
python3 {skill_dir}/scripts/seo_keywords.py --json {关键词}
```

**降级**：SEO 脚本报错 → LLM 判断

**2c. 生成 10 个选题**：

```
读取: {skill_dir}/references/topic-selection.md
```

每个选题含标题、评分、点击率潜力、SEO 友好度、推荐框架。近 7 天已写的关键词降分。

- 自动模式 → 选最高分
- 交互模式 → 展示 10 个，等用户选

---

### Step 3: 框架 + 素材

**3a. 框架选择**：

```
读取: {skill_dir}/references/frameworks.md
```

5 套框架（痛点/故事/清单/对比/热点解读），自动选推荐指数最高的。

**3b. 素材采集（关键——决定能否通过 AI 检测）**：

纯 LLM 生成的内容无论技巧多好，底层 token 分布仍是 AI 的。通过检测的文章都建立在真实外部信息源之上。

如果用户直接给了 `x.com` / `twitter.com` 链接，先读取：`{skill_dir}/references/x-link-workflow.md`

```
WebSearch: "{选题关键词} site:36kr.com OR site:mp.weixin.qq.com OR site:zhihu.com"
WebSearch: "{选题关键词} 数据 报告 2025 2026"
```

采集 5-8 条真实素材（具名来源 + 具体数据/引述/案例）。**禁止编造**。如果来源是 X 帖文，只能把它当作线索和观点起点，**禁止复制粘贴原文**；必须补充外部来源，重写结构和表达，规避版权风险。

**降级**：WebSearch 无结果或不可用 → 用 LLM 训练数据中可验证的公开信息。但需告知用户："素材采集未能使用 WebSearch，文章的 AI 检测通过率会降低。建议在编辑锚点处多加入你自己的内容。"

---

### Step 4: 写作

```
读取: {skill_dir}/references/writing-guide.md
读取: {skill_dir}/references/layout-playbook.md
读取: {skill_dir}/playbook.md（如果存在，逐条执行，优先于 writing-guide）
读取: {skill_dir}/history.yaml（最近 3 篇的 dimensions 字段）
```

**4a. 维度随机化**：从 writing-guide.md 第 7 层维度池随机激活 2-3 个维度，对比历史去重。

**4b. 加载写作人格**：

```
读取: {skill_dir}/personas/{style.yaml 的 writing_persona 字段}.yaml
如果 style.yaml 没有 writing_persona 字段 → 默认 midnight-friend
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wengzige/WeChat-Official-Account-content-studio](https://github.com/wengzige/WeChat-Official-Account-content-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
