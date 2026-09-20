---
trigger: always_on
description: - 已发布版本以 [package.json](package.json) 的 `version` 为准 → <https://github.com/zlZayn/dsh-zhihu-search>
---

# dsh-zhihu-search — 维护索引

## 状态

- 已发布版本以 [package.json](package.json) 的 `version` 为准 → <https://github.com/zlZayn/dsh-zhihu-search>
- npm → <https://www.npmjs.com/package/dsh-zhihu-search>（由 [release.yml](.github/workflows/release.yml) 手动触发，一次跑完 bump → 发布 → tag → GitHub Release）
- 功能、插件页配置卡片与本地真机验证全部完成；工具清单见 [README.md](README.md)。
- 市场收录：仓库已带 `dsh-plugin` topic；`awesome-dsh-plugin` 的 [PR #5037](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin/pull/5037) 已提，等评审 → [笔记](.agents/notes/2026-09-12-marketplace-submission.md)

## 全局规则

- 结论必须来自实测，不得来自文档推断 → [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- 红线以测试固化，共五条（依赖分层、呈现隔离、模型上下文隔离、无全局状态、模型不见原始语法）→ [test/README.md](test/README.md)
- 对外可见行为变化，同一次改动内同步 [README.md](README.md) 与 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- 改根 [README.md](README.md) 必同改 [README_en.md](README_en.md)，冲突以中文为准
- 「版本兼容」章节只讲分水岭与真源指针，**不抄会漂的宿主版本**；判据：`engines.dsh` 一旦落后于实际部署的宿主线（[compat.yml](.github/workflows/compat.yml) 的 declaration 作业转红即为信号），该章与两份 README 的「前置」必须同批复核
- 决策理由 → [.agents/notes/](.agents/notes/)
- 发版授权：patch / minor 按 [docs/PUBLISHING.md](docs/PUBLISHING.md) 的问题链定档后**直接发**；**major 必须先问人类**；**零行为变更不发版**（纯文档 / 测试 / CI / 等价重构）

## 事实来源（只查不抄）

- 版本号、依赖、宿主兼容性 → [package.json](package.json)
- 工具参数、默认值与上限 → [src/tools/](src/tools/) 的常量与 [src/transport.ts](src/transport.ts) 的端点契约常量（上限只住后者，工具层取别名），由 [test/tool.test.ts](test/tool.test.ts) 守护
- 端点语法与实测偏差 → [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- 构建产物的运行方式 → [docs/PUBLISHING.md](docs/PUBLISHING.md)
- DSH 平台知识（插件装配、Remote API、设置卡片、i18n、工具契约）→ [DSH cookbook](https://github.com/deepseek-ai/deepseek-harness/tree/master/docs/cookbook)，每篇都有 `.zh.md` 中文版
- **有自更新来源的事实一律指向来源**：测试与类型检查 → Actions；发布版本 → npm badge；产物一致性 → 哈希比对（见 [docs/PUBLISHING.md](docs/PUBLISHING.md) 的「打包内容」）。别在本文件留快照数值。

## 常用命令

- `npm run build`（host tsc + client tsc + esbuild）· `npm run typecheck` · `npm test`（先 build 再 vitest）· `npm run test:contract`（打真实接口，需 `ZHIHU_ACCESS_SECRET`，日常 CI 不跑）
- 真机验收：`ZHIHU_ACCESS_SECRET=xxx node scripts/acceptance.mjs [包目录]` —— 默认验 profile 里装的那份，覆盖真实接口 + 宿主 schema 校验 + 渲染文本，见 [scripts/README.md](scripts/README.md)
- 发版：`gh workflow run release.yml -f tier=patch|minor|major` —— 唯一入口，档位按 [docs/PUBLISHING.md](docs/PUBLISHING.md) 的问题链定；无行为变更时 [守卫](scripts/release-guard.mjs) 会拦下（`-f force=true` 才能越过）
- 兼容性换包（本地复现 [compat.yml](.github/workflows/compat.yml)）：`node scripts/compat-swap.mjs swap next` → `npm install --ignore-scripts` → `node scripts/compat-swap.mjs verify next`。**它会改写 `package.json`**，只在一次性 clone 里跑；声明面单独查用 `check next`

## 验证快照（2026-09-16 实跑）

数字与版本一律看自更新来源（理由见「[文档网络与自更新](#文档网络与自更新)」）：测试与类型检查 → [Actions](https://github.com/zlZayn/dsh-zhihu-search/actions)，发布版本 → [npm](https://www.npmjs.com/package/dsh-zhihu-search)。下面只记不随数字漂移的定性结论；更早轮次的真机验证见 [.agents/notes/](.agents/notes/) 与 `git log`。

- 契约测试：本机与 GitHub Actions 都实跑通过（后者由 repo secret `ZHIHU_ACCESS_SECRET` 供燃料）；每周一由 [contract.yml](.github/workflows/contract.yml) 跑；日常 `npm test` 不含 live 探针，只多一份底座的离线自检
- 真机验收：[scripts/acceptance.mjs](scripts/acceptance.mjs) 覆盖扩池 / 输出对称 / www 归一化 + 宿主 schema 校验 + 渲染文本；升级 + host 重启后在 profile 安装副本上跑通，工具面同参数复验一致
- 发版守卫：只有文档 / 工具脚本改动的区间在 `npm ci` 之前被拦下（后续步骤全 skipped，npm 侧零动作）；含 `src/` 的区间正常放行
- 诚实渲染：到顶必说 / 来源构成分流 / 空态首句条件限定由 [test/presentation.test.ts](test/presentation.test.ts) 固化；`count` 回满上限不额外提示（刻意防噪音）
- 宿主兼容性：由 [compat.yml](.github/workflows/compat.yml) 每周对 `next`（承诺线）与 `alpha`（前瞻线）换包，跑的是现有套件、不写新测试；结论与处理链归 [docs/PUBLISHING.md](docs/PUBLISHING.md) 的「兼容性」。这里只留定性结论：**类型面会先于行为面动** —— alpha 线上类型面已红而 260 个测试全绿，所以「测试全绿」不能当作「兼容」的结论
- 明文迁徙：**真机跑通**（2026-09-16）—— 装入 1.6.2 + 重启 host 后，`settings.yaml` 的 `zhihu-search:` 段只剩 `disableNativeWebSearch`，值（40 位十六进制、与原明文逐字一致）落进 `.credentials.yaml` 的 `refs`，两个文件同一秒被改写。此前在 `lib/` 产物 + 真实 provider + 本机 `settings.yaml` **副本**上也跑通过（段内清理、其他 section 与注释原样保留、第二次运行是空操作）

## 待办

- 清理旧明文通道：等使用者跨过当前版本后，删 `Config.accessSecret` 与 [src/migrate.ts](src/migrate.ts)（**必须一起删**，理由见 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) 的「`accessSecret` 为什么仍留在 schema 里」）
- **声明面已落后于实际部署**：本机宿主跑在 **alpha 线**上（比 next 线新），而 `peerDependencies` 只到 **next 线**的版本 —— 预发布区间取不到 alpha 上的版本。等 alpha 切到 next（或发正式版）时按 [docs/PUBLISHING.md](docs/PUBLISHING.md) 的「兼容性」放宽范围、同步两份 README，并按 Q1/Q2 定档；**放宽时一并定下限** —— 抬到新 next 即放弃旧下限，保留它则它落在两条 lane（next / alpha）之外，无人覆盖。**新下限不得低于引入 `plugins.bundle.config` 的那个版本**（`engines.dsh` 现在声明的下限就是它）：配置面只向该槽注册，落在更早的宿主线上装上去，配置页**静默不出现**。
- 处置 `@deepseek-ai/dsh-code-runtime`：`devDependencies` 里**没有任何文件引用它**，且它在 alpha 线上停在一个比 next 线还旧的版本（现查 `node scripts/compat-swap.mjs check latest`）—— 换包脚本因此每个 alpha 轮都要告警跳过它一次。删掉即消失
- **npm 上已发布版本的头图会断**：npm 页面按 `main`（HEAD）取 README 里的图，而 v1.6.3 及更早的 README 写的是 `assets/cover.svg` —— 该文件已随头图换新（`banner.svg`）删除。**最新**那页会随下次发版自动修好；更早版本的页面文字在发布时就定死，除非把 `cover.svg` 补回。下次发版后顺手看一眼 npm 页面头图即可

## 活跃坑（工具链与 DSH 平台）

知乎 API 自身的反直觉处归 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)；**模块内的坑下放到对应子目录的 `AGENTS.md`**（在那里工作时自动注入），此处只留跨模块、踩了整条链就崩的几条。

- **生效链取决于 profile 怎么挂的，先查再假设**：`Get-Item <profile>\node_modules\dsh-zhihu-search | Select LinkType,Target`。
  - **符号链接到仓库**：`npm run build`（含 `npm test` 的 build）**直接写线上**，`dsh-client-hmr` 轮询 `lib/client.js` 当场换掉**浏览器半体**；host 半体要重启才换。改客户端半体因此免发版即生效，但**构建即上线** —— 没验证过的构建会立刻影响正在用的界面。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zlZayn/dsh-zhihu-search](https://github.com/zlZayn/dsh-zhihu-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
