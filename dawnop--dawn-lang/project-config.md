---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 这是什么

**Dawn** —— 一门自制语言，编译到 JVM 字节码（也能经 GraalVM 出 native 二进制）。
编译器**已自举且只此一套**：`selfhost/` 用 Dawn 写成（词法到 codegen + LSP），
从上一 release 的种子 jar 自举；最初的 Kotlin 实现归档在 `kotlin-final` tag。
它不是玩具：同作者的 [dawnop-site](https://github.com/dawnop/dawnop-site)
整个生产后端（博客 + 网盘 + WebDAV）100% 跑在它编出来的代码上。

语言设计的权威定义在 [`docs/spec.md`](docs/spec.md)，里程碑历史在 [`docs/design.md`](docs/design.md)。

## 语言约定（**先读这条**）

**代码一律英文，文档一律中文。** 这条执行得很彻底，不是倾向：

- `.dawn` 源码（`selfhost/`、`site/`、`playground/`、`packages/`、`std/`）注释全英文；
  报错信息、CLI 输出全英文。
- `docs/` 全中文（索引与状态分层见 [docs/README.md](docs/README.md)）。
  （篇数不在这里复述——`scripts/doc-check.py` 每次跑都会报，那才是不会过期的计数。）
- **对外那一层反过来：英文是正本，中文是译本。** `README.md` 是英文原文，
  `README.zh-CN.md` 是它的译本。**改对外文案时先改英文，再改中文译本**——
  译本头部的 `<!-- doc-check: translation-of ... @ <digest> -->` 记着原文的摘要，
  英文一动、中文没跟，`scripts/doc-check.py` 就红（登记表在该脚本的 `TRANSLATIONS`）。
  方向是这么定的：谁是派生物，腐烂就落在谁身上；对外层的读者大多不读中文，
  把英文做成派生物等于把腐烂藏在最多人看、最没人校对的那一面。
- **提交信息也是英文**，一行祈使句主题。这里曾长期写着 `type(scope): 中文摘要`——
  摘要用中文这条**自 2026-07-25 起再没出现过**（`git log --format='%s' -200` 里 0 条中文，
  历史上 126 条）；`type(scope):` 前缀也基本退了（近 200 条里 4 条）。以 `git log` 为准。
  正文写什么见文末「重要约定」。
- **All public GitHub writing must be in English, without Chinese text.** This includes
  commit subjects and bodies, PR titles and bodies, PR/issue comments, code review
  replies, and any other publicly posted text. The Chinese documentation convention
  must never be applied to PR bodies or other public discussions. Check the entire
  text before publishing, not just the title or subject. Do not rewrite pushed
  commit history to enforce this rule retroactively; obtain maintainer confirmation
  before translating already-merged PR bodies.

写代码时别把 docs 的语言带进去，反之亦然。

## 文件头注释：讲**为什么**

每个文件在顶层声明之前有一段注释，说明这个文件为何存在、以及它做了哪些不显然的取舍。
不是「这个文件定义了 Parser」那种复述，是「为什么选 `io.get-coursier:interface`
而不是 coursier 本体」那种。新增文件请照做。

## 命名是**语义**，不是风格

`lower_snake_case` = 值/函数/模块，`PascalCase` = 类型/构造器。**这是强制的**：
parser 靠首字母大小写消歧（`TYPEIDENT` 是独立 token），所以改大小写是改语义、不是改风格。
权威表述在 [`docs/spec.md`](docs/spec.md) §1（「命名约定是强制的」那条）。

## 常用命令

```bash
./bin/dawn --version                     # 首次自动拉种子并重建工具链
./bin/dawn test selfhost                 # 编译器主体测试
./bin/dawn test compiler-plan            # 独立 Planner/manifest 测试
./bin/dawn run examples/data/shapes.dawn      # 单文件
./bin/dawn run examples/projects/hello_mod     # 多模块项目
./bin/dawn test site                     # 站点生成器的 Dawn 测试
./bin/dawn fmt compiler-plan std site selfhost packages examples --check  # Dawn 代码格式检查
./site/build.sh                          # 端到端建站（含 Playground 前端 bundle）

./scripts/selfhost-fixpoint.sh           # 自举固定点：种子→A→B→C，B==C
./scripts/build-release-jar.sh -o /tmp/dawn-selfhost.jar  # 唯一 release JAR 配方
./scripts/selfhost-prev-diff.sh          # N vs N−1 差分（emit 语料 + 生态扫描）
./scripts/selfhost-run-diff.sh           # CLI 转写对拍 vs 上一 release
./scripts/selfhost-lsp-diff.sh           # LSP 会话对拍 vs 上一 release
```

> `bin/dawn` 需要 JDK 21。没设 `JAVA_HOME` 时它会在 `~/tools/graalvm-*` 里找
> （macOS 的 `Contents/Home` 与 Linux 的顶层 `bin/` 两种布局都试）。
> 种子 = `scripts/seed-release.txt` 钉住的 release 的 `dawn-selfhost.jar`，
> 缓存在 `.dawn/seeds/`；离线或调试用 `DAWN_SEED=<jar>` 指本地 jar。

## 目录结构

```
selfhost/          编译器主体（Dawn 写 Dawn）；消费 compiler-plan，ASM 只属于这里的 JVM 后端
compiler-plan/     无 Java 的 source/manifest/MVS/fetch 规划包；不是 packages/* 发布包
selfhost/src/      分十个目录，依赖单向向下（拓扑序即下面的顺序），入口留根：
                   embed/  生成物，不许手改（stdsrc rtsrc unicode_case unicode_class）
                   front/  词法/语法/诊断/格式化（token lexer parser ast diag suggest fmt lexdump astdump）
                   check/  类型与检查（types tast exhaustive jsig cx passes checker）
                   ir/     Core IR 及其上的 pass（core lower interp reach coredump）
                   jvm/    JVM 后端（codegen emit ops help jreflect rtclasses jarw testrun jfold）
                   pkg/    compiler-facing 包操作（maven vendor add）；规划基础在 compiler-plan/
                   driver/ 模块图与整程序驱动（analyze stdlib checkdump）
                   c/      native 后端（emitc cdriver ctestrun rc）
                   lsp/    语言服务（server lspc lspq）
                   contract/ 白盒契约探针（probe cold prefix bench 等），读 pub(pkg) 的检查器状态；
                             不在 main/nmain 的模块图里，由 test 门的 `dawn test selfhost` 跑
                   根：main.dawn nmain.dawn doc.dawn version.dawn
std/               标准库源（构建 selfhost 时编译进独立 jar 的 stdsrc 模块）
packages/          可发布/复用源码包（json、web、fspath、sha2、inflate、tea），[deps] 消费
site/              用 Dawn 自己写的静态站生成器（自举）
site/play-ui/      Playground 编辑器（TypeScript + Vite + CodeMirror 6）
playground/        Dawn 写的 playground 后端
editors/vscode/    VS Code 插件
docs/              设计文档（中文）
examples/          示例
```

codegen 的**运行时 intrinsic 契约**声明在 `selfhost/src/check/types.dawn`（`Rt` / `Intr` /
`intrinsics()`，约 1645–1806 行）：语言只说一个 primitive 归哪个**运行时模块**
（`RtStrings`/`RtBytes`/`RtArray`/`RtIo`），由各后端自己决定那是什么——JVM 后端在
`emit.dawn` 用 `rt_class`/`rt_intrinsic_class`（586/598 行）映到类名，native 后端映到
C 翻译单元。背景与分期见
[docs/runtime-intrinsics-design.md](docs/runtime-intrinsics-design.md)。

> 这段以前写的是「`emit.dawn` 的 `rt_intrinsic_target` 表」。那张表**已经不存在**了——

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dawnop/dawn-lang](https://github.com/dawnop/dawn-lang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
