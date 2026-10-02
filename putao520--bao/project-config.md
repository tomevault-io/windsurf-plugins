---
trigger: always_on
description: **高性能反指纹浏览器运行时。** SpiderMonkey 引擎 + servo 全功能浏览器 + Node.js/Bun API 始终在线 + 内置 Stealth 反指纹,一个 Rust 运行时全搞定。
---

# Bao (包子) — Bun + SpiderMonkey + Servo

**高性能反指纹浏览器运行时。** SpiderMonkey 引擎 + servo 全功能浏览器 + Node.js/Bun API 始终在线 + 内置 Stealth 反指纹,一个 Rust 运行时全搞定。

## 核心愿景

把浏览器引擎、JS 运行时、反指纹能力统一到一个 Rust 二进制里:

- **反指纹浏览器** — 默认对抗 TLS/HTTP2/Canvas/Navigator/WebGL/Audio/行为指纹检测
- **Bun 兼容运行时** — `require` / `fs` / `http` / `crypto` / `bun:sqlite` 等 Node.js + Bun API 始终在线,与 Web API 同一 JSContext 共存
- **Headless 多页面库** — `PagePool` 多页面管理,`PageHandle` 高层 API(navigate / evaluate / screenshot)
- **CDP 自动化** — 内置 CDP Server,Playwright/Puppeteer 可直连 `ws://127.0.0.1:9222`

## 核心原则(铁律,所有工作必须遵守)

### 1. Bun 适配 Servo(Servo 是上游真源)

**遇到冲突改 Bun 不改 Servo。Servo 是上游,Bun 是下游。**

- Servo 代码禁止修改(BCE-002 / BCE-004 用户破例授权的 `script_thread.rs` / `lib.rs` patch 除外,已沉淀)
- Bao 层(`bao_engine` / `bao_browser` / `bao_cdp` / `bao_cdp_client` / `bao_stealth` / `bao_runtime` / `bao_uloop`)只适配 servo 的接口与数据模型
- Bun 的 C/Zig 层 → Rust 替换;JSC → SpiderMonkey 桥接是唯一需要手写的桥接层

### 2. JSContext 模型(BCE-20260621-001 修订版,废止旧"唯一 JSContext 共享"铁律)

**全局唯一 JSEngine + 每个 ScriptThread 持有自己的线程局部 JSContext**(servo 上游 `script_runtime.rs` 的 `RustRuntime::get()` 是 thread-local slot;SAFETY 注释:"only one JSContext can exist on the thread")。

- 各模式(CLI / browser / CDP)各自在其所属线程内使用该线程的 JSContext
- DOM ↔ Node.js 互操作必须发生在**同一线程内**(跨线程会破坏 activation 栈导致 SIGSEGV,此即 PagePool 混沌 SIGSEGV 根因)
- **禁止跨线程传递 `JSObject` 裸指针(铁律)**:`JSObject` 归属于创建它的线程的 JSContext。bao 层不得在跨线程结构(`DashMap` / `Mutex` / 全局 `static`)中持有 `JSObject` 裸指针,跨线程只能传 `PageId` / 句柄 / 序列化数据
- `bao_engine` / `bao_browser` 通过 `RustRuntime::get()` 获取当前线程的 JSContext(thread-local),不创建独立 JSEngine

### 3. 复用优先(Bun crate > 社区库 > 翻译他语言库 > 手写)

Bun workspace 中 ~85 个纯 Rust crate(零 JSC)是经过生产验证的高性能实现,**100% 复用,禁止手写已有功能**。

```
1. workspace 内 bun_* crate(已编译、已优化、已测试)
2. crates.io 成熟库(url, sha2, hmac, kurbo, svgtypes, etc. — 优先查依赖树内已有版本)
3. 无 Rust 库时:翻译同功能他语言库(如 mozilla C++ 参考实现)
4. 仅当 1/2/3 都没有时才允许手写(用户裁决 2026-09-27:实在没办法才自己裸写)
```

**只有以下情况允许手写 Rust**:

1. loop 核心必须与 `FilePoll` 共享 epoll fd → `bao_uloop` 的 epoll tick 是必要的
2. JSC → SM 桥接层(`bao_engine`)是必要的
3. Servo 集成桥接层(`bao_browser`)是必要的
4. CDP / Stealth / Node.js 兼容层(`bao_cdp` / `bao_stealth` / `bao_runtime`)是必要的

**禁止手写**(链接 / 复用 C++ 二进制或 Bun crate):

- `us_socket_*` / `us_socket_group_*` / `us_listen_socket_*` → 链接 C++ `libuwsockets.cpp` 二进制
- `bsd_send` / `bsd_shutdown` 等 BSD socket 辅助 → C++ 二进制已有
- HTTP 解析/响应 → `bun_uws::App`(C++ 二进制)已有
- DNS → `bun_dns` 已有
- 模块解析 → `bun_resolver` 已有
- Base64 → `bun_base64` 已有

### 4. 三化原则

| 原则 | 含义 | 检查点 |
|------|------|--------|
| **高性能化** | 零拷贝、SIMD、mmap、io_uring — 复用 Bun 已有的优化 | 禁止 `Vec::new()` 手写 buffer、禁止 `String::from_utf8_lossy` 替代零拷贝 |
| **去锁化** | 单线程 JS 执行模型下禁止 `Mutex`/`RwLock`,用 `thread_local!` + `RefCell` | `Mutex` 仅用于跨线程共享(HTTP 等真正的并发场景) |
| **成熟库化** | workspace 已有 crate > crates.io 成熟库 > 手写 | 每个新函数先 grep workspace crate 是否已有实现 |

### 5. SPEC SSOT + 范围守恒 + BCE(C-1 / C-5 / C-7)

- **SPEC SSOT** — `.spec/` 是唯一真相来源。SPEC 有定义按 SPEC 执行;SPEC 未定义停止报告用户,禁止自行补充
- **范围守恒** — 交付范围 ≡ SPEC 定义范围,双向零差集
- **BUG 类根除(BCE)** — 任何错误修复后强制走 BCE 闭环(归因 → 泛化 → 全项目横扫 → 批量根治 → 全量确认残留=0 → 防复发沉淀)。完整定义见 `~/.claude/rules/bug-class-eradication.md`

## 命名规范

| 层级 | 规则 |
|------|------|
| 用户品牌 | `bao`(`bao run` / `bao test` / `bao browser`) |
| JS 全局对象 | `Bun.*`(保留) + `Bao.*`(别名,同一对象) |
| 内部 Rust crate | `bun_*` 不改(保持上游兼容);`bao_*` 是新建层 |
| 环境变量 | `BUN_*`(保留) + `BAO_*`(新增别名,`NodeRuntime::new()` 调用 `init_env_aliases()` 把 `BAO_<SUFFIX>` 复制到 `BUN_<SUFFIX>`) |
| 代码引用 | 保留所有 Bun 内部引用 |

原则:用户输入 `bao`,代码里还是 `bun`。最小化与上游 Bun 的 diff。

## 架构分层

```
┌──────────────────────────────────────────────────────────┐
│                     bao (CLI binary)                      │
│            bao_bin → bao_cli (clap subcommands)           │
├────────────┬────────────┬──────────┬─────────────────────┤
│ bao_engine │ bao_browser│ bao_cdp  │ bao_stealth         │
│ SpiderMonkey│  Servo 桥  │ CDP WS   │ 反指纹              │
│ JSC→SM 桥  │ PagePool   │ Router   │ TLS JA3/JA4         │
│ context/   │ PageHandle │ Session  │ HTTP2 AKAMAI        │
│ job_queue  │ evaluate   │ 12 域    │ Canvas/WebGL/Audio  │
├────────────┴────────────┴──────────┴─────────────────────┤
│ bao_cdp_client  Playwright 风格高层 API(Browser/Page/...) │
├──────────────────────────────────────────────────────────┤
│ bao_runtime  Node.js/Bun 兼容(fs/http/crypto/sqlite/ffi) │
├──────────────────────────────────────────────────────────┤
│ bao_uloop  事件循环(epoll tick,共享 FilePoll fd)         │
├──────────────────────────────────────────────────────────┤
│          Bun ~85 个纯 Rust crate(零修改复用)             │
├──────────────────────────────────────────────────────────┤
│ mozjs(SpiderMonkey FFI) · libservo · boringssl · cdp-protocol │
└──────────────────────────────────────────────────────────┘
```

### Bao 层 crate

| crate | 路径 | 职责 |
|-------|------|------|
| `bao` | `src/bao` | **对外唯一公共 lib**：整栈 re-export（引擎+浏览器+runtime+CDP+Stealth 始终链接，无产品 feature 拆分） |
| `bao_engine` | `src/bao_engine` | SpiderMonkey 引擎封装,re-export `bun_sm` 核心类型;`context` + `job_queue` 自有模块 |
| `bao_runtime` | `src/bao_runtime`(crate 名 `bun_runtime`) | Node.js/Bun API 兼容层;`NodeRuntime`(Node.js 运行时入口;旧名 `BaoRuntime` deprecated alias) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [putao520/bao](https://github.com/putao520/bao) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
