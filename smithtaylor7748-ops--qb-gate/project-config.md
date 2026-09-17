---
trigger: always_on
description: 给接手这个项目的人（以及 AI 助手）看的硬规矩。每一条都是踩出来的，不是推理出来的。
---

# 改这个仓库之前先读这里

给接手这个项目的人（以及 AI 助手）看的硬规矩。每一条都是踩出来的，不是推理出来的。

实现层面的「为什么这么写」在 [docs/DESIGN-NOTES.zh-CN.md](docs/DESIGN-NOTES.zh-CN.md)，
修不掉的缺口在 [docs/KNOWN-ISSUES.zh-CN.md](docs/KNOWN-ISSUES.zh-CN.md)。

---

## ⛔ 改完了 = 本机那份也更新了

**任何一轮改动落地，都要把本机装着的那份一起更新掉。** 不只是修 bug ——
界面重排、加功能、改文案，一律算。

**这一步不需要使用者开口要，它是「改完了」的定义的一部分。**

仓库改好了、本机还跑着旧版，等于什么都没修 —— 而且下次排查时会对着新代码看旧行为，
得出的结论全是错的。这个坑的代价不是「少更新一次」，是**后面每一次调试都在骗自己**。

### 注意哪些命令不会更新本机那份

| 命令 | 产出 | 会不会动本机装的那份 |
|---|---|---|
| `npm run build` | 只出 `dist/` | ❌ |
| `npm run tauri dev` / `npm run dev` | 临时进程，关了就没 | ❌ |
| `npm run demo` | 同上，而且喂的是假数据 | ❌ |
| **`npm run tauri build`** | `target/release/bundle/nsis/QB Gate_<版本>_x64-setup.exe`（workspace 的 target 在仓库根） | 出安装包，**还要装** |

### ⛔ 给使用者的命令一律写 PowerShell 形式

**使用者的终端是 Windows PowerShell，不是 Git Bash。** 这不是风格偏好 ——
下面这些 bash 写法在他那里是**当场报错**：

| bash 写法 | 在 PowerShell 里的下场 | 该写成 |
|---|---|---|
| `A && B` | 语法错误（5.1 没有管道链操作符） | 逐条列出，或 `A; if ($?) { B }` |
| `sha256sum f` | `CommandNotFoundException` | `Get-FileHash f -Algorithm SHA256` |
| `/d/claude-gate/…` | 不是有效路径 | `D:\claude-gate\…` |
| `2>/dev/null` | 建出一个名叫 `null` 的文件 | `2>$null` |
| `head -n 5 f` | 没有这个命令 | `Get-Content f -TotalCount 5` |

实测栽过一次（2026-09-15）：给他的安装包校验命令用了 `sha256sum`，
他照着最显眼的那条跑，当场 CommandNotFound。

⚠ **`A; if ($?) { B }; if ($?) { C }` 是错的** —— 第二个 `$?` 反映的是上一个
`if` 语句的结果，不是 `B` 的。要串就嵌套，或者干脆逐条跑。

### 收工清单

前端六项，**逐条跑，任何一条红了就停**（PowerShell 没有 `&&`，别伪装成一行）：

```powershell
npm run build
npm test
npm run types:check
npm run format:check
npm run test:ui
npm run release:check
```

Rust 四项，同样逐条：

```powershell
cargo test --workspace
cargo clippy --workspace --lib -- -D warnings
cargo fmt --all -- --check
cargo deny check
```

出安装包（release 编译约 3–5 分钟）。**产出安装包不等于改完了，还要装** ——
见本节开头那一条：

```powershell
npm run tauri build
```

**这两组就是 CI 跑的那一套**，别只跑前两条就以为绿了 —— `test:ui`（72 个响应式/
主题组合 + 6 个交互流程）和 `cargo deny`（advisories / bans / licenses / sources）
各自抓过别处抓不到的东西。`cargo test --workspace` 的通过数**只许涨不许跌**。

然后：

1. **版本号往前提。** 不是三处，是**四个文件五个位置**，漏一个
   `npm run release:check` 当场报错：

   | 文件 | 位置 |
   |---|---|
   | `package.json` | `version` |
   | `package-lock.json` | 顶层 `version` **和** `packages[""].version` 两处 |
   | `src-tauri/Cargo.toml` | `version`（必须写字面量，release-check 拿正则读它） |
   | `src-tauri/tauri.conf.json` | `version` |
   | `Cargo.toml`（仓库根） | `[workspace.package]` 的 `version` —— 其余 13 个 crate 从这里取 |

   **权威是 `npm run release:check`，不是这张表**：表会过期，那条命令不会。
   文档里一旦引用了某个版本号，就必须真的存在那一版；
2. 装上新的安装包；
3. **装之前先确认旧的那份叫什么名字。** 改过名（ClaudeGate → QB Gate）之后，
   新安装包**不会覆盖**旧名字那一份，会变成两套并存：两个卸载项、两个快捷方式、
   两套运行期数据。先卸旧的再装新的。

---

## ⛔ 中转会话不归门禁的**关停**策略管

`LaunchTarget::gated()` 与 `LaunchTarget::stops_with_gate()` 是**两个**判断，
不许合并：

| 问题 | 谁回答 | 中转的答案 |
|---|---|---|
| 起之前要不要验 IP 解锁、起完要不要持租约 | `gated()` | **要**。Deny ACE 是按文件加的，不认身份；让中转绕过解锁，就是拿中转会话把 claude.exe 解锁、再从终端起官方的 —— 现成的绕过入口 |
| 门禁判不过时要不要收掉这个会话 | `stops_with_gate()` | **不收**。中转请求打第三方端点、用你自己买的 Key、不带官方 OAuth 身份，Anthropic 那边看不见。收它换不到任何保护，却会把正在写的对话弄丢 —— 而中转站的典型使用者恰恰就是出口 IP 会变的那群人 |

同理，会话内 hook 不写进中转环境目录，而且 `set_hook` 会**主动摘掉**旧版本留在
那里的那一份（只是不再写入的话，升级上来的人身上会留一个没人管却一直在拦的 hook）。

`usecase::gate_ops::stop_managed` 里那句
`if all_sessions { execute() } else { execute_official() }`
是同一条不变量的另一半：门禁驱动的关停按 PID 杀进程时也要放过中转，
否则前面判断白做。

（这个函数原来在 `gate::stop_managed`。A1 把它连同 `run_watchdog` 一起搬进了
`usecase::gate_ops` —— 它要同时碰 killswitch / plugins / sessions / tray /
operations，是跨域编排而不是门禁自己的事。留在 `gate` 里正是 `gate` 变成
「伪装成底层的编排器」的原因。判定与执行仍在 `gate`。）

## ⛔ 不许加的功能

这三类不是「暂时没做」，是**明确不做**。加进来会让整个项目的定位垮掉，
也会让 [DISCLAIMER.md](DISCLAIMER.md) 变成谎话。

| 不做什么 | 为什么 |
|---|---|
| **设备指纹伪装**（UUID / 主机名 / MAC / machine-id 改写） | 主要用途是多账号规避，与硬约束「所有账户必须本人拥有」直接冲突。DISCLAIMER 写死了「不对账户状态作任何承诺」与「不为规避封禁而设计，也无法达到该目的」（按这两句话去搜，别记行号 —— 行号每改一次 DISCLAIMER 就漂一次） |
| **内置代理 / VPN**（自己当代理、mTLS 中继、链式转发、把**整机**流量强制走代理） | DISCLAIMER 第 5 节。做了这个，那一节就是假的。⚠ 0.19.0 开了两个**有边界的**口子（浏览器出站锁、系统代理修改），见下面「两个口子」一节 —— 那两个口子之外，这一条照旧 |
| **自动换号**（按额度、429、限流自动切槽位） | 合规边界四条的前两条。存在这条路径，「多槽位」的定位就从「管理你自己的账户」变成了「规避限制」 |
| **联网查额度**（调 OAuth 内部接口、抓 `/usage` 背后的端点） | 未公开接口，上游一改就断；而且 DISCLAIMER 写着不调它。用量只许读官方客户端**自己写在本机的文件**（`accounts/usage.rs`），零网络请求，只用于显示，面板不据此做任何决定 |

检测与**如实报告**不在此列 —— 面板可以告诉你「你的时区和出口对不上」，
但不替你改机器身份。两者的区别是：前者让使用者知情，后者替他伪装。

### 本机路由不在此列（0.14.0，使用者定的）

中转站那个只绑 `127.0.0.1` 的 API 路由器**是允许的**，别按上面那一行把它删掉。
使用者的原话：「这个只是中转站，而文档里说不做的是账户，两者不一样。」

分界线是**它替谁做决定**：

| | 允许 | 不允许 |
|---|---|---|
| 换的是什么 | 你自己填进去的中转站 | 官方 OAuth 账户槽位 |
| 为什么可以 | 你自己买的 Key、第三方端点，Anthropic 那边看不见 | 按额度 / 429 自动切槽位 = 「自动换号」，合规边界头两条 |

两条路径在代码上是隔死的：`qb-station` 不依赖 `qb-accounts` / `qb-launch`，
`src-tauri/tests/architecture.rs` 的 `relay_breakers_can_never_reach_account_switching`
钉着这件事。**别为了图省事把那条测试删掉或者往 `ALLOWED_SIDEWAYS` 里补一行。**

路由器本身的三条硬约束（`crates/qb-app/src/local_router.rs`）：只绑回环、
不改系统代理设置、不做链式转发。这三条是 DISCLAIMER 第 5.1 节的措辞依据 ——
**动了其中任何一条，就要同步改那一节**，否则免责声明描述的是另一个软件。

### 两个口子（0.19.0，使用者定的）

0.22.1 使用者另行明确要求：IP 纯净度中的「禁用本机 IPv6」默认开启，启动时应用，
关闭开关时恢复每张网卡原值。这是 IPv6 网卡绑定的独立例外，不改变下面代理与
防火墙的点击约束。原值必须先持久化；UAC 拒绝、部分失败、网卡移除时保留恢复记录，
界面以实际读取的绑定状态报告结果。启动应用完成后才恢复租约和启动看门狗。

使用者明确要求做这两件事，于是上面那条「内置代理 / VPN」开了两个**有边界的**口子。
`DISCLAIMER` 第 5.2 节是它们对外的措辞依据 —— **动了下面任何一条边界，就要同步改那一节**。

| 口子 | 允许 | 不允许 |
|---|---|---|
| **浏览器出站锁** | 给使用者点名的**那一个**浏览器 exe 加 Windows 防火墙**出站**规则：只放行指定的 VPN / TUN 接口 | 碰别的程序；加入站规则；改路由表、改 DNS、做转发；「全局」模式 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smithtaylor7748-ops/qb-gate](https://github.com/smithtaylor7748-ops/qb-gate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
