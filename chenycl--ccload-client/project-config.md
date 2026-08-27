---
trigger: always_on
description: 给编码代理（Claude Code / Codex / OpenCode / Gemini CLI …）和新加入的人类开发者看的
---

# AGENTS.md

给编码代理（Claude Code / Codex / OpenCode / Gemini CLI …）和新加入的人类开发者看的
仓库须知。**先读这一份，再动代码。**

`CLAUDE.md` 只是指向本文件的软链接式入口，内容不重复。

---

## 这是什么

`ccload-client` 是 [ccLoad](https://github.com/caidaoli/ccLoad) 的桌面客户端：
Tauri 2 壳体（Rust）+ React 18 前端，把 Go 写的 ccLoad 内核作为 sidecar 托管起来，
并把本机各个 CLI（Claude Code / Codex / Gemini CLI / Grok Build / OpenCode）的配置
指向它。

```
┌─ src/            React 前端（Vite + TS + Tailwind + TanStack Query）
├─ src-tauri/      Rust 壳体（commands/ 是 IPC 边界，services/ 是实现）
├─ vendor/ccLoad/  上游内核源码，由 scripts/fetch-kernel.mjs clone 到此（普通
│                  目录，不是 git submodule），**只读**
├─ scripts/        内核拉取与编译（Node，无依赖）
└─ KERNEL_VERSION  钉住的上游内核 tag
```

---

## 硬约束

### 1. 不要改 `vendor/ccLoad`

内核是上游项目，本仓库只消费它。需要新内核能力时的正确做法是：改
`KERNEL_VERSION` 里的 tag → `pnpm kernel:fetch` → `pnpm kernel:build`。

在 `vendor/` 下打补丁会让「壳体能编过、用户机器上编不过」，而且下一次
`kernel:fetch` 会把补丁冲掉。

### 2. 内核已有的能力，客户端不要重做一遍

分工是明确的：

| 归内核 | 归壳体 |
| --- | --- |
| 代理转发、渠道选择、故障转移、协议转换（Anthropic ↔ OpenAI ↔ Gemini ↔ Codex）、计费统计、模型清单拉取 | 托管进程、写各 CLI 的配置文件、备份/回滚、把内核的 Admin API 包装成界面 |

动手写新功能之前先在 `vendor/ccLoad` 里查一遍：`/v1/*` 和 `/v1beta/*` 已经是
Any 路由（内核**本身就是**本地 OpenAI/Anthropic 出口代理），
`GET /admin/channels/:id/models/fetch` 已经能问上游要模型清单，
`POST /admin/channels/models/refresh-batch` 已经支持 `merge|replace`。
重复实现一遍只会多一份要维护的、行为还不一致的代码。

具体到几个反复有人想在壳体里重写的：

* **改上游请求的 header / body** —— 内核的 `channels.custom_request_rules`
  已经做了（`internal/app/custom_rules.go`）：`headers[]` 支持
  `remove | override | append`（`remove` 还能按逗号 token 精确剔除，比如从
  `anthropic-beta` 里只去掉一个 flag），`body[]` 支持按点分路径
  `remove | override`。认证头（`authorization` / `x-api-key` /
  `x-goog-api-key`）有黑名单，规则会被静默忽略并打 warn。壳体要做的是把它
  包装成界面，不是再写一个转发层。
* **强制某个模型** —— 渠道模型条目上的 `redirect_model` 是 1:1 重定向
  （`resolveActualModel`）；想「不管请求什么都发 X」用 `custom_request_rules`
  的 body override（路径 `model`），它在协议转换之后、发出之前生效。
* **多层 fallback** —— 内核只做一跳，客户端的「模型链」把链拆成 N 个同别名、
  优先级递减的渠道来实现，见 `services/fallback.rs`。

### 3. 写用户配置文件的三条规矩

这部分踩过的坑最多，改 `src-tauri/src/services/cli_*.rs` 之前务必看完：

* **原子写 + 保权限。** 一律走 `cli_io::write_atomic`。`std::fs::rename` 会
  **交换 inode**，目标文件原有的权限位不会保留 —— `~/.claude.json` 里有
  OAuth 账号和一堆 MCP bearer token，被我们从 `0600` 降到 `0644` 过一次。
  `carry_permissions` 失败时宁可整个写入失败，也不能让带凭据的文件以更宽松的
  权限落地。
* **先快照。** 任何写入前调 `BackupStore::snapshot`，用户能在「快照历史」回滚。
  每个 CLI 最多保留 5 份，按时间覆盖，但**首份 pristine 快照永不淘汰**。
* **合并，不要整块替换。** MCP 服务器、profile、模型目录都要按键合并 ——
  整块 `insert` 会把用户手写的 `startup_timeout_sec` / `cwd` / 自定义模型
  一次抹掉。「导入」在语义上必须是**追加**：不动用户当前选中的模型、不动他
  已经绑好的槽位。见 `services/model_import.rs` 的模块注释。

开发期请在设置里打开「CLI 写入走沙箱」，写入会落到
`~/.ccload-client/sandbox/`，不碰真实配置。

#### 3b. 写用户的 markdown 指令文件：只碰标记块

「系统注入」（`services/system_inject.rs`）往这五个文件里写东西：

| CLI | 全局指令文件 |
| --- | --- |
| Claude Code | `~/.claude/CLAUDE.md` |
| Codex | `~/.codex/AGENTS.md` |
| Gemini CLI | `~/.gemini/GEMINI.md` |
| Grok Build | `~/.grok/AGENTS.md`（Grok 截断到 10000 字符，静默） |
| OpenCode | `~/.config/opencode/AGENTS.md` |

它们和 settings.json 不是一类东西：**用户不认为这是工具在管的文件**，
`~/.claude/CLAUDE.md` 里往往是攒了几个月的个人规则，抹掉不可逆。所以只认
`<!-- ccload:begin -->` / `<!-- ccload:end -->` 之间的内容，块外一个字节都不动。

两个已经写进测试、别退回去的细节：

* 只有 BEGIN 没有 END 时**按「没有块」处理**，不能从 BEGIN 删到文件尾 ——
  半个标记多半是用户手工删了一半，贸然删到尾会吃掉他后面写的所有东西。
* 装了卸、卸了装反复来回，不能攒出越来越多的空行。

块内每一小节前面还有一行自己的标记（`<!-- ccload:vision -->` 这种），**回显靠
它，不靠比对文字**。前端曾经的做法是「把某一段单独渲染一遍，再看块里包不包含
这段文字」—— 那个判断在我们自己改一个字的那天就失效了：用户机器上的块是上个
版本写进去的，逐字对不上，于是勾选框显示成没勾，整段旧文字被当成用户手写内容，
再按一次「更新」就写出一段旧的加一段新的。解析放在 `parse_block` 里，和渲染同
一个文件、同一组测试盯着；没有标记的老块走标题兜底，界面上标成「旧版」，按一次
「更新」就升成新格式。

### 4. macOS 包必须是 universal

壳体和内核的架构不一致时，Apple Silicon 上被 Rosetta 翻译的 WebKit 会 SIGBUS，
表现是白屏/闪退。`tauri build --target universal-apple-darwin`，内核也要出
两份再 `lipo`。CI 里已经这么做，本地手打包别偷懒。

---

## 常用命令

```bash
pnpm install
pnpm kernel:fetch        # 按 KERNEL_VERSION 检出 vendor/ccLoad
pnpm kernel:build        # 编内核 → src-tauri/binaries/
pnpm dev                 # 前端 + 壳体热重载

pnpm typecheck                                   # tsc --noEmit
cd src-tauri && cargo clippy --all-targets -- -D warnings
cd src-tauri && cargo test
```

**提交前这三条必须全绿**，CI 跑的就是它们。

不要直接运行 `src-tauri/binaries/ccload`：它没有 `--version` 之类的短路参数，
任何参数都会让它启动服务器，并在当前目录建一个 `data/ccload.db`。

### 会话卡在 400 too long 的时候

Claude Code 按**模型声明的窗口**决定何时自动压缩，而走 ccLoad 时真正拦你的是
**中转那一家的上限**。两个数对不上（典型：模型名挂了 `[1m]`，中转其实只给
500k），阈值就被算在一个不存在的分母上 —— 等它触发已经越过真实天花板了。之后
`/compact` 自己也发不出去，因为它同样要把整段 transcript 发上去。

界面上在「会话救援」页；命令行等价物：

```bash
python3 scripts/rescue-session.py <session.jsonl>          # 只看报告
python3 scripts/rescue-session.py <session.jsonl> --write  # 真的改
```

先退出那个 Claude Code 窗口再动手 —— 进程里有内存态，会把你的修改盖回去。

预防的办法是把真实上限告诉客户端，在「CLI 接管」页的环境变量里填：
`CLAUDE_CODE_MAX_CONTEXT_TOKENS` 填中转的真实上限，
`CLAUDE_CODE_AUTO_COMPACT_WINDOW` 填一个留足余量的值（压缩请求本身也要把整段
对话发一遍，顶着上限做不成任何事）。

---

## 自带的两个 MCP 服务器

客户端二进制自己就是 MCP 服务器 —— 按 `argv[1]` 分流（见 `lib.rs`），所以装进
CLI 的只是一条指向本二进制的 stdio 命令，没有第二个东西要安装或打包。

| 子命令 | 服务器名 | 实现 | 干什么 |
| --- | --- | --- | --- |
| `vision-mcp` | `ccload-vision` | `services/vision_mcp.rs` | 把图交给多模态模型描述 —— 让文本模型「看得见」 |
| `image-mcp` | `ccload-image` | `services/image_mcp.rs` | 文生图 / 改图 —— 让所有模型「画得出」 |

两边共用一套东西，**不要各抄一份**：`Image`、`mcp_text`、`load_source`（含
`[Image N]` 按编号取图）、`same_endpoint`、`record_call` 都在 `vision_mcp` 里，
`pub(crate)` 导出。界面上那段「哪几家装了 / 装 / 卸 / 重写」是

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChenYCL/ccload-client](https://github.com/ChenYCL/ccload-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
