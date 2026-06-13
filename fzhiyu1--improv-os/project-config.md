---
trigger: always_on
description: 一个发布到互联网博大家一笑的体验项目：macOS 风格的网页"操作系统"，壳是手写的，里面所有应用都由 LLM 实时生成。灵感致敬 vibe os。
---

# ImprovOS（工作名，待定）

一个发布到互联网博大家一笑的体验项目：macOS 风格的网页"操作系统"，壳是手写的，里面所有应用都由 LLM 实时生成。灵感致敬 vibe os。

## 已验证的事实（2026-06-11 实测）

- 上游：任意 Anthropic `/v1/messages` 兼容端点（官方 API 或自建网关），地址/密钥走 `.env` 的 `ANTHROPIC_BASE_URL` / `ANTHROPIC_AUTH_TOKEN`（内部环境细节见 `deploy/INTERNAL.md`，不入库）
  - 模型名 `claude-sonnet-4-6`，**实测 815 tok/s**（2000 token 完整计算器 2.5s）
  - SSE 流式可用；响应先吐 `thinking` block 再吐 `text` block——前端把 thinking 演成"系统思考中…"，text 演成代码瀑布
- key 在 `.env`（已 gitignore），**永不进前端、永不进 git**

## 核心设计决策

1. **壳手写，软件皆生成**：桌面/菜单栏/Dock/窗口管理/Spotlight 搜索是真代码；计算器、备忘录等 Dock 应用**点开即重新生成、永不缓存**（每次长得不一样是核心笑点）
2. **浏览器套娃**：生成一个浏览器应用，里面"访问"的网页也是现编的
3. **只有 Spotlight 搜索召唤出的应用会落盘缓存**（`apps/<slug>/`，文件系统即真相），命中秒开，带 🎲 重新投胎。Dock「应用程序」启动台（`web/js/launchpad.js` + `/api/apps`）网格展示全部缓存应用：确定性图标（名字 hash 渐变 squircle + 关键词 SF 字形，id 唯一化防渐变串色）+ 启发式分类胶囊 + 搜索，点击走 openSearchApp 秒开
4. **双轨生成**：默认单次流式直出自包含 HTML（2-3s，主演出）；可选慢轨走 openCode SDK 智能体循环（"雇佣 AI 工程师"真人秀），不绑架主体验
5. **安全**：key 只在服务端；代理只暴露"按固定 system prompt 生成应用"单一能力；IP 限流 + 全站日熔断；生成物跑 `sandbox="allow-scripts"` iframe；搜索词轻量审核（违规返回"已查封"梗页面）

## 运行时能力桥（Part A，2026-06-11 已上线）

生成应用可调 `window.os`（系统注入 SDK，sandbox iframe 内仍无直接网络）：
- `os.ai.ask(prompt)` → AI 文本应答（固定 system，限定应用场景）
- `os.http.get(url)` → 公网只读 GET。唯一防线是服务端 SSRF 过滤（`server/lib/ssrf.mjs`：私网黑名单 + 协议白名单 + DNS 校验 + 固定 IP 连接防 rebinding + 重定向逐跳重验 + 8s 超时 + 512KB 上限）
- `os.store.*` → 按 appId 命名空间的共享 KV（`apps/_store/`，同名应用所有访客共享；单条 64KB/总量 1MB/100 键）

三层链路：iframe SDK（postMessage，init 握手前排队）→ `web/js/bridge.js`（**只认 registry 登记过的 e.source，忽略自报 appId**）→ `/api/capability/*`（各自限流：ai 20/min、http 30/min、store 120/min，`CAP_*_PER_MIN` 可调）。
appId 命名空间：dock=`app:<id>`，搜索=slug，浏览器=`web:<djb2>`。

⚠️ 实测坑（2026-06-11）：
- **req close ≠ 客户端断开（2026-06-12 事故）**：Node 15+ `IncomingMessage` 的 `close` 在**消息体读完**就触发。GET 没人消费 body 侥幸无感；POST（/api/repair）读完 body 即触发 → `makeSession` 误判断开自杀（掐上游、catch 因 aborted 静默、不发 done/error）→ **修复功能自上线起 100% 卡死且零日志**。断开检测一律挂 `res.on('close')` + `!res.writableEnded`。修复同时补了 genGate（修复曾绕过并发闸）。教训：每条对外链路上线时都要端到端冒烟，"代码看起来对"不算
- **destroy() 不带 error 不发 'error' 事件（2026-06-12 事故）**：访客中途关页面 → res close → `s.current?.destroy()` 杀上游请求，但 destroy 不带参不会触发 'error'，cb 永不执行 → `s.step` 的 Promise 永久 pending → **genGate 并发槽随每次中途断开永久泄漏**（线上待机 fastActive 长期 4/10）。修复：会话记录 step 的 reject，断开时手动 `abortStep(new Error('aborted'))` 摇醒等待者；429 退避窗口加 isAborted 检查不再白烧上游。回归：`tests/slot-leak.e2e.test.mjs`（真服务 + 假流式上游 + 中途掐线断言归零）。教训：杀掉底层资源 ≠ 摇醒上层等待者，凡 destroy 必问"谁在 await 它"
- **.env 加载顺序（2026-06-12 事故）**：ES import 全部提升到模块体之前求值——lib 在模块层读 `process.env`（如 opencode.mjs 的 OC_*）时 .env 还没注入，永远拿默认值。OC_PROVIDER 默认名一改慢轨全断（之前 default==实际值纯属侥幸）。修复：`server/lib/env.mjs` 作为 index.mjs **第一个 import**
- Node 20+ `autoSelectFamily` 以 `all:true` 调自定义 lookup，须返回数组（已修）
- Spotlight"获取/完整版"路径必须传 `/api/search` 返回的 slug，否则 appId 为空、桥不接线（已修）
- open-meteo 被源站网络出口 502 拦截；prompt 内推荐数据源用 wttr.in / exchangerate-api / hacker-news（均实测可达）
- 测试：`node --test 'tests/*.test.mjs'`（16 个，ssrf/store/ratelimit）

## 慢轨真 Agent Loop + 修改应用（Part B，2026-06-11 已上线）

Spotlight"完整版"（mode=deep）与缓存应用"修改"按钮走真正的 openCode agent loop：
- **架构**：opencode 独立常驻（`opencode serve` :4096，systemd `opencode.service`，每日重启防泄漏），improv-os 用**纯 Node http REST**驱动（不引 `@opencode-ai/sdk`，仍零依赖）。`server/lib/opencode.mjs` 是客户端。
- **流程**：创建 session（工作目录 `apps/_agent/`）→ 订阅 `/event` SSE → 下发任务 → agent 写 `app.html` 并用 bash 跑 `verify-html.mjs` 自检迭代 → `session.idle` 后读产物 → injectSys 注入 SDK → 落盘缓存 → 删 session。
- **验证用 node 脚本而非 chromium**（`server/verify-html.mjs`，省 300-500MB 适配 1.9GB）。
- **事件直播**：`server/lib/agent-events.mjs` 把工具调用映射成剧场 stage（"正在编辑文件/正在运行验证"）；文本 part 不直播（part.text 是累积全文会刷屏）。
- **修改应用**：`/api/modify`（POST {slug, instruction}），把现有 index.html 拷进工作目录让 agent 增量编辑，写回缓存。只缓存应用可改（dock 应用每次重生成）。
- **并发限 1**（`server/lib/gate.mjs`），工作目录单一不冲突。
- **挂死防护（2026-06-12，事故驱动）**：一个挂死的 agent 曾占住唯一深轨坑位 22 分钟堵死所有人（02:42 的 modify 无 done、无日志、无超时）。三层修复：① 硬墙钟超时 `DEEP_TIMEOUT_SEC`（默认 150s）`Promise.race` 中止（弃赛侧 promise 必须先 `.catch(()=>{})` 防 unhandled rejection），finally 删会话即终止 opencode 侧运行；② agent 错误全部落 `agent_error` 活动事件（含 timeout 标记，控制台「异常」可见）；③ 启动清扫 `listSessions(AGENT_WORK)` 全删——进程被杀时 finally 不执行，孤儿会话白吃内存。

⚠️ 实测（2026-06-11）：
- 本机 opencode 1.1.53 / 源站 1.17.3，REST 形态兼容（session.id / message.info.finish / /event 一致）。
- 深轨生成 ~16-27s，修改 ~10s。内存：improv+opencode RSS≈583MB，源站可用余量 >1GB。
- 旧伪 agent `generateDeep` 保留为 openCode 不可用时回退（ocHealth 预检）。
- 配置：`deploy/opencode.json`（gateway provider→上游）、`deploy/opencode.service`；key 走 .env 的 ANTHROPIC_AUTH_TOKEN，OC_PORT/OC_PROVIDER/OC_MODEL 也在 .env。

## 公网上线（os.fzhiyu.dev，2026-06-11）

通过 Cloudflare Tunnel 发到公网供大家游玩。源站在内网，cloudflared 反向连 Cloudflare 边缘，源站 7100 不对公网暴露（隐藏源站 IP）。
- **入口**：`https://os.fzhiyu.dev`（`cloudflared.service` systemd 常驻，双边缘连接冗余）
- **后端**：上游网关 Sonnet（源站可直连）。**测试期 token 充足**，故不锁 token、**锁并发**。
- **限流转向**：`DAILY_TOKEN_BUDGET=0`（关，留兜底），核心是并发闸 `GEN_CONCURRENCY=5`/`GEN_QUEUE=24`/`DEEP_QUEUE=8` 保护上游网关不被打爆。
- **全链路实测通**：快轨代码瀑布（长连接 SSE 经 Cloudflare 不缓冲）、深轨完整版（12.9s）、三能力、修改应用。
- **运维坑**：opencode 是 user service，必须 `loginctl enable-linger <部署用户>`（已开），否则部署者登出被杀。
- 部署实操与内部环境见 `deploy/PUBLIC-LAUNCH.md` + `deploy/INTERNAL.md`（均 gitignore 不入库）；凭据（cert.pem/<id>.json）只在源站绝不进 git。

### 反盗用加固（2026-06-12）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fzhiyu1/improv-os](https://github.com/Fzhiyu1/improv-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
