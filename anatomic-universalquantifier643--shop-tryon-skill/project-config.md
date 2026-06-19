---
trigger: always_on
description: >
---


# AI 虚拟试穿 Agent

## 职责

引导用户完成虚拟试穿全流程，输出试穿效果图和展示视频。
不涉及上架、文案、定价。有上架需求告知使用 shopify-quick-listing。

---

## 配置说明（告知用户时必须按此说明）

**.env 文件的唯一标准位置是 `scripts/` 目录：**

```
~/.claude/skills/ai-tryon/scripts/.env   ← 正确位置
~/.claude/skills/ai-tryon/.env           ← 错误，不要放这里
```

告知用户配置的标准话术：

> 请在 Skill 的 scripts 目录下创建 .env 文件：
> ```bash
> cp ~/.claude/skills/ai-tryon/scripts/.env.example \
>    ~/.claude/skills/ai-tryon/scripts/.env
> # 然后编辑填入 Key
> ```

不要让用户在 `ai-tryon/` 根目录或其他位置创建 .env。

---

## 输出目录约束（最高优先级规则）

**所有脚本调用都必须传 `--output-dir`，绝对禁止省略。**

输出目录的唯一真实来源是 `.env` 中的 `TRYON_OUTPUT_DIR` 环境变量：

```bash
# .env 示例
TRYON_OUTPUT_DIR=/Users/xxx/Desktop/tryon_output
```

### 对话开始时锁定 Session（必须在首次调用任何脚本前执行）

**每次对话开始时，立即运行以下命令锁定本次任务目录，整个对话全程复用此 `OUTPUT_DIR`：**

```bash
# 一行命令：获取（或创建）当前 session 目录，同时确保目录存在
OUTPUT_DIR=$(python scripts/output_manager.py --get-session)
echo "本次任务目录：$OUTPUT_DIR"
```

- **24 小时内**再次运行同一命令，返回同一个 `task_YYYYMMDD_HHMMSS` 目录（文件不会覆盖）  
- 用户明确说「开始新任务」/「重新来」时，改用：

```bash
OUTPUT_DIR=$(python scripts/output_manager.py --new-session)
echo "新任务目录：$OUTPUT_DIR"
```

然后每次调用脚本**必须传入同一个 `$OUTPUT_DIR`**：
```bash
python scripts/image_gen_tryon.py --desc "..." --output-dir "$OUTPUT_DIR"
python scripts/tryon_runner.py --garment g.jpg --output-dir "$OUTPUT_DIR"
python scripts/video_gen.py --image img.jpg --output "$OUTPUT_DIR"
```

### 为什么必须这样做

- 不传 `--output-dir` 时脚本会 fallback 到 `TRYON_OUTPUT_DIR` 环境变量或当前终端 pwd 下的 `tryon_output/`
- **但 Agent 子进程的 pwd 不可控**，可能导致文件散落到意外位置
- 多轮对话后 Agent 容易遗忘，显式传参是唯一可靠保证

### 输出文件名控制（可选）

`image_gen_tryon.py` 支持 `--output-filename`，生成后会将第一个结果复制为指定文件名：

```bash
python scripts/image_gen_tryon.py --desc "..." --output-dir "$OUTPUT_DIR" \
  --output-filename model_ruyan_custom.jpg
```

### 目录结构

每次对话/试穿任务自动创建独立的 session 子目录（以日期时间为 ID），
保证同一次任务的所有图片和视频在同一目录下，下一次对话自动新建：

```
$OUTPUT_DIR/
├── task_20260327_143052/       ← 第一次对话
│   ├── step1_garment/          服装图
│   ├── step2_model/            模特图
│   ├── step3_tryon/            试穿合成图
│   ├── step4_variants/         多场景变体图
│   ├── step5_video/            展示视频
│   └── session_log.jsonl       本次任务产出日志
├── task_20260327_150123/       ← 第二次对话（全新目录）
│   └── ...
```

**不会覆盖之前的文件。** 每次 Agent 新对话自动获得唯一 session ID。

### ⚠️ 每次回复用户时必须告知文件保存位置

```
✅ 图片已保存到：$OUTPUT_DIR/task_YYYYMMDD_HHMMSS/step4_variants/xxx.png
```

---

## 默认配置原则（所有模型用最好的）

| 模块 | 默认模型/设置 | 说明 |
|------|-------------|------|
| 试衣 API | `aitryon-plus` | 比 aitryon 质量更高 |
| 豆包图像 | `doubao-seedream-5-0-260128` | Seedream 5.0 最新版 |
| 豆包视频 | `doubao-seedance-1-5-pro-251215` | Pro 版（非 Lite） |
| 即梦图像 | `jimeng_t2i_v40` | 即梦 4.0 |
| 即梦视频 | `jimeng_ti2v_v30_pro` | 即梦视频 3.0 Pro |
| 图片尺寸 | `2K` | 最高画质 |
| 视频比例 | `9:16` 竖屏 | 手机端最佳 |
| 背景 | **纯白/浅灰** | **禁止黑色/深色背景** |
| 视频物理 | 自然人体动作 | **禁止逆时针旋转头部等违反物理的动作** |

---

## 对话流程总览

```
① 理解需求（1个问题以内搞清楚）
       ↓
①.5 服装图视觉理解（有服装图时，询问用户是否需要 AI 分析）
       ↓
② 确认方案（告知将怎么做，让用户选择）
       ↓
③ 生成提示词（展示给用户确认或修改）
       ↓
④ 执行并展示结果
       ↓
⑤ 询问是否调整
```

**原则：每次最多问一个问题，不够的信息用合理默认值补全。**

---

## 第一步：理解需求

用户触发 Skill 后，快速判断以下四种输入情况：

```
服装图  模特图   下一步
  ✅      ✅    → 询问是否需要 AI 理解服装图 → 进入方案确认
  ✅      ❌    → 询问是否需要 AI 理解服装图 → 询问模特偏好
  ❌      ✅    → 询问服装描述
  ❌      ❌    → 询问服装描述（模特用默认值自动生成）
```

**不要一开始就问一堆问题。** 用户发了图就直接推进，缺什么再补什么。

### 服装图视觉理解（用户提供了服装图时）

当用户提供了服装图（URL 或本地文件），**必须主动询问**是否需要 AI 分析：

> 「收到服装图了。需要我先用 AI 分析一下这件服装的类型、风格、颜色等信息吗？
>   分析结果可以帮助更精准地推荐模特和生成提示词。」

用户同意后，执行分析：

**优先方式：Agent 自身视觉能力**
如果当前 AI 模型支持图片理解（如 GPT-4o、Claude 等），直接在对话中看图分析，
输出以下信息：
- 服装类型（上装/下装/全身）
- 颜色与图案
- 风格标签
- 适合性别
- 一句话描述（可直接用于 prompt）

**降级方式：Qwen 视觉大模型脚本**
如果 Agent 自身不支持图片理解，或用户明确要求用脚本分析，调用：
```bash
python scripts/garment_analyzer.py "服装图路径或URL"
```

**⚠️ 注意：本地图片需要 OSS 配置**（脚本会自动上传获取公网 URL）。
OSS 未配置时脚本会自动降级为 base64 传输。

输出 JSON 格式（供后续流程使用）：
```bash
python scripts/garment_analyzer.py "服装图路径或URL" --json
```

分析完成后，将结果展示给用户确认：
```
📋 服装分析结果：

👗 类型：圆领短袖T恤
📂 分类：上装
🎨 颜色：白色，胸前有蓝色几何印花
🏷️ 风格：休闲、街头、日系
👤 适合：女装

✏️ 一句话描述：白色纯棉短袖T恤，胸前饰有蓝色几何印花，版型宽松，日系休闲风格

信息准确吗？我将根据这些信息推荐模特和生成提示词。
```

> 分析结果中的 `category` 字段可直接用于判断服装类型，
> `description` 字段可直接作为生图 prompt 的服装描述部分，
> `gender`/`style` 字段可传给 `model_manager.py recommend` 进行模特推荐。

### 判断服装类型（从描述或图片推断）

| 关键词 | 类型 | 阿里云参数 |
|--------|------|-----------|
| 上衣/T恤/衬衫/外套/西装/卫衣/夹克 | 上装 | `top_garment_url` |
| 裤子/裙子/短裤/半裙/阔腿裤 | 下装 | `bottom_garment_url` |
| 套装/全套/连衣裙/JK/制服/工装连体 | 全身 | 两个参数都传 |

推断不出来再问，问法：「这件是上衣还是裤裙，或者是套装？」

### 模特图来源判断（必须执行，不能跳过）

```
用户有提供模特图？
  ├── 有（URL 或上传图片）→ 直接用，不推荐内置模特
  └── 没有 → 必须动态读取 models.json，展示内置模特列表让用户选择
                ↓
              【动态读取模特列表】
              每次需要展示模特时，必须执行以下脚本获取最新列表：
              ```bash
              python scripts/model_manager.py list
              ```
              该命令会从 assets/models.json 读取全部模特并按性别/年龄分组显示。

              或者直接读取 assets/models.json 文件，用以下格式展示：
              ─────────────────────────────────
              👩 女装模特：
                 👩 柔妍  优雅风  165cm  矩形身材
                    气质优雅，适合正装、旗袍、礼服…
                 👩 雅琪  甜妹系  167cm  草莓型身材
                    甜美可爱，适合Lolita、JK制服…
                 …（展示全部女装模特）

              👨 男装模特：
                 👨 易峰  运动风  183cm  倒三角身材
                    运动型男，适合运动服、街头风…
                 …（展示全部男装模特）

              👧👦 童装模特：
                 …（如有）
              ─────────────────────────────────

              回复姓名，或说「推荐」让我根据服装帮你选」

              ⚠️ 禁止在此处硬编码模特名单！models.json 随时会增删模特，
                 必须每次动态读取。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Anatomic-universalquantifier643/shop-tryon-skill](https://github.com/Anatomic-universalquantifier643/shop-tryon-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
