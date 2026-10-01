---
trigger: always_on
description: > Bun-like JS runtime，直连 Mozilla SpiderMonkey（经 `servo/mozjs` Rust 绑定）。
---

# AGENTS.md — winterjs2 工作规约

> Bun-like JS runtime，直连 Mozilla SpiderMonkey（经 `servo/mozjs` Rust 绑定）。
> 从 `winterjs2-old`（WinterCG server + spiderfire）推倒重来，老项目只当参考，不合、不动。
>
> **新会话开工顺序**：本文件 → `docs/plan3.md` §0（唯一进度入口）→ 动某域前
> 在 `docs/pitfalls.md` 索引里 grep 该域条目。其余文档见 `docs/README.md`（多为存档，不必读）。

## 0. 工作流铁律

1. **每次修改都要 `git commit`，提交后即 `git push`**：小步提交；提交前必看 `git status --short` +
   `git diff`，只 stage 意图内的文件，绝不提交 secrets；push 到 `origin/master`
  （用户已常设授权，今后无需再问；push 前确认工作区干净、无多余提交混入）。
2. **先查证据再下结论**：读文件、跑构建、跑 `./target/debug/winterjs2` 实测；
   与文档矛盾以实测为准并更新文档。
3. **踩坑必记**：新坑追加到 `docs/pitfalls.md` 末尾（编号续排 `4.N`：症状 → 根因 →
   修法 → 复现 → 推广铁律）；只有**新的通用铁律**才在本文件 §4 摘要补一行。
4. **依赖随缘更新**：除 `mozjs` 必须精确钉死外（§2），其余 caret 不锁上限，
   `cargo update` 随便跑；跑坏就地修并回写 `docs/dependencies.md`。
5. **新工具先找轮子**：想手写新工具/模块/功能时，先去 crates.io 找依赖，
   符合标准记入 `docs/dependencies.md`，然后**停下来问用户**，点头后才引入。
6. **能 safe 不 unsafe**：新增 `unsafe` 前先证伪 safe 路线（§6 三问）；存量只减不增。
7. **测试三件套随功能落地**（完工标准含三绿）：
   - 模块测试：`src/` 内 `#[cfg(test)]`，覆盖纯 Rust 可测逻辑。
   - 黑盒测试：`tests/`，经 CLI 断言用户可见行为；文件按 src 域对齐
     （`tests/<域>.rs`，共享 helper 进 `tests/common/mod.rs`；node 域二层
     `tests/node/<mod>.rs` 经 `#[path]` 挂壳，域内脚手架进 `tests/node/helpers.rs`）；
     每个新 API 含**正常 + 报错 + 边界**三件；`UNSAFE-BOUNDARY` 新增必配 panic 路径用例。
   - 冒烟：§3 五条，构建后必跑，不过不提交。
   - 全量回归：`cargo nextest run`（见 §4 测试与跑分）。
8. **CLI 全 flag 规范**：无裸子命令、无裸位置参数——所有动作一律 `-x/--xxx`；
   一次恰好一个动作，多给即错；动作的必需值紧贴其 flag；修饰 flag
   （`--dry-run/--registry/--port` 等）只在对应动作下生效。help/补全/man 由同一套
   flag 生成（`localized_command`）。
9. **单文件 ≤1000 行**：项目内全部 `.rs`（`src/`+`tests/`+`benches/`）与 `src/**/*.js`
   （内嵌 JS SOURCE，2026-09-25 D3 纳入）不超过 ~1000 行，无豁免（`sample/` 不管）；
   提交前跑 `scripts/check-lines.sh`；超限 JS 用 `scripts/split-js.py <js> <rs>` 按方法边界切片
   （`concat!(include_str!…)` 字节恒等，脚本内断言；提交前再 `cmp` HEAD 原件）。
   拆分纪律：① 纯搬移先行（`git diff -w` 只见路径），调用方经 `pub use` 原位重导出；
   ② 一文件一提交，每步 0 警告 + 对应域测试绿；③ 引擎协议代码（`runtime`/`state`/
   `jsapi_glue`/`jobqueue`/`modules`）只拆纯逻辑，会话管线与 trace 不动；
   ④ Node 移植的 JS SOURCE 沿"域/算法族"切，SOURCE 随实现走。
10. **大数据走外置盘**：系统盘余量紧张，node 套件检出、sweep 工件、探针一律放
    `/Volumes//wjs-data`（软链 `~/wjs-data`）；`cargo test` 不改 `TMPDIR`（4.207），见 plan3 §0.5。

## 1. 基线（2026-09-09）

- `mozjs = "=0.26.0"`（Gecko 153，crates.io 最新发布版），`Cargo.lock` 入库。
- Rust stable 最新（现 1.98），edition 2024（即 stable 最新；2027 尚不存在）。
- 版本号用 CalVer `YY.MM.发版日`——**第三位是发版日不是顺序补丁号**（9 月 13 日发版即 `26.9.13`，9 月 27 日即 `26.9.27`；cargo 可解析；`^26.9.0` 即年内自动升）。
  依赖清单与 10-target 矩阵见 `docs/dependencies.md`。
- 无 `rust-toolchain` pin、无 spiderfire/ion 依赖、无 server/request_handlers。
- CLI（全 flag，§0.8）：`winterjs2 --run <file|script>`（带脚本后缀→文件直跑；
  裸名→package.json `scripts` 优先、同名文件回落；JS bin 递归自身执行，零 node）/ `winterjs2 --eval <code>` /
  `winterjs2 --config [--schema]` / `winterjs2 --completions <shell>` / `winterjs2 --man` 等，
   见 `src/`（cli/runtime/dispatch/error/logging/settings/alloc 模块；`runner.rs` 已由 `runtime.rs` 接替，见 plan.md）。
- 依赖 2026-09-10 起全量入库（docs/dependencies.md 头部决策记录），代码按 Phase 接线。

## 2. 依赖铁律

- **mozjs 永远钉死精确版本**，只跟随 servo release 手动升级。
  **绝不 `cargo update -p mozjs`**（会浮到 servo main HEAD，当场炸）。
- 退路：若 153 线踩到上游 bug，退回 ESR140 线
 （`mozjs = "=0.15.18"` + `mozjs_sys =140.14.0-lts`，Servo 线上在用的线）。

## 3. 构建（macOS Apple Silicon，每次新 shell 必 export）

```bash
export SDKROOT="$(xcrun --show-sdk-path)"
export LIBCLANG_PATH="/opt/homebrew/opt/llvm/lib"   # bindgen 用
export PATH="/opt/homebrew/opt/llvm/bin:$PATH"
cargo build
```

- `mozjs_sys` 走预构建 `libjs_static.a`，debug 全量约 25 秒，不用怕。
- 验证：`./target/debug/winterjs2 --eval '40 + 2'` → `42`；
  `./target/debug/winterjs2 --eval 'throw new Error("boom")'` → 非 TTY 下 node 形
  （`eval.js:1` / 源行 / `^` / 空行 / `Error: boom` / `    at eval.js:1:7`），exit=1
  （TTY 下由 miette 图形渲染，带代码框，语义同；2026-09-25 D4）。

### 冒烟（每次构建后必跑，不过不提交）

```bash
./target/debug/winterjs2 --eval '40 + 2'                                                    # → 42
./target/debug/winterjs2 --eval 'await new Promise(r=>setTimeout(()=>r(1),10))'              # → 1
./target/debug/winterjs2 --eval 'new URL("https://ex.com/?a=1").search'                     # → ?a=1
./target/debug/winterjs2 --eval 'new TextEncoder().encode("hi").length'                     # → 2
./target/debug/winterjs2 --eval 'await (await fetch("data:text/plain,x")).text()'         # → x
```

## 4. 铁律摘要（全文见 `docs/pitfalls.md`，编号即 `§4.N`）

**GC / 引擎边界**
- `evaluate_script` 返回后、逐任务微任务执行前，调 JSAPI 先进 `AutoRealm`（4.1/4.116）。
- 跨 GC 存活的 JS 值：`Box<Heap>` 定址 + trace 同步，缺一不可；禁裸 `Heap` 进可搬运容器（4.40/4.68）。
- Rust→JS 调用：函数体第一行把全部 JS 值参数入 `rooted!` 槽，之后才许分配；调用链中间值、
  队列批量取出值一律当场 rooted——`call_one` 入口 rooting 不保调用间（4.80/4.118/4.141/4.161）。
- `with_rooted`/`with_plain` 不可嵌套，闭包内只做纯数据（4.14）。
- 不 drop `Runtime`/`JSEngine`（`process::exit` + forget）；同进程再跑 JS 走 `run_isolated` 新线程；
  `engine` 先于 `rt` 声明（4.8/4.22/4.24）。
- 非 JS 线程（notify/回调线程）不碰 TLS state，生数据送回 JS 线程再判定（4.153）。
- GC finalize 内只做纯 Rust 簿记，addon 回调排到安全点（4.78）；napi 值随 scope、跨 scope 走 ref（4.77/4.79）。

**事件循环**
- 同步决议 promise 的结算点到 park 之前必须至少一轮 `RunJobs`；退出旗检查放 `RunJobs` 之后（4.18/4.46）。
- park 唤醒集 ≠ 存活判定集：unref 源到点要醒、不续命、不算 progressed（4.94）。
- fatal 类失败（入口错/未处理 rejection）要有提前跳出的检查点，不能只在循环尾收割（4.70/4.137）。
- `nextTick` 走原生队列，禁用 `queueMicrotask` 模拟（4.118）；"等回包"路径禁阻塞 native，改投递 + 轮询（4.112）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bemly/winterjs2](https://github.com/Bemly/winterjs2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
