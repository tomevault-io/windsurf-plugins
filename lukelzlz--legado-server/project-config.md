---
trigger: always_on
description: > **核心项目定位**：将 **Legado（开源阅读 Android 平台应用）** 完整迁移与重构为可在标准服务器与 Docker 容器中独立运行的高性能**无头后端（Ktor + Kotlin JVM + SQLite）**与现代化**Web 客户端（React 19 + TypeScript + Vite）**。
---

# Repository Guidelines & Constitutional Conventions

## 1. Project Core Mission & Objective

> **核心项目定位**：将 **Legado（开源阅读 Android 平台应用）** 完整迁移与重构为可在标准服务器与 Docker 容器中独立运行的高性能**无头后端（Ktor + Kotlin JVM + SQLite）**与现代化**Web 客户端（React 19 + TypeScript + Vite）**。

### 关键迁移与设计原则
1. **彻底解耦 Android 平台依赖**：严禁在 `server/` 或 `web/` 模块中引入 Android Framework 原生组件（如 Activity、Context、Room、Android WebView、Android UI 等），必须使用跨平台的纯 JVM 技术栈与标准 Web API 替代。
2. **服务端无头（Headless）化架构**：服务端作为独立的中心服务，承载书源规则解析与执行沙箱（Rhino JS + Jsoup + JsonPath）、多书源并发搜索、离线书籍与封面缓存、订阅同步、PBKDF2/Session 鉴权与 SQLite 数据持久化。
3. **深度兼容书源生态**：完全复用并兼容 Legado 现有丰富的书源协议与规则定义，确保现有网络书源可在服务端正确、安全且高效地解析。
4. **现代化 Web 阅读体验**：Web 端作为跨平台终端，提供大章节目录虚拟化、单书源直连秒开、离线缓存进度同步及沉浸式阅读器交互。

---

## 2. 仓库边界与文档驱动规范 (Repository Boundary & Cleanliness)

本项目严格执行**文档驱动开发（Doc-Driven Development）**与工程整洁度控制：

- **严禁随地大小便**：严禁在项目根目录或业务源码目录中散落 `PLAN.md`, `NOTES.md`, `TODO.md`, `temp/`, `plan/` 等临时或未受管文档。
- **文档统一收敛机制**：
  - 需求与功能提案 (PRD) 归入 [`docs/proposals/`](docs/proposals/)
  - 架构决策记录 (ADR) 归入 [`docs/decisions/`](docs/decisions/)
  - 原始推演、历史排错与工作记忆归入 [`docs/sessions/`](docs/sessions/)
  - 用户实操验收手册与验证清单归入 [`docs/acceptance/`](docs/acceptance/)
- **工程复杂度惩罚**：严禁为了单一补丁随意增加无意义的抽象层、胶水层或重复工具类。优先采用满足当前需求的最简、最直接解法。

---

## 3. 最短验证命令集 (Shortest Verification Commands)

AI 在修改代码后，必须优先执行本节定义的最短、最精准命令自测：

```sh
# --- 1. 前端类型检查与自动化测试套件 ---
npm --prefix web run check
npx tsx web/test/run-all.ts

# --- 2. 服务端本地 JVM 单元测试 ---
./gradlew :server:test

# --- 3. 前后端综合一键验证 ---
npm --prefix web run check && npx tsx web/test/run-all.ts && ./gradlew :server:test

# --- 4. 本地 Docker 容器构建与热更新验证 (用户常用测试基准) ---
docker build -f Dockerfile.server -t test-legado-server:latest .
docker stop test-legado 2>/dev/null || true
docker rm -f test-legado 2>/dev/null || true
docker run -d --name test-legado -p 8080:8080 \
  -e ADMIN_PASSWORD=admin123 \
  -e LEGADO_SECURE_COOKIES=false \
  -v $(pwd)/.data:/data test-legado-server:latest
curl -s -o /dev/null -w "%{http_code}\n" http://127.0.0.1:8080/index.html
```

> **Windows 本机执行注意（详见 [`docs/sessions/SESSION-HIST-008`](docs/sessions/SESSION-HIST-008-windows-environment-and-verification-baseline.md)）**：
> - JDK 固定用 **Amazon Corretto 21**；`JAVA_HOME` 未持久化时需前缀 `$env:JAVA_HOME="C:\Program Files\Amazon Corretto\jdk21.0.12_9"`；
> - `./gradlew :server:test` 在本机**长期固定 52 个失败**（测试删不掉被占用的 SQLite，macOS 不暴露）。**判定回归必须与「干净基线 worktree 的失败集合」逐条比对，禁止只看失败数量**；
> - 前端依赖缺失时先 `npm --prefix web install --ignore-scripts`（esbuild 的 postinstall 会被拦截），再跑 `check` / `tsx test/run-all.ts`；
> - 明文 HTTP 联调统一带 `LEGADO_SECURE_COOKIES=false`，否则浏览器不回传 Secure Cookie、表现为「登录后立刻掉线」。

---

## 4. 完成度状态阶梯 (Completion States)

AI 与人类协作时必须明确当前达到的完成度阶梯，严禁混淆概念：
1. `[Modified 本地改动]`：代码已编写完成，但尚未执行测试。
2. `[Tested 单测/检查通过]`：已通过本地最短单测（`gradlew :server:test`）与静态类型检查（`npm run check` / `tsx run-all.ts`）。
3. `[Deployed 容器/服务联调完成]`：已部署/更新至本地 Docker 容器并启动，且日志与接口无报错。
4. `[Accepted 用户签字确认]`：用户已根据验收手册实操验证并明确回复“验收通过/LGTM”。
5. `[Pushed 提交与推送]`：代码已提交至 Git 并推送到远端仓库。

---

## 5. 部落知识库与历史踩坑 (Tribal Knowledge)

基于历史会话全量扫描提炼的架构潜规则与避坑指南：

- **[书源解析/兼容] `bookSourceUrl` 允许任意非空唯一字符串**：Legado 书源生态中部分聚合或定制书源（如 `大灰狼融合VIP5.0`）使用自定义中文或标识作为 `bookSourceUrl`，服务端严禁粗暴强制要求 `http(s)://`；同时前端与服务端导入均需兼容 UTF-8 BOM 编码及 `{ data: [...] }` / `{ sources: [...] }` / `{ bookSources: [...] }` 等外层包装结构。
- **[SQL/Kotlin] 严禁使用可空列做存在性 Elvis 判断**：在 JDBC / SQLite 结果集提取中，务必区分“字段值为 NULL”与“数据行不存在”。例如 `SELECT cover_key FROM book_shelf`，若书籍无封面则字段为 `NULL`，直接 `rs.getString(...) ?: return null` 会误判书籍不存在。
- **[前端/Form] 按钮显式声明 `type="button"`**：表单内的所有辅助操作按钮（如停止搜索、清空、排序切换）必须显式标注 `type="button"`，否则点击会触发 HTML 表单默认 `submit` 事件导致搜索意外重启。
- **[翻页/排版] 跨章逆向翻页定位守卫**：从章节开头回翻到上一章时，必须携带 `targetPosition = 'bottom'` 标记，且必须在 DOM/分栏异步排版完成后再执行末尾定位，严禁在未完成排版前盲目计算滚动高度。
- **[性能] 缓存优先直出与流式防抖**：进入阅读器时优先命中本地 `BookCacheService` 离线缓存分片，避免等待全量远程 TOC；流式搜索推送高频数据时前端需保持批量节流合并渲染。
- **[凭据/序列化] 万能 Cookie 解析与强类型 DTO**：服务端 CookieJar 存库前必须通过 `parseCookieString` 统一归一化为 `k1=v1; k2=v2` 格式，杜绝存入原始 JSON 数组脏数据；Ktor 路由响应严禁使用非多态的 `Map<String, Any>`，必须使用 `@Serializable data class`。
- **[TTS/朗读] Edge-TTS 协议与 Chrome 假死守卫**：Edge-TTS WebSocket 通信中 SSML 必须严格做 XML 特殊字符转义（`&`, `<`, `>`, `"`, `'`），并且 WebSocket 通信块必须加 `try-catch(abort)` 彻底规避超时句柄悬挂；浏览器端 `SpeechSynthesis` 在无心跳朗读超过 15 秒时会被 Chrome 自动静默冻结，前端必须保持定时短暂停与恢复的看门狗循环。
- **[TTS/朗读] Edge-TTS WebSocket 握手版本必须与 Chromium 同步更新**：Edge-TTS 连接头中 `Sec-MS-GEC-Version`（如 `1-143.0.3650.75`）与 `User-Agent` 中的 Chrome 版本号必须保持一致；同时须携带随机 `Cookie: muid=<16字节大写hex>` 头，否则 WebSocket 握手被微软服务端拒绝导致合成静默失败（返回 0 字节音频）。版本信息参考 `edge-tts` Python 包的 `constants.py`。
- **[TTS/朗读] 孤立标点切片导致 ERR_REQUEST_RANGE_NOT_SATISFIABLE 的三层防御**：TTS 分句正则可能将中文对话引号 `"` 切为孤立碎片，发送空/纯标点文本至 Edge-TTS 会返回 0 字节音频，前端 `URL.createObjectURL(0字节blob)` 后浏览器发出 Range 请求，得到 HTTP 416 崩溃。**必须在三处同时加守卫**：① `splitSentences` 过滤去标点后有效字符 `< 2` 的碎片；② `HttpAudioTtsEngine.speak/prefetch` 调用 `isEffectiveText()` 判断，无效时 `setTimeout(onEnd,0)` 跳过；③ 服务端 `EdgeTtsService.synthesize` 检测去标点后有效字符 `< 2` 直接返回 `ByteArray(0)` 不请求上游。
- **[TTS/朗读] 跨章连播状态与正文加载竞态守卫**：切章（`changeChapter`）时必须同步将 `loadedChapterUrl` 置空并清除旧 `content`，连播 `useEffect` 必须严格校验 `loadedChapterUrl === chapter?.url` 且使用 `playTtsChunkRef.current` 调用最新闭包，杜绝切章瞬间误读上一章旧正文；正文段落点击选播严格守卫 `if (!ttsActive) return`，防止普通阅读点选误触发朗读。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukelzlz/legado-server](https://github.com/lukelzlz/legado-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
