---
trigger: always_on
description: 给在这个仓库里干活的 AI 的说明书，人看也欢迎。**动手前先读「硬规矩」与「关键机制」两节。**
---

# AGENTS.md

给在这个仓库里干活的 AI 的说明书，人看也欢迎。**动手前先读「硬规矩」与「关键机制」两节。**

## 这是什么

DSH-X：DeepSeek Harness（`dsh`）的第三方启动器。仓库叫 DSH-X，`package.json` 里的名字是更早的 `dsh-versions`。它启动的是 dsh 官方原版 Web 界面，自己不重做 dsh 的页面。

```
原生外壳   launcher/（Rust）        DSH.exe / DSH-X.app：托盘、窗口、开机自启、单实例
Node 服务  start.js → server.js     管理页后端 + 版本 / 插件 / 整合包 / 同步的全部业务
管理界面   public/index.html        单页手写 HTML + 原生 JS，无框架、无构建步骤
注入钩子   compat/  perf/           被注入 dsh 进程，修补运行时行为（见下）
```

**零第三方运行时依赖**：所有 import 都是 `node:` 内置或相对路径，没有构建步骤。安装包自带 Node 运行时（打包时下载到 `vendor/`），所以不依赖用户机器环境。

## 怎么跑、怎么验

| 命令 | 干什么 |
|---|---|
| `npm start` | 起管理页（开发入口） |
| `npm run server` | 只起网页服务，不开浏览器 |
| `npm test` | 全部用例（`node --test`，41 个文件、300+ 用例，约 16 秒） |
| `node --test test/packs.test.mjs` | 单个用例文件 |
| `npm run dist` | 打安装包（Windows 需 Rust + Inno Setup 6；macOS 需 Rust + Xcode CLT） |
| `node scripts/make-pack.mjs` | 把 `packs/<名字>/` 打成 `.dspack`（产物在 `release/packs/`） |

- **跑测试别把输出接管道**（`| head`、`| tail` 会把输出缓冲成 0 字节，等于白跑）；要留证据就重定向到文件。
- 没有 lint / 格式化配置，**也别加**：风格靠已有代码和这份文档。
- 用例里有三只"假服务器"（`test/fake-s3.mjs`、`fake-webdav.mjs`、`fake-proxy.mjs`），网络相关改动靠它们离线验证。

## 硬规矩

- **改完不要自动 commit、不要自动 `npm run dist`、不要发版。** 改动留在工作区，等用户明确指令；用户习惯自己按功能拆提交。
- commit message 用**中文**。
- README 中英成对改（`README.md` + `README.en.md`）；**改 README 默认是"砍"不是"加"**——用户会周期性说"写得太多"。
- Release notes 照 dsh 官方风格：一行一条、分 2–4 块、挂 `@贡献者`；不写根因、验证过程、安装说明。QQ 群公告压到 400 字符内（只写用户看得见的变化，下载只留落地页一条链接），存一份到 `release/群公告-<版本>.txt`。
- 注释写**为什么**（这个仓库的风格：每个不显然的决定都带一段解释，很多是踩坑记录），用中文；不写逐行翻译代码的注释。
- `release/release-key.pem` 是发版签名私钥：不进仓库、别提交、别把内容打印到日志；动相关流程时提醒用户备份。

## 目录地图

| 路径 | 是什么 |
|---|---|
| `start.js` | 入口：端口与日志、被原生外壳拉起时的窗口信号（`__DSH_SHOW__`） |
| `server.js` | 管理页后端 + 全部业务（最大文件）：HTTP 路由、版本管理、插件安装、整合包、启动失败自愈 |
| `settings.js` | 设置读写；`safe*` 一族是校验函数（端口、路径、代理、语言……都从这过） |
| `registry.js` / `version.js` | dsh 版本发现、pnpm 交互（进度解析）、版本号比较；装版本失败时把 npm 的收尾报错换成具体提示（`describeNpmFailure`：ETARGET 发版不齐 / 404 没有这个包） |
| `plugins.js` | 插件开关：直接改 profile 的 `cordis.patch.yml`，不需要 dsh 在跑 |
| `mcp.js` / `skills.js` | MCP server 与技能的读写、探测、开关 |
| `packs.js` | 整合包：`.dspack`（manifest v5 ZIP）的解析 / 检查 / 安装 / 导出 / 回滚 |
| `zip.js` / `zipfile.js` | zip 的写与读（手写，无依赖） |
| `sync.js` | 同步引擎：S3 / WebDAV / 本地目录 / ZIP 四种后端共用一个"远端"抽象 |
| `proxy.js` | `netFetch`：Node 全局 fetch 不看系统代理，所有外网请求走它 |
| `platform.js` | 平台差异集中地（每用户目录、运行时布局、外壳文件名）；**打包脚本必须跟它保持一致** |
| `plugin-tool.js` | 命令行开关插件（dsh 起不来时的救命工具），有可执行位 |
| `stdio-unblock.cjs` | stdout/stderr 阻塞模式修补（`--require` 注入用） |
| `public/` | 管理界面与看板娘素材（`index.html` 是主体） |
| `scripts/` | 打包（`pack.mjs` / `pack-mac.mjs` / `pack-common.mjs` / `dsh-setup.iss`）、图标（`make-icons.py`）、落地页素材、发版清单与 SBOM |
| `plugins/` `packs/` | 插件源码（发 npm）与随包发的整合包配方（见「关键机制」，**两个目录别合并**） |
| `compat/` `perf/` | 注入 dsh 进程的钩子：兼容修补与加速 |
| `launcher/` | Rust 外壳（`main.rs`、托盘 / 窗口 / 自更新） |
| `test/` | 用例与假服务器 |
| `docs/` | GitHub Pages 落地页 + 看板娘设计稿（`mascot-design/`） |
| `assets/` | 图标 master（`icon.png` → `make-icons.py` 出全套） |
| `data/` `release/` `vendor/` | 运行时数据与产物，gitignored，可重建，**不用整理** |

**顶层 .js 平铺是有意的，不是没整理**：安装目录也是平铺的——`scripts/pack-common.mjs` 的 `APP_FILES` 按名字把文件原样拷到安装根，`launcher/main.rs` 也在根上找 `start.js`。要把它们收进 `src/` 得同时改四处（`APP_FILES` 与拷贝目标、`platform.js`/`server.js` 的根路径推导、Rust 的查找路径、`dsh-setup.iss`）并重新打包验证，别只挪文件。

## 关键机制

### 版本、实例与数据

- 每个 dsh 版本装在数据目录（默认 `%APPDATA%\DSH\data`，设置里可改）的 `versions/<版本>/`。
- **一个「版本 × profile」一个实例**，可以同时跑，各占各的端口：默认给会起 web 的 profile 传 `--port 0`，由系统现挑，启动器再从 dsh 打印的地址里读回真实端口。
- 端口也能手工钉死（控制页那个「端口」输入框 → `settings.json` 的 `instancePorts`，键 `版本@profile`）：钉了就传 `--port <它>`，链接每次启动都一样（书签、手机上的地址才留得住）。钉住的组合起不来时（`EADDRINUSE`）报错要点名端口和占用者。**这是 dsh 实例的端口，跟管理页端口（`port`）是两回事，两者不能撞。**
- dsh 本体数据在 `DSH_HOME`（默认 `~/.dsh`，可配），跨版本共享，所以换版本不用重装插件。
- 启动器自己的设置与日志在 `%APPDATA%\DSH`（macOS 为 `~/Library/Application Support/DSH`）。

### profile

dsh 的环境隔离单位。模板名（web / headless / acp / sdk / …）会自动初始化，自定义名必须先有 `package.json`，否则 dsh 拒绝启动。插件按 profile 分，会话与记忆共享。启动器提供切换与开关；插件开关就是往 `cordis.patch.yml` 写 `- id: <行> / disabled: true`，删掉该块即恢复。

恢复档（`recovery.js`，官方 desktop `sanitizeProfile` 的同款语义）：`POST /api/recover` 把补丁层**改名**成 `cordis.patch.yml.bak-<时间戳>`（备份即改名、不解析补丁内容），再按侧车记录摘掉清单里非 `@deepseek-ai/*` 的 bundle；`POST /api/recover/restore` 把备份改回来、现役补丁先挪到新备份、并按侧车放回被摘的 bundle。改补丁层/清单前都过 `withProfileLock`（profile 目录里一个存 PID 的 `lock` 文件，`wx` 独占创建、属主僵死才清理）——与官方锁同名同语义。这是逐行自动禁用都失效时的最后手段；清单 JSON 坏掉时只告警、补丁层照样移走（救援优先，官方是整单失败）。

### 插件与整合包（两个目录，别合并）

- `plugins/<包名>/` 是**插件实现的源码与发布源**（npm 包：`dsh-x-memory`、`dsh-x-sync`、`dsh-x-aquarium`），**不随安装包发**：插件都发到 npm，用户从插件页 / 整合包装，装完能看远程版本、能更新。改了这里的代码要 `npm publish`（版本号记得加）。
- `packs/dsh-x-recommended/` 是**整合包配方**（`manifest.json` + `dspack.json` + README），**不含代码**，随安装包发；依赖一律写 npm 版本号。
- 没有「内置插件」这条路：预置到当前 profile 的开关、随包发插件的复制逻辑、整合包里的 `"…": "bundled"` 依赖解析都已撤（`planInstall` 见到 `bundled` 会直接报错）。
- 为什么不能合并：`.dspack` 只认 `overrides/`、`home/`、`profiles/` 与几个根文件，插件源码放进去会被当"不属于整合包"跳过（`packs.js` 的解析）；反过来 `plugins/` 下每个子目录都被当成一个 npm 包。

### 注入进 dsh 的钩子（compat / perf）

- `compat/register.mjs`、`perf/register.mjs` 由启动器注入；worker 线程注入的是 `compat/worker-events.cjs`（worker 的 execArgv 是空的，只能靠 NODE_OPTIONS）。
- NODE_OPTIONS 里**只写裸文件名**，目录靠 NODE_PATH 传——NODE_OPTIONS 按空格分词，安装目录带空格会被拆坏。
- dsh 自己的 Node 开关（系统证书库、请求头上限、两个 ESM 补丁钩子）走**命令行参数**，不塞 NODE_OPTIONS：后者会被 dsh 的所有子进程继承，agent 在 shell 里跑的 node 可能是老版本。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yyh-001/DSH-X](https://github.com/yyh-001/DSH-X) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
