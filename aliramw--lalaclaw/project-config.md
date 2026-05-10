---
trigger: always_on
description: 开发态需要同时启动前端和后端，不使用 `dist/`。
---

# AGENTS.md

## 开发服务启动 / Dev Servers

开发态需要同时启动前端和后端，不使用 `dist/`。  
Run both frontend and backend in development. Do not use `dist/`.

### 前端开发服务 / Frontend

在项目根目录运行：

```bash
npm run dev -- --host 127.0.0.1 --port 5173 --strictPort
```

前端访问地址 / Frontend URL:

```text
http://127.0.0.1:5173
```

### 后端开发服务 / Backend

在项目根目录运行：

```bash
PORT=3000 HOST=127.0.0.1 node server.js
```

后端接口地址 / Backend URL:

```text
http://127.0.0.1:3000
```

### 联调说明 / Notes

- 页面入口使用 `http://127.0.0.1:5173`。Use `http://127.0.0.1:5173` as the main dev entry.
- `vite.config.mjs` 已将 `/api/*` 代理到 `http://127.0.0.1:3000`。`vite.config.mjs` already proxies `/api/*` to `http://127.0.0.1:3000`.
- 不要用 `npm start` 做前端联调，它依赖已有 `dist/`。Do not use `npm start` for frontend dev verification.
- 默认自动连接本地 OpenClaw。The app auto-detects local OpenClaw by default.
- 强制 `mock` 模式：`COMMANDCENTER_FORCE_MOCK=1 PORT=3000 HOST=127.0.0.1 node server.js`。

## 版本维护 / Versioning

- 版本号变更时必须同步更新 `CHANGELOG.md`。Update `CHANGELOG.md` whenever the project version changes.
- 发布版本必须保持 npm 兼容。同一天的第 N 个版本使用 `YYYY.M.D-N`，例如 `2026.3.17-2`，不要使用 `YYYY.M.D.N`。Release versions must stay npm-compatible. For the Nth release on the same day, use `YYYY.M.D-N`, for example `2026.3.17-2`, not `YYYY.M.D.N`.
- `CHANGELOG.md` 需明确记录新增、修改、修复和重要行为变化。Record additions, changes, fixes, and important behavior changes clearly.

### 发布顺序 / Release Order

- 发布时按固定顺序执行，避免版本号、tag、npm 包和 GitHub release 中间状态不一致。Follow a fixed release order to avoid mismatches between the version, tag, npm package, and GitHub release.
- 先把版本号从当前版本 bump 到目标版本，例如 `2026.3.17-5` -> `2026.3.17-6`。First bump the version from the current release to the target release, for example `2026.3.17-5` -> `2026.3.17-6`.
- 写 `CHANGELOG.md`，并同步更新 `README`、`documentation-quick-start` 等文档里的示例版本号。Update `CHANGELOG.md`, then sync example version numbers in `README`, `documentation-quick-start`, and related docs.
- 跑一轮关键测试和构建，至少覆盖 release 前最关键的 lint、test、build。Run the key validation steps before release, at minimum the critical lint, test, and build commands.
- 在 `npm publish` 之前，必须验证“将要发布的 npm 产物”，不能只验证源码工作区。Before `npm publish`, validate the actual npm artifact that will be released instead of only validating the source checkout.
- 发布前必须执行 `npm run pack:release`，让 tarball 输出到 `artifacts/`，并在干净临时目录里安装该 tarball 做一次安装态验证。Run `npm run pack:release` before release so the tarball lands in `artifacts/`, then install that tarball in a clean temporary directory for installed-package validation.
- 安装态验证至少要覆盖一次真实启动路径：能启动服务、首页不白屏、浏览器 console 无新的 runtime error。The installed-package validation must cover one real startup path: the app starts, the first screen is not blank, and the browser console shows no new runtime errors.
- `vite build`、打包日志或安装态 smoke test 中出现 `Circular chunk`、chunk 初始化错误、首屏空白等信号时，视为 release blocker，不得继续发布。Treat `Circular chunk`, chunk-init failures, blank-first-screen symptoms, or similar build/install smoke signals as release blockers.
- 任何修改 `vite.config.*`、`manualChunks`、构建产物入口、Mermaid/Monaco/预览器等打包敏感区域的改动，发布前都必须补跑一次安装态 smoke test。Any change touching `vite.config.*`, `manualChunks`, bundle entry behavior, or packaging-sensitive areas such as Mermaid, Monaco, or preview pipelines must rerun the installed-package smoke test before release.
- 验证通过后再提交并推送到 `origin/main`。Only commit and push to `origin/main` after those checks pass.
- 推送完成后再创建 Git tag 和 GitHub release。Create the Git tag and GitHub release only after the main branch has been pushed.
- 最后发布 npm 包。Publish the npm package last.
- 每次发布新版本时，必须单独询问维护者这次是否要给该版本打 `stable` dist-tag，不能默认自动打。For every new release, explicitly ask the maintainer whether this version should receive the `stable` dist-tag; never assume `stable` promotion automatically.
- 如果维护者还没有明确确认，默认只发布版本本身，或先发布到非 `latest` / 非 `stable` 的 dist-tag（例如 `next`），不要擅自改动 `stable`。If the maintainer has not explicitly confirmed, publish the version itself only, or use a non-`latest` / non-`stable` dist-tag such as `next`; do not change `stable` on your own.
- 如果维护者确认要打 `stable`，优先用 `npm dist-tag add lalaclaw@<version> stable` 提升一个“已经发布过”的版本，而不是为了改 tag 重发同版本。If the maintainer confirms `stable`, prefer promoting an already-published version with `npm dist-tag add lalaclaw@<version> stable` instead of republishing the same version only to change tags.
- 如果维护者确认本次不应再保留某个版本的 `stable` 标记，可以用 `npm dist-tag add lalaclaw@<older-version> stable` 把 `stable` 挪回旧版本，或在确有需要时用 `npm dist-tag rm lalaclaw stable` 移除 `stable`。If the maintainer decides a version should no longer be `stable`, move `stable` back to an older version with `npm dist-tag add lalaclaw@<older-version> stable`, or remove the tag with `npm dist-tag rm lalaclaw stable` when that behavior is explicitly desired.

### 发布产物验收 / Release Artifact Validation

- 推荐顺序：
  - `npm run lint`
  - `npm test`
  - `npm run build`
  - `npm run pack:release`
- 之后在干净目录中至少执行一次：
  - `npm install ./artifacts/lalaclaw-<version>.tgz`
  - 通过发布包的真实入口启动应用，而不是回到源码目录复用开发环境
- 验收时至少确认：
  - 安装后的应用能成功启动
  - 首屏能正常渲染，不出现白屏
  - 浏览器 console 没有新的 runtime / chunk 初始化错误
  - 如果本次改动涉及生产打包或懒加载分块，优先再检查一次 Network / console，确认没有循环 chunk 或缺失 chunk
- 如果需要更稳的发布节奏，可先用非 `latest` dist-tag 做一次 registry 验证，通过后再切到正式标签。If needed, publish to a non-`latest` dist-tag first, verify from the registry, and only then promote it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aliramw/lalaclaw](https://github.com/aliramw/lalaclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
