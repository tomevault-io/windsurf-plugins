---
trigger: always_on
description: **fastapi-react-admin** —— 中后台底座。产品标识集中在 `apps/web/src/lib/brand.ts`
---

# CLAUDE.md

**fastapi-react-admin** —— 中后台底座。产品标识集中在 `apps/web/src/lib/brand.ts`
（改名字、改版本只动那一处）。

后端 fork 自 fastapi-best-architecture 并适配 SQL Server，
前端基于 React 19 + TanStack Router/Query/Table v9 + shadcn（Base UI 底座）。

架构的承重方式是榫卯：`i18n ← ui ← platform ← web` 严格单向，每层只暴露形状
（组件 / 契约 / 页面），不靠胶水互相粘。登录页那枚咬合标记（`TenonMark`）
表示的就是这个，不是项目名。

**这份文档是踩坑记录与硬纪律的唯一一份。** 全是实测出来的结论，
不是风格偏好 —— 违反了会坏，而且多数是**静默**地坏。

## 按任务导航

**这份根文件只放跨模块的东西**（结构 · 起服务 · 硬纪律）。模块级的踩坑记录
拆进了各自目录的 `AGENTS.md` —— Claude Code 读到那个目录下的文件时会**自动**
把它加载进上下文，不用你手动指。人要查的话按下表直接翻：

| 我要… | 读 |
|---|---|
| **第一次上手** | 本文件的「结构」·「本地起服务」·「硬纪律」（必读） |
| 加列表页 / CRUD 页 · 主从页 · 监控页 | [`packages/platform/src/pages/AGENTS.md`](packages/platform/src/pages/AGENTS.md) |
| 加设置屏 —— 骨架本身 / 参数配置那种 | [`_shared` 分册](packages/platform/src/pages/_shared/AGENTS.md) / [`config` 分册](packages/platform/src/pages/config/AGENTS.md) |
| 动多页签 / 标签条 / 偏好设置 / 侧边栏 | [`packages/platform/src/shell/AGENTS.md`](packages/platform/src/shell/AGENTS.md) |
| 动文件上传 / 预览 / 附件 | [`packages/platform/src/pages/file/AGENTS.md`](packages/platform/src/pages/file/AGENTS.md) |
| 动查询区 / 筛选条件 | [`packages/ui/src/components/query-bar/AGENTS.md`](packages/ui/src/components/query-bar/AGENTS.md) |
| 动富文本 / 正文里的图片 | [`packages/ui/src/components/rich-text/AGENTS.md`](packages/ui/src/components/rich-text/AGENTS.md) |
| 挑组件 / 改尺寸覆盖不生效 | [`packages/ui/AGENTS.md`](packages/ui/AGENTS.md) |
| 加文案 / 动多语言 | [`packages/i18n/AGENTS.md`](packages/i18n/AGENTS.md) |
| **显示时间 / 动时区** | [`packages/i18n/AGENTS.md`](packages/i18n/AGENTS.md) 的「服务端时间一律过 `src/datetime.ts`」 |
| 动后端模型 / 接口 / SQL · 跑 pytest | [`apps/api/AGENTS.md`](apps/api/AGENTS.md) |
| 动定时任务 / Celery / 调度 | [`apps/api/backend/app/task/AGENTS.md`](apps/api/backend/app/task/AGENTS.md) |
| 写或跑前端 E2E | [`apps/web/e2e/AGENTS.md`](apps/web/e2e/AGENTS.md) |
| 动菜单 / 权限 / 死链判定 | 硬纪律 6 + [`pages/menu/AGENTS.md`](packages/platform/src/pages/menu/AGENTS.md) |

每个目录下有一对 `AGENTS.md`（真身）+ `CLAUDE.md`（指向它的符号链接）：
Claude Code 只认 `CLAUDE.md`，其余 agent 工具认 `AGENTS.md`，一份内容两边都读得到。

**加新结论时追加到离代码最近的那一份**，不要往根文件堆 —— 根文件超过约 200 行
就开始掉注意力，而模块分册是按需加载的，写多少都不占别人的预算。
写完跑 `pnpm ctx:check`（见下）。

## 让这套文档不腐烂：`pnpm ctx:check`

这份文档全是**实测出来的结论**，而结论会过期 —— 过期的方式是**静默**的：
它照旧言之凿凿地指着一个已经不存在的文件。实测样本：这套规则第一次跑起来
就抓到「别用 command.tsx」这条 —— 那个 cmdk 组件早就删了，规则却还教了很久。

所以凡是能被机器核对的断言就让机器核对，和 `i18n:check` 同一个物种：

```bash
pnpm ctx:check          # 死引用 / 死链接 / 死脚本 / 死 testid / 行数预算
```

| 规则 | 级别 | 抓什么 |
|---|---|---|
| `dead-path` | 错误 | 反引号里的文件路径在仓库里找不到 |
| `dead-link` | 错误 | markdown 相对链接指向不存在的文件 |
| `dead-script` | 错误 | 反引号里的 pnpm 脚本没有任何 package.json 声明 |
| `dead-testid` | 错误 | 提到的 `data-testid` 源码里不存在 |
| `dead-anchor` | 错误 | 正文里的章节交叉引用指向一个全仓都不存在的章节 |
| `cross-file-anchor` | 错误 | 那一节在**别的分册**里 —— 拆分册最容易留下的债，改成相对链接 |
| `empty-scope` | 错误 | `AGENTS.md` 所在目录下没有源码（模块被搬走了） |
| `budget` | 警告 | 根文件 > 400 行 / 分册 > 500 行 —— 该拆了 |

它**不**校验文字对不对（那要人读），只校验「指向的东西还在不在」。
这一层能自动守住，剩下的才值得花人的注意力。

### 这套文档怎么自己长大

**修完一个静默失败的 bug，就在同一次改动里把结论追加到最近的那份分册。**
不是「以后有空补文档」—— 隔一天就只剩「改对了」，当时那个「为什么会
静默地错」的判断没了，而那才是这份文档唯一的价值。

判据是一句话：**违反了会坏，而且多数是静默地坏。** 风格偏好、
能从代码直接读出来的事实、一次性的调试过程，都不写。

写的时候用 `/ctx` 技能（`.claude/skills/ctx/SKILL.md`），它管三件事：
挑分册 · 按这里的文体写（症状 / 根因 / 修法 / **实测证据** 四件套）· 收尾跑校验。

## 结构

```
apps/api/          FBA fork（Python，uv 管理）
apps/web/          业务应用；routes/ 只声明 schema/守卫，不渲染页面
packages/i18n/     多语言包：语言文件 · i18next 实例 · 校验脚本（最底层）
packages/ui/       shadcn 原语，零业务
packages/platform/ 平台能力：api-client · auth · shell · pages
```

依赖方向单向：**`i18n` ← `ui` ← `platform` ← `apps/web`**。
**`ui` 永远不 import `platform`；`i18n` 不 import 任何 workspace 包**（连
`react-i18next` 都不依赖 —— 它要保持框架无关，React 绑定在 app 层注入）。

## 本地起服务

```bash
docker start fba_mssql fba_redis          # SQL Server :1433 / Redis :6380
pnpm dev                                  # api :8088 · web :8888 · celery worker（含内嵌 beat）
```

`apps/api` 和 `apps/worker` 都是 pnpm workspace 成员（`package.json` 里只有一个
`dev` 脚本、零 JS 依赖），所以 `turbo dev` 会同时起**三个**进程，TUI 里各一个日志窗格。
单起某一个：

```bash
pnpm --filter api dev                     # 等价于 cd apps/api && uv run python -m uvicorn ...
pnpm --filter web dev
pnpm --filter worker dev                  # celery worker -B（worker + 内嵌 beat）
```

`apps/worker` 里**没有代码**，只有那一个脚本 —— 存在的理由是让 worker 的日志有
自己的窗格。写进 `apps/api` 的 `dev` 里（`uvicorn & celery & wait`）也能跑，
但两份日志会挤在一个窗格里交替刷，而 worker 的日志很密。

🔴 **`-B`（内嵌 beat）只用于开发。** 多副本部署时每个副本都会跑一个 beat，
同一条调度被触发 N 次。生产要分开，且 beat **只起一个**：

```bash
pnpm --filter api celery:worker           # 可以多副本
pnpm --filter api celery:beat             # 只能一个
```

⚠️ 前端端口固定在 **8888**（`vite.config.ts` 的 `server.port` + `strictPort: true`）。
**换端口要同时改三处**，只改一处的失败方式都不长得像端口问题：

| 改哪里 | 漏了的表现 |
|---|---|
| `apps/web/vite.config.ts` | —— |
| `backend/core/conf.py: CORS_ALLOWED_ORIGINS` | 页面能开，但**所有接口 CORS 失败** |
| `backend/plugin/oauth2/plugin.toml` 的两条 `OAUTH2_FRONTEND_*_REDIRECT_URI` | 第三方授权成功后**回跳到空端口** |

`strictPort: true` 是刻意的：不写它 Vite 会在端口被占时自己 +1 漂到 1126，
而上面两处白名单是写死的 —— 宁可起不来，也不要「起来了但接口全挂」。

账号 `admin` / `123456`。登录要过验证码，验证码答案在 Redis：
`docker exec fba_redis redis-cli --raw GET "fba:login:captcha:<uuid>"`。

后端契约改动后跑 `cd packages/platform && pnpm gen:api` 重新生成 `schema.d.ts`。

---

## 硬纪律（违反会坏，不是风格问题）

### 1. 平台页面组件必须 router-独立

`params` / `search` **只能走 props**，页面内部不得调用

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yilecoding/fastapi-react-admin](https://github.com/yilecoding/fastapi-react-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
