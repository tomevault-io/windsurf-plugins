---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 仓库总览

Mineral 是一个多源终端音乐播放器(ratatui),目前已落地"模型 + 网易云 channel + TUI 雏形",目标是把本地音乐与多个云源合并为一组统一的 `Vec<Song>` 供 UI 消费。

整个仓库是单一 cargo workspace(根目录 `Cargo.toml`),成员通过 glob 声明:

## 常用命令

```bash
cargo build                                       # 构建整个 workspace
cargo run  -p mineral --features mock             # 运行 TUI,使用 mineral-channel-mock 喂假数据
cargo t                                           # = nextest run --workspace(跑全仓测试,见下 alias)
cargo td                                          # = test --workspace --doc(nextest 不跑 doctest,单独兜)
cargo snap                                        # = insta test --test-runner nextest --review(改了快照时)
cargo nextest run -p mineral-channel-netease      # 仅跑网易云 crate 的测试
cargo test --test crypto_vectors                  # 跑加密自检 harness(byte-for-byte 比对 openssl)
cargo fmt                                         # 格式化(rustfmt)
cargo fmt --check                                 # CI 用,不修改
cargo clippy --workspace --all-targets -- -D warnings  # 严格 lint(包括函数体量约束)
```

测试运行器是 **cargo-nextest**(需 `cargo install cargo-nextest cargo-insta`);`cargo t` / `td` / `snap` 是 `.cargo/config.toml` 里的 alias。nextest 配置见 `.config/nextest.toml`(daemon e2e 串行组 + retries)。nextest **不跑 doctest**,doctest 单独走 `cargo td`。

所有测试**禁止**debug run, debug 运行花的时间比 release 编译多得多, 大头时间都在编译

**禁止**先跑部分测试再跑全量，浪费大头编译时间， 直接跑全量

## 架构要点

### 数据模型 = 跨 source 的契约

`mineral-model` 的设计原则是"平铺合并":模型里没有 source-specific 字段。任何 channel 实现都把网络/本地原始数据先映射到 `mineral-model` 的类型,再交给上层。这意味着新增 channel 时,**不要**给 `Song` 加 source-only 字段——要么提升为通用字段,要么保留在 channel 内部 dto 里。来源**不再是独立字段**:`Song::source()` / `Album::source()` 等从各自 `id` 的 namespace 派生(见下)。

ID 类型(`SongId`、`AlbumId` 等)由 `mineral_macros::define_id!` 生成,是**结构化**的 `{ namespace: SourceKind, value: IdString }`——来源是 ID 的内在属性,全局唯一性由类型强制(裸值只在 source 内唯一)。约定:

- **构造统一走 `Id::new(namespace, value)` 单入口**(如 `SongId::new(SourceKind::NETEASE, raw)`)。未来换 namespace 表示只动这一处。
- `value()` / `as_str()` 取**裸值**喂 channel 后端(网易云请求体要纯数字串)/ 日志;`qualified()`(= `namespace:value`)给任务去重键等需要全局唯一字符串的地方。
- ID 派生 `Eq/Hash`(含 namespace),可直接当 HashMap key 而**不会跨源碰撞**。需要随机生成的 ID(如本地音乐)用 `define_uuid!` 的 `new_uuid(namespace)`。

`SourceKind` 仿 `http::StatusCode`:**newtype + 关联常量**(`SourceKind::NETEASE` / `LOCAL` / `MOCK`),内部 `&'static str` 故 `Copy`、强类型、**开放**(插件经 `from_static` 运行时铸造)。**身份只认 `name`**——`label`(UI 展示名)/ `palette`([`PaletteRole`],主题无关的调色角色,TUI 经 `Theme::source_color` 落地)是随 `name` 走的展示元数据,不参与 `Eq`/`Hash`/serde(序列化只写 `name`,反序列化按 `name` 解析回常量,未知名 intern)。因此 UI 给来源配图标/颜色**不该 match `SourceKind`**,读 `.label()`/`.palette()` 即可,插件源自动有合理展示。

### `MusicChannel` trait 是抽象边界

`mineral-channel-core::MusicChannel`(`async_trait`)定义了所有 channel 必须实现的方法:搜索、详情、播放 URL、歌词、可选的登录/用户播放列表。**TUI 只面向这个 trait 编程**.。

**术语:`channel`(适配器)≠ `source`(身份)**——`source`(`SourceKind`)是数据的**来源身份**,烙进每个 ID 的 namespace(回答"这条数据来自哪");`channel`(`MusicChannel` 实现)是取数的**连接器 / 适配器**(回答"用哪个后端去取"),经 `channel.source()` 声明它服务哪个 source。注释与命名别把两者混用:讲 ID 归属 / `Song::source()` / `sources.<name>` 配置时用 **source(来源)**;讲搜索 / 详情 / 取 URL 的后端实现时用 **channel**。同名异义的 `rodio` 声道 / `tokio` channel 与本词表无关,别牵连。

### 配置托管:daemon 是唯一 watcher,client 只消费推送

配置(file + session 覆盖)是 daemon 上的一份**运行时状态**:daemon mtime 轮询
config.lua、合成 `merge(default, user, overlay)`、落型校验后经 `Event::ConfigChanged`
推整树给订阅 client(握手先重放一帧);脚本 `mineral.config.override(path, value)` 是
合成的一层,path 必须是真实配置路径,坏覆盖按 serde 报错路径剔除并警告。**client 不看
文件**(TUI 启动本地 load 一次只是自举,连上即被推送顶替)。

client 侧配置消费两条规矩:
- **现读优先**:组件直接读 `state.cfg`(Arc,换整棵即热更),**不许**构造期把配置值
  拷进自己字段(第二数据源)。
- 确需构造期折算 / 固化的(拍数折算、FFT 预计算、缓存预算),必须挂
  `App::apply_config` 单入口(就地重设:`retempo` 保动画相位、`set_budgets` 不清缓存),
  并配一条重载测试(仿 `mineral-tui/src/runtime/reload.rs` 的既有测试)。

## Rust 工程约定

### 当前 workspace 强制的 lints

完整列表见 `Cargo.toml [workspace.lints.rust]` / `[workspace.lints.clippy]`(随项目演进,以那里为准)。
按职责分组,以下都是 `deny`:

- **panic 类**:`panic` / `unwrap_used` / `expect_used` / `indexing_slicing` / `index_refutable_slice` —— 测试也不豁免,改用 `?` + `assert_*`。
- **数值安全**:`as_conversions` / `cast_lossless` —— 强转用 `TryFrom` / `try_into`。
- **进程安全**:`exit` / `mem_forget`。
- **错误处理**:`map_err_ignore` —— 不许 `.map_err(|_| ...)` 丢上下文。
- **并发**:`mutex_integer` / `maybe_infinite_iter`。
- **所有权 / 性能**:`implicit_clone` / `needless_pass_by_value` / `cloned_instead_of_copied`。
- **代码清晰度**:`branches_sharing_code` / `mismatching_type_param_order` / `option_option` / `wildcard_imports` / `redundant_closure_for_method_calls` / `uninlined_format_args` / `manual_let_else` / `use_self` / `format_push_string`。
- **体量约束**(详见下文):`too_many_lines` —— 阈值 300,在 `clippy.toml`。
- **rust 层**:`warnings = "deny"` / `unsafe_code = "forbid"` / `missing_docs = "deny"`。

显式 `allow`:`if_same_then_else` / `len_without_is_empty` / `missing_safety_doc` / `module_inception`。

### 其余约定(未必经 lint,仍需遵守)

* 禁止 `unsafe_code`、`unwrap`、`expect`、`panic!`、`as`(数值强转)、wildcard import(`use foo::*`)。需要数值转换时用 `TryFrom` / `try_into`;需要快速失败时用 `?` + `color_eyre::eyre::WrapErr`(`.wrap_err(...)` / `.wrap_err_with(...)`,trait 也兼容 `.context(...)` / `.with_context(...)`)。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [10knamesmore/Mineral](https://github.com/10knamesmore/Mineral) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
