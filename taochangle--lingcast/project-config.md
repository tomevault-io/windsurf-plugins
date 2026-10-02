---
trigger: always_on
description: 本文档供 AI Agent 或开发者在**新设备上 clone 本仓库后**快速理解现状并继续开发。
---

# Talking Avatar Platform — 开发交接说明

本文档供 AI Agent 或开发者在**新设备上 clone 本仓库后**快速理解现状并继续开发。
完整的上手步骤见 [README.md](./README.md)。

## 1. 项目是什么

端到端口播数字人平台：管理后台上传形象图片 + 选择 Edge-TTS 音色创建数字人，后台用
**LivePortrait 预处理生成基础视频 + Edge-TTS 语音 + Wav2Lip(ONNX) 口型合成**生成
带声音的说话视频。

## 2. 当前技术栈（已落地，勿按旧文档改回）

- 前端：React + TypeScript + Vite + Tailwind + shadcn/ui（基于 satnaing/shadcn-admin），
  pnpm 10.25.0 管理，`frontend/admin/`（BFF 多客户端矩阵：`admin/live/web/telegram`）。
- 后端：Go + Gin + GORM，`backend/`（模型、S3、Redis、handlers 分为 admin/live/telegram/web）。
- AI Worker：Python **3.11**（`worker/.python-version`），uv 管理依赖（`pyproject.toml`
  + `uv.lock`），**不要使用 requirements.txt**。
- 存储：S3 兼容对象存储（RustFS，Docker Compose 内运行）；Go 用 `aws-sdk-go-v2`，
  Python 用 boto3；服务间只传 S3 Key，不用本地路径。
- 基础设施：Docker Compose；**MariaDB 11** + **Redis 8.2.2-alpine** + RustFS
  + **service-rag**（本地 RAG 微服务：zvec 全文索引 + Jieba 中文分词，
  Docker 内 `services/rag/`，端口 8001，零模型依赖）+ **service-tts**
  （Edge-TTS 微服务：async edge-tts → 16kHz PCM WAV → S3，`services/tts/`，
  端口 8002，S3 共享存储，媒体不过 HTTP）+ **docs**（API 文档网关：nginx
  聚合各 HTTP 微服务的 Swagger/OpenAPI，经管理端 `:8080/doc/<prefix>` 访问）。
- 部署模式：Docker 里跑基础设施 + API + 前端 + 轻量 Mock Worker；**真实 AI Worker
  在宿主机原生运行**：macOS Apple Silicon（MPS/CoreML）、Linux NVIDIA CUDA、
  Linux **AMD ROCm**（RX 6800 XT 等 RDNA2，见 README 对应章节）。宿主机仅开放
  必要端口：3000（观众端）/ 8080（管理端）/ 1935（RTMP 推流）/ 6379（Redis）/
  9000（RustFS）；`api-admin`/`api-live`/`api-telegram`/`api-web`/`api-scheduler`、`service-rag`、
  `service-tts` 均在内网，不发布端口。
- 依赖组（`worker/pyproject.toml`）：`models`（macOS 默认）、`cuda`、
  `rocm`（仅 `onnxruntime-rocm`，torch 用官方 ROCm 镜像或手动装）——cuda/rocm
  互斥，Linux 用 `--no-group` 二选一；ROCm 容器里 `uv sync` 必须加 `--inexact`
  保留镜像预装的 torch。

## 3. 当前实现状态

- ✅ Avatar Studio（创建）：形象名称/头像展示/图片/Edge-TTS 音色（localStorage 缓存，
  默认 zh-CN-XiaoxiaoNeural）+ 人物设定（年龄/身高/体重/族裔/感情状态/性格）。
  **创建数字人不再自动生成视频**：只建身份 + 默认场景，`ready` = 至少一个场景 +
  至少一条视频（上传或生成均可），视频全部由用户在「场景」页添加。
- ✅ Broadcast（离线播报）页面：选数字人 + 脚本 → 任务轮询 → 播放成品。
- ✅ 场景（Scene）两级结构：一个数字人可有多个场景（标题/描述/封面），每个
  场景下 1-N 个驱动视频（视频 + 描述）；创建数字人时的 base 视频成为默认
  场景的默认视频（直播/播报兜底，不可删）。表：`scenes` + `scene_videos`
  （均带 avatar_id 索引，支持按数字人筛选）。人物设定（年龄/身高/体重/族裔/
  感情/性格）收进 `avatars.persona` JSON；`base_video_s3_key` 与旧
  `avatar_videos` 已彻底移除（启动时一次性迁移）。
- ✅ 任务进度 + TTS 复用：离线任务上报 `stage`（tts/lipsync/mux）+ `progress`
  （1% 步进），前端任务中心/播报历史显示阶段与进度条；TTS 首次合成后存
  `tts/tasks/{id}.wav`（S3），重试时直接复用、跳过 Edge-TTS。
- ✅ 客户端直播间（观众端）：**独立 Next.js + TailwindCSS 项目**（`frontend/live/`，端口
  **3000**，不属于管理后台）——首页有导航头（身份/注册/登录/退出）与**分类筛选**
  （`GET /api/live` 携带 avatar 的 `category`），`app/rooms/[avatarId]/page.tsx`
  用 xgplayer 拉 HTTP-FLV + 聊天输入 `POST /api/live/:id/message`；`app/api/[...path]`
  与 `app/live/[...path]` 路由处理器在**服务端**代理（`API_ORIGIN` / `LIVE_ORIGIN`
  运行时读取，docker 内为 `http://api-live:8082` / `http://srs:8080`，本地默认
  `http://localhost:8080`）。
- ✅ 聊天身份与记录持久化：`live_users/telegram_users`（游客/账号，bcrypt）与 `live_messages`
  （user/bot）两张表；`POST /api/chat/guest` 发临时身份，`register` 原地升级游客行
  （同一 userId 保住历史），`login` 把当前游客的消息合并进账号，退出后重新取新游客
  ID；`GET /api/chat/history?avatarId=` 拉持久化聊天记录；客户端身份存
  localStorage（`tav_chat_identity`），由 `IdentityProvider` 全局管理，注册/登录弹窗
  首页与直播间共用。`POST /api/live/:id/message` 会同时入库用户消息与机器人完整回复。
- ✅ 直播配置：Avatar 表 JSON 字段 `live_settings`（字幕
  `subtitleEnabled/subtitleFont/subtitlePosition/subtitleBorder/subtitleSize` +
  闲置推流 `idleSceneId/idleSwitchMode(interval|random)/idleSwitchSeconds`），
  `PUT /api/avatars/:id/live-settings` 读写；start 控制消息与 `GET /api/live`
  都会带上配置（含 `idleVideos` S3 列表）；worker 的 `SubtitleRenderer` 支持
  顶部/底部、描边宽度、字号，字体按文件名从 `worker/fonts/` 解析（缺失回退系统
  默认）；Live Studio「字幕设置」与「默认推流视频」卡片保存后自动重启直播生效。
- ✅ 管理端用户列表：`GET /api/users`（游客+账号，含消息数），侧边栏「用户相关 →
  用户列表」（/users）展示真实数据。
- ✅ **Telegram Mini App 登录**：`POST /api/auth/telegram`（api-telegram）用
  `TG_BOT_TOKEN` 校验 `initData`（HMAC-SHA-256：`secret=HMAC("WebAppData", token)`，
  对除 `hash` 外按字典序的 `key=value` 串签名，24h 新鲜度检查），通过后按
  `telegram_id` upsert `live_users/telegram_users`（非游客，用户名优先 TG username、冲突回退
  `tg_<id>`），设置 HttpOnly `tg_uid` cookie 并返回 `{userId,username,...}`；
  `frontend/admin/nginx.conf` 单独把 `/api/auth/telegram` 路由到 api-telegram（TMA
  走同一网关）。前端 `frontend/telegram` 初始化 `WebApp.ready()` 后把
  `WebApp.initData` POST 到 `${VITE_API_ORIGIN}/api/auth/telegram`，含加载中/
  成功/失败三态。
- ✅ **Ngrok 本地联调**：根目录 `ngrok.yml` 双隧道（`tg-app`→宿主机 3002、
  `api-gateway`→宿主机 8080）；compose 新增 `ngrok` 服务
  （`ngrok/ngrok:latest`，`start --all`，4040 检查台，`host.docker.internal`
  已映射 host-gateway）；`.env(.example)` 需填 `NGROK_AUTHTOKEN`/`TG_BOT_TOKEN`/
  `VITE_API_ORIGIN`（TMA 构建期注入 api-gateway 的 https 地址）。
- ✅ **Telegram Webhook 自动注册**：api-telegram 启动后在后台 goroutine 轮询
  `http://ngrok:4040/api/tunnels`（`NGROK_API_URL`，2s × 10 次重试），找到
  `api-gateway` 隧道的 `public_url` 后 POST
  `https://api.telegram.org/bot{token}/setWebhook` 注册
  `{public_url}/api/telegram/webhook`（不阻塞 Gin 启动）；占位端点
  `POST /api/telegram/webhook` 返回 200，管理端 nginx 将其与
  `/api/auth/telegram` 一起路由到 api-telegram。
- ✅ **Telegram 注册环境变量优先**：`TG_WEBHOOK_URL` / `TG_MINIAPP_URL` 设置后
  直接使用（生产固定域名），缺失时才轮询 ngrok 对应隧道（webhook→api-gateway，
  菜单按钮→tg-app）；两者都设置时完全不查询 ngrok；`setWebhook` +
  `setChatMenuButton`（web_app）分别注册，日志明确标注 URL 来源
  （Environment / ngrok）。
- ✅ 国际化（中/英）：管理端（react-i18next，`frontend/admin/src/i18n/`）与观众端

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taochangle/LingCast](https://github.com/taochangle/LingCast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
