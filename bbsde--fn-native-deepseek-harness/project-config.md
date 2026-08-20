---
trigger: always_on
description: 把 DeepSeek Harness（dsh，AI Agent 框架）打包成飞牛 fnOS 原生应用（.fpk）的工程。
---

# AGENTS.md — fn-native-deepseek-harness

把 DeepSeek Harness（dsh，AI Agent 框架）打包成飞牛 fnOS 原生应用（.fpk）的工程。
上游：https://github.com/deepseek-ai/deepseek-harness （MIT，npm 包 `@deepseek-ai/dsh`）。

命名约定：**应用标识一律用 `dsh`**（appname、网关前缀 `/app/dsh`、运行用户 `dsh`、共享目录
`dsh/workspace`、显示名 `DS·H`）；只有仓库名保留全称 fn-native-deepseek-harness。

## 架构（为什么长这样）

上游 dsh 的 Web UI **拒绝绑定 127.0.0.1 以外的地址**（CLI 直接报错，防 RCE 暴露），
且自身无任何登录认证。因此：

```
浏览器 → fnOS 统一网关 /app/dsh（NAS 登录态，转发 X-Trim-* 头）
        → Unix socket /var/apps/dsh/target/app.sock（实际在 /vol1/@appcenter/dsh/app.sock）
        → relay（src/app/bin/relay.mjs，Node）
            - 校验 X-Trim-Isadmin === 'true'，否则 403（此入口等价于主机 shell，管理员专用）
            - 剥 /app/dsh 前缀
            - Host/Origin/Referer 重写为 127.0.0.1:3080（通过 dsh 的 browser-trust fence）
            - 删 accept-encoding 后对 HTML 做运行期重写（__DSH_BOOT__ 注入的 /plugins/ URL 前缀化）
            - 注入 crypto.randomUUID polyfill（fnOS 桌面经 HTTP+局域网 IP 访问是非安全上下文，
              该 API 不存在，dsh 前端拿它生成 RPC 关联 ID——工作区选择器会因此报错）            - 代理 WebSocket 升级（/api/events.mux、/api/events.host）
        → dsh web（127.0.0.1:3080，永远只绑回环）
```

数据布局（**不要**把 DSH_HOME 放共享目录）：

- `DSH_HOME=$TRIM_PKGVAR/dsh`（即 `/vol1/@appdata/dsh/dsh`）：`.credentials.yaml`
  （上游强制 owner-only 权限位，放共享目录会拒绝启动）、profiles/（可执行插件代码）、会话与设置。
- dsh 运行时整树以单文件 `src/app/runtime.tar.gz` 进 fpk（33k 文件打成 1 个，安装秒级），
  `cmd/install_callback`/`upgrade_callback` 在安装/升级时解压到 `$TRIM_PKGVAR/runtime`，
  cmd/main 从那里启动 dsh（解压失败的报错会指向重装）。
- 共享目录 `dsh/workspace`（data-share 声明，实际在 `/vol1/@appshare/dsh/workspace`）：
  agent 工作目录。cmd/main 启动 dsh 前 `cd` 进去，新会话 cwd 默认取 `process.getcwd()`，
  产出文件天然落在共享区，文件管理器可见。
- dsh 进程的 `HOME` 也指向共享 workspace（目录选择器默认列 `os.homedir()`，不设 HOME 会
  落到不存在的 `/home/dsh` 报 ENOENT）；npm/XDG 缓存重定向到 `$DSH_HOME_DIR` 下避免
  点文件污染共享区。

关键 TRIM_ 环境变量（实测值）：`TRIM_APPDEST=/vol1/@appcenter/dsh`、
`TRIM_PKGVAR=/vol1/@appdata/dsh`、`TRIM_DATA_SHARE_PATHS=/vol1/@appshare/dsh/workspace`。
`/var/apps/dsh/shares/workspace` 是共享目录的软链（注意是 `shares` 不是 `share`）。

## 目录

```
src/                 # fnpack 打包根（manifest、config/、cmd/、app/bin/relay.mjs、app/ui/）
cache/dsh-runtime-x86_64/  # 构建缓存：x86_64 的 dsh node_modules（勿手改，勿提交）
cache/dsh-runtime-arm64/    # 构建缓存：arm64 的 dsh node_modules（仅 CI/原生 arm 主机产生）
src/app/runtime.tar.gz  # 构建期生成：runtime 整树单文件 tar（33k 文件打包成 1 个，
                      #   安装秒级；cmd/install_callback 解压到 $TRIM_PKGVAR/runtime）
src/app/bin/seed-market.mjs  # 启动期种子：市场插件的 shim/软链/profile 种子
src/app/bin/profile-salvage.mjs  # lastgood 快照 + 看门狗的外科手术式恢复（见测试生命周期节）
src/app/bin/catalog-cache.mjs  # 市场目录的本地 stale-while-revalidate 缓存（回环，见内置插件市场节）
src/app/bin/supervise-web.sh  # 常驻监督循环：托管重启 + web/relay 崩溃自愈（见测试生命周期节）
scripts/             # fetch-dsh / rewrite-dist / build / 本地与真机测试脚本
package.json         # dshVersion / pnpmVersion 钉死两组上游版本（市场本体在线装，不钉）
assets/ICON.png      # 图标母版 600x600；make-icons.mjs 导出 @2x（64pt→128px、
                      #   256pt→512px，fnOS 桌面按 HiDPI 2x 渲染）。build.sh 带
                      #   新鲜度守卫：母版比导出图新则打包时自动重生成。
```

## 构建（必须理解远程安装的原因）

```bash
./build.sh              # 自动取 npm 上游最新版 → 钉版 → 远程安装 → 重写 → fnpack → dist/
./build.sh 0.1.0-rc.6   # 构建指定上游版本
npm run build           # 等价于 build.sh 的钉版路径（不查 npm、不拷 dist）
```

- `./build.sh` 是主入口：**fpk 版本镜像上游 dsh 版本**（manifest `version=` = dshVersion，
  如 `0.1.0-rc.6`），装到设备上看到的应用版本即所带上游版本。同一上游的纯封装修复
  （relay/脚本改动）重新发布时用 `DSH_WRAPPER_BUILD=1 ./build.sh`（版本变
  `0.1.0-rc.6.1`）。入口为浏览器新标签页打开（ui/config `type: "url"`），不是桌面 iframe。
  **`DSH_WRAPPER_BUILD` 的值就是修订号后缀**（`=11` → `0.1.0-rc.7.11`）。
  **封装修订号不能跨 9→10 进位**：fnOS 部分安装路径（桌面 UI 的手动安装/升级）按字符串比较
  版本，`0.1.0-rc.7.10/7.11` < `0.1.0-rc.7.9`（'1'<'9'）会被拒"不符合系统要求"，且拒绝
  发生在客户端、journal 无任何 APP_ 事件；CLI install-fpk 不做此检查。修订号到 9 之后再
  发版要跳到对字符串和数值比较都更大的段（如 `.90` 起），`.12`→`.13` 这类不跨 9 的递增安全。
  输出 `dist/dsh_<版本>.fpk`（附 .info.txt），并把所用上游版本写回 `package.json` 的
  `dshVersion`（钉版是唯一上游版本来源，**精确钉死**，rc 阶段破坏性变更多）。
  同版本重复构建走快速路径（跳过远程安装与重写——rewrite-dist 带幂等预检）；换新版本自动
  走全流程，**若上游打包方式变化，重写门禁会让构建大声失败**，此时按门禁报错更新规则集再重跑。
- **工作区就在 fnOS 机器上时（HOME-NAS：/vol3/1000/Projects/fn-native-deepseek-harness）**：
  `DSH_BUILD_HOST=local DSH_WRAPPER_BUILD=1 ./build.sh 0.1.0-rc.6`，fetch 直接在本机
  nodejs_v24 下装进 `cache/`，不再走 SSH。npm 缓存重定向到 `cache/npm-cache`（本机
  shell 的 `npm_config_cache` 指向 `$DSH_HOME/.npm-cache`，构建树必须绕开）。注意：
  在这台机器上装的 dsh 里开的会话，其工作区若就是本仓库，**升级安装会杀掉会话**——
  先出包、择机装，装完开新会话接续；`sudo appcenter-cli` 只能由管理员在宿主 shell 执行。
- **npm install 必须在 Linux x64 上执行**：dsh 有原生依赖 node-pty（需编译或预编译产物）
  和 koffi（install 脚本装原生模块），Windows/`--ignore-scripts` 装出的树在 fnOS 上必崩
  （症状：plugin tree failed to load / pty.node not found / Koffi missing）。
  `fetch-dsh.mjs` 通过 SSH 在构建机（`DSH_BUILD_HOST`，默认 nas31）上用设备同款
  nodejs_v24 运行时安装，tar 回传（保符号链接），并校验 pty.node 是 Linux ELF。
- nas31 需要一次性装好工具链：`sudo apt-get install -y g++ make python3`（node-pty 编译用）。
- **npm registry**：nas31 远程路径与 fnOS 本机路径默认走 npmmirror（`registry.npmmirror.com`，
  node-gyp 头文件同步走镜像）——CN 网络下 npmjs 直连一个 535 包的冷安装要 ~10 分钟，
  镜像把下载瓶颈消掉后剩 node-pty 编译本身；CI 的美国 runner 保持 npmjs。
  `DSH_NPM_REGISTRY` 可覆盖。`pack-runtime.mjs` 带新鲜度跳过：cache 里现成 tar 比整棵
  staging 树都新就不重打（33k 文件重打一次要几分钟）——同依赖的封装修订重建因此只要
  秒级（fetch/rewrite/pack 三段全跳，只剩 fnpack）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bbsde/fn-native-deepseek-harness](https://github.com/bbsde/fn-native-deepseek-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
