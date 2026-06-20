---
trigger: always_on
description: Use when generating Amazon listing images (main/scene/detail/A+) from user specs and reference images, and you must call wan2.7-image with strict Amazon-compliant prompt rules.
---


# Amazon 商品图生成（Wan2.7 可复用 Skill）

## Overview

将“用户输入 + 参考图”转成可执行的 Amazon 商品图生成方案：先做需求澄清与结构化输入，再按图片类型生成严格约束的提示词，最后通过 `wan2.7-image-skill` 调用生成并输出可直接用于 Amazon Listing 的图片。

**REQUIRED SUB-SKILL:** `wan2.7-image-skill`

## When to Use

- 需要生成 Amazon Listing 的商品主图、场景图、细节图、A+ 图
- 用户提供了 1–3 张参考图，要求保持"同一商品身份一致性"
- 需要把"营销文案"以严格规则渲染到图片里（只允许出现指定文本，且只出现一次）
- 需要对已生成的图片进行编辑修改（背景更换、颜色调整、元素增删等）
- 需要将多个参考图（多角度或多 SKU 变体）自然融合到同一场景中生成图片
- 需要把生成流程做成可复用的标准化输入 + 提示词 + 调用方式

## Outputs

- 每张图：`{编号, type, size, prompt, image_url}`
- **编号格式**：`IMG-001`、`IMG-002`...，每次生成自动递增
- **【强制要求】必须为每张生成的图片分配唯一编号**，包括默认图生产品图、融合生图和编辑图
- **【重要】image_url 必须是完整的预签名 URL，包含所有查询参数（Expires、OSSAccessKeyId、Signature 等），禁止截断或省略任何参数**
- 推荐输出集合：
  - `main` 1–2 张（白底主图）
  - `scene` 1–4 张（生活方式/场景图）
  - `detail` 1–4 张（细节特写）
  - `aplus` 1–4 张（A+ 氛围营销图）
  - `dimension` 1 张（尺寸标注图，仅当用户提供尺寸数据时生成）

## Input Intake（输入逻辑）

### 1) 必填信息（最少可开工）

- 商品名称（用于提示词的语义锚点）
- 参考图 1–3 张（本地文件 / 公网 https URL / 已上传的 oss:// URL）
- 需要生成的图片类型与数量（main/scene/detail/aplus）

### 2) 强烈建议补充（显著提升可控性）

- 类目（category）、材质（material）、尺寸（dimensions）
- 使用场景（useCase）、目标人群（targetAudience）
- 风格档案（style profile）：`minimal_modern`（现代简约） / `japanese_soft`（日系柔和） / `luxury_editorial`（轻奢质感）/ `rounded_bold`（圆润大粗体）/ `yahoma_light`（雅黑light）

当用户未提供 `dimensions` 时，必须在场景图/A+图里明确要求"真实世界尺寸与比例"，并用环境物体做比例参照，避免产品被生成成不符合常识的大小。

### 3) 文案（可选）

对 `scene/detail/aplus` 单张图可填写一条"要渲染到图里"的文案（description）。该文案必须满足：

- 单行文本
- 必须逐字逐符号完全一致（不得改写、翻译、补字、加标点）
- 图片中除该文案外不得出现任何其他文字

### 4) 文案处理检查点（MANDATORY）

**在生成提示词前必须执行以下检查：**

1. **文案识别检查**
   - 检查用户输入中是否包含文案（如"文案：xxx"、"copy: xxx"、"带xxx文案"等）
   - 提取文案内容并验证格式是否符合要求（单行文本，无特殊格式）

2. **模板选择检查**
   - 如果检测到文案：必须选择对应的"有文案"模板（如 scene 有文案模板）
   - 如果未检测到文案：必须选择对应的"无文案"模板

3. **提示词构建检查**
   - 有文案时：必须在提示词开头添加完整的"唯一文本规则"锚点
   - 有文案时：必须在提示词中明确指定文案位置和排版规范
   - 有文案时：必须在提示词末尾添加 TEXT_TO_RENDER_REPEAT 验证行

4. **执行前验证**
   - 检查生成的提示词是否包含 CRITICAL TEXT TO RENDER 块
   - 检查文案内容是否与用户输入完全一致
   - 检查是否有任何可能导致文案渲染失败的模糊表述

**错误处理：**
- 如果检测到文案但未正确构建提示词：停止执行并给出明确错误提示
- 如果文案格式不符合要求：提示用户修正文案格式
- 如果提示词中包含模糊表述（如"No text unless provided"）：替换为明确的文案渲染指令

### 5) 风格映射规则（MANDATORY）

**中文风格输入到英文风格档案的映射：**

| 中文风格输入 | 英文风格档案 | 描述 |
|-------------|-------------|------|
| 现代简约、简约现代、现代风、简约风 | `minimal_modern` | 几何无衬线字体，干净现代电商感 |
| 日系、日系风、日式、和风、日系柔和 | `japanese_soft` | 柔和手写/圆润字体，温柔轻量 |
| 高级、高级感、轻奢、轻奢质感、编辑风、杂志风 | `luxury_editorial` | 高对比现代衬线，轻奢质感 |
| 圆润大粗体、圆润粗体、胖体、可爱粗体 | `rounded_bold` | 超圆润几何无衬线，粗壮饱满，友好亲切 |
| 雅黑light、微软雅黑、细体、纤细字体 | `yahoma_light` | 几何无衬线，纤细轻量，现代简洁 |

**Typography 选择规则：**
- `minimal_modern`：使用 `modern geometric sans-serif headline, slightly condensed, clean letterforms, high readability, medium weight`
- `japanese_soft`：使用 `rounded minimal sans-serif with a soft friendly feel, warm tone, subtle` 或 `soft handwritten pen lettering, gentle thin-to-medium strokes, warm and subtle`
- `luxury_editorial`：使用 `high contrast modern serif, editorial style, refined and elegant, medium weight`
- `rounded_bold`：使用 `ultra-rounded geometric sans-serif headline, extra bold weight, soft corners, friendly and playful, high impact`
- `yahoma_light`：使用 `clean geometric sans-serif, light weight, subtle, microsoft yahei inspired, minimal and modern`

**执行要求：**
- 当用户输入中文风格名称时，必须映射到对应的英文风格档案
- 必须根据映射的风格档案选择相应的 typography
- 确保生成的文案风格与用户选择的风格一致

## Generation Settings（生图设定）

### 分辨率建议

- `main/scene/detail`：建议 `1600*1600`（或 `1K/2K`）
- `aplus`：建议宽幅，如 `1200*600`、`1464*600` 等（按 A+ 模板需要调整）

### 一致性策略

- 参考图用于“主体一致性”：在所有类型中使用同一组参考图作为输入（1–3 张）
- 提示词中强制声明“同一商品身份不改变”：形状、颜色、材质、细节不变

## Preset Scenes（预设场景库）

生成 scene 类型图片时，可从以下 11 个预设场景中选择：

| 编号 | 场景名称 | 场景描述 | 场景提示词片段 |
|------|---------|---------|--------------|
| 1 | 桌面 (Desk/Table) | 摆件、数码产品、文具等 | Place the product naturally on a clean desk surface. **Lighting: 45° side window light, softbox intensity 65%, shadow softness medium. Light temperature: 4000K neutral daylight.** Add subtle shadows. The desk can have minimal items like a notebook, pen, or coffee mug to establish scale and context. Keep the product as the clear focal point with the desk surface taking up the lower third of the frame. |
| 2 | 浴室 (Bathroom) | 洗漱用品、浴室收纳等 | Position the product in a modern bathroom setting. **Lighting: Top-down overhead fixture + ambient vanity mirror lights, 3200K warm tone, slight moisture mist post-effect for depth.** Use marble, quartz, or white tile as backdrop. Add context with items like folded towels, a mirror reflection, or small plants. Premium spa-like atmosphere with soft volumetric light. |
| 3 | 卧室 (Bedroom) | 床头装饰、纺织用品等 | Arrange the product naturally in a bedroom scene. **Lighting: Warm 2700K table lamp from side, soft fill from window (curtain diffused), shallow DOF f/2.8.** Place on a nightstand, dresser, or bed. Use soft bedding (linen or cotton), warm lamp lighting, and subtle textures like throw pillows or blankets. Cozy, restful atmosphere. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [colddew-yj/amz-product-img-gen-skill](https://github.com/colddew-yj/amz-product-img-gen-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
