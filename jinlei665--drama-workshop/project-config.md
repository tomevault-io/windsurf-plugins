---
trigger: always_on
description: **项目名称**: Drama Studio (短剧漫剧创作工坊)
---

# Drama Studio - AGENTS.md

## 项目概览

**项目名称**: Drama Studio (短剧漫剧创作工坊)
**项目描述**: AI驱动的短剧视频生成工具，支持分镜管理、图片生成、视频生成等功能
**技术栈**:
- **Framework**: Next.js 16 (App Router)
- **Core**: React 19
- **Language**: TypeScript 5
- **UI 组件**: shadcn/ui (基于 Radix UI)
- **Styling**: Tailwind CSS 4
- **Database**: Supabase (PostgreSQL)
- **Storage**: 阿里云 OSS (ali-oss SDK)
- **AI SDK**: coze-coding-dev-sdk (图像、视频、语音生成)

## 核心功能模块

### 0. 脚本管理 (Scripts)
- **路径**: `src/app/api/scripts/`
- **数据库表**: `scripts`
- **功能**: 创建脚本、编辑脚本、删除脚本、查看脚本列表
- **数据结构**:
  - `id`: 唯一标识符
  - `project_id`: 项目ID
  - `title`: 脚本标题
  - `content`: 脚本内容
  - `description`: 脚本描述
  - `status`: 状态（active/inactive）
  - `created_at`: 创建时间
  - `updated_at`: 更新时间
- **AI 分析**: `/api/scripts/analyze` - 根据 AI 分析生成角色和分镜，直接插入数据库
- **注意事项**:
  - Supabase PostgREST schema cache 需要手动刷新（在 SQL Editor 中执行 `NOTIFY pgrst, 'reload';`）
  - 沙箱环境无法连接到 Supabase 的 IPv6 数据库，已实现自动 fallback 机制

### 1. 项目管理 (Projects)
- **路径**: `src/app/api/projects/`
- **数据库表**: `projects`
- **功能**: 创建项目、查看项目列表、删除项目

### 2. 分镜管理 (Scenes)
- **路径**: `src/app/api/projects/[id]/scenes/`
- **数据库表**: `scenes`
- **功能**: 创建分镜、编辑分镜、删除分镜、查看分镜列表
- **数据结构**:
  - `id`: 唯一标识符
  - `sceneNumber`: 分镜编号
  - `title`: 标题
  - `description`: 描述
  - `dialogue`: 对话内容
  - `action`: 动作描述
  - `emotion`: 情绪氛围
  - `characterIds`: 关联角色ID列表
  - `imageKey`: 图片存储key
  - `imageUrl`: 图片URL
  - `videoUrl`: 视频URL
  - `videoStatus`: 视频生成状态
  - `status`: 状态
  - `metadata`: 元数据（包含镜头类型、相机运动等）

### 3. 角色管理 (Characters)
- **路径**: `src/app/api/projects/[id]/characters/`
- **数据库表**: `characters`
- **功能**: 创建角色、编辑角色、删除角色、查看角色列表
- **数据结构**:
  - `id`: 唯一标识符
  - `name`: 角色名称
  - `appearance`: 外观描述
  - `frontViewKey`: 正面视图存储key
  - `imageUrl`: 图片URL

### 4. AI 生成功能

#### 4.1 图片生成
- **路径**: `src/app/api/generate/scene-image/`
- **SDK**: coze-coding-dev-sdk (image-generation)
- **功能**: 根据场景描述生成场景图片
- **存储**: 自动上传到阿里云 OSS，设置公开读取权限

#### 4.1.1 人物库图片生成
- **路径**: `src/app/api/generate/character-image/`
- **SDK**: coze-coding-dev-sdk (image-generation)
- **功能**: 根据人物描述生成人物图像（文生图）
- **存储**: 自动上传到阿里云 OSS，设置公开读取权限

#### 4.1.2 人物库三视图生成
- **路径**: `src/app/api/generate/character-triple-views/`
- **SDK**: coze-coding-dev-sdk (image-generation)
- **功能**: 根据参考图片生成三视图（图生图）
- **存储**: 自动上传到阿里云 OSS，设置公开读取权限

#### 4.1.3 项目角色视图生成
- **路径**: `src/app/api/generate/character-views/`
- **SDK**: coze-coding-dev-sdk (image-generation)
- **功能**: 生成角色的正面视图
- **存储**: 自动上传到阿里云 OSS，设置公开读取权限

#### 4.1.4 项目人物形象生成（文生图）
- **路径**: `src/app/api/generate/appearance-from-text/`
- **SDK**: coze-coding-dev-sdk (image-generation)
- **功能**: 根据文字描述生成人物形象
- **存储**: 自动上传到阿里云 OSS，设置公开读取权限

#### 4.1.5 项目人物形象生成（图生图）
- **路径**: `src/app/api/generate/appearance-from-image/`
- **SDK**: coze-coding-dev-sdk (image-generation)
- **功能**: 根据参考图片生成新的人物形象
- **存储**: 自动上传到阿里云 OSS，设置公开读取权限

#### 4.2 视频生成
- **路径**: `src/app/api/generate/videos/`
- **SDK**: coze-coding-dev-sdk (video-generation)
- **功能**:
  - 单帧模式：使用一张图片生成视频
  - 首尾帧模式：使用两张图片（首帧和尾帧）生成视频
  - 支持自定义视频时长（4-12秒）
  - 支持选择视频比例（16:9 或 9:16）
- **存储**: 自动上传到阿里云 OSS，设置公开读取权限
- **特殊处理**:
  - 自动上传本地图片到阿里云 OSS
  - 生成公网 URL 供 Bot API 访问
  - 禁用重试机制（maxRetries: 0），失败立即返回错误

#### 4.3 语音生成
- **路径**: `src/app/api/generate/voice/`
- **SDK**: coze-coding-dev-sdk (audio)
- **功能**: 根据文本生成语音
- **用途**: 为角色配音

#### 4.4 角色视图生成
- **路径**: `src/app/api/generate/character-views/`
- **SDK**: coze-coding-dev-sdk (image-generation)
- **功能**: 生成角色的正面视图

### 5. 视频合并
- **路径**: `src/app/api/videos/merge/`, `src/app/api/episodes/[id]/merge-videos/`
- **功能**: 将多个视频片段合并成一个完整视频
- **工具**: FFmpeg

### 6. 人物库管理 (Character Library)
- **路径**: `src/app/api/character-library/`, `src/app/characters/`
- **数据库表**: `character_library`
- **功能**: 创建人物、编辑人物、删除人物、查看人物列表、从人物库导入到项目
- **数据结构**:
  - `id`: 唯一标识符
  - `name`: 人物名称
  - `description`: 人物描述
  - `appearance`: 外貌描述
  - `personality`: 性格描述
  - `tags`: 标签（如性别、年龄等）
  - `image_url`: 参考图片 URL
  - `front_view_key`: 正面视图存储 key
  - `style`: 图像风格（realistic、anime、cartoon、oil_painting）
  - `created_at`: 创建时间
- **API 端点**:
  - `GET /api/character-library`: 获取人物库列表（支持搜索）
  - `POST /api/character-library`: 添加人物到人物库（支持上传参考图和文字描述）
  - `PUT /api/character-library?id=xxx`: 更新人物库中的人物
  - `DELETE /api/character-library?id=xxx`: 从人物库删除人物
- **前端功能**:
  - 人物卡片显示参考图片
  - 新建人物时支持上传参考图和填写文字描述
  - 编辑人物信息
  - 上传参考图生成三视图（图生图）
  - 删除人物
- **项目人物管理**:
  - 支持从人物库导入人物
  - 支持添加项目人物到人物库
  - 支持文字生成新形象（文生图）
  - 支持形象管理（上传图片、添加形象、设置主形象、拖拽排序）
- **注意事项**:
  - 人物库 API 支持 Supabase + pg fallback 机制，解决 IPv6 连接问题
  - 参考图片上传到阿里云 OSS，生成公网 URL
  - 三视图使用 image-to-image 生成，保持人物特征一致

### 6.5 AI 独立生成模块 (AI Create)
- **路径**: `src/app/create/`, `src/app/api/create/`
- **功能**: 提供独立的 AI 生成功能（不存入数据库，直接生成和下载）
- **页面结构**:
  - `/create` - AI 生成首页（导航卡片展示四种生成功能）
  - `/create/text-to-image` - 文生图页面
  - `/create/image-to-image` - 图生图页面
  - `/create/text-to-video` - 文生视频页面
  - `/create/image-to-video` - 图生视频页面
- **页面导航**: 
  - AI 生成首页显示导航卡片，点击进入对应功能页面
  - 首页包含返回主页按钮
  - 所有子页面都包含返回按钮，方便用户返回 AI 生成首页

#### 6.5.1 文生图 (Text-to-Image)
- **路径**: `src/app/api/create/text-to-image/route.ts`
- **前端页面**: `src/app/create/text-to-image/page.tsx`
- **功能**:
  - 输入文本提示词和反向提示词
  - 选择图像风格（写实、动漫、卡通、油画等）
  - 选择图像尺寸（512x512 到 1536x1024）
  - **LLM 优化提示词**：使用 AI 智能优化用户输入的提示词
  - 生成图像并支持预览、下载、复制链接
- **存储**: 生成结果优先上传到阿里 OSS，失败则保存到本地 `public/ai-create/`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jinlei665/drama-workshop](https://github.com/jinlei665/drama-workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
