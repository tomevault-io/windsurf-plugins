---
trigger: always_on
description: GhostLock：Android 内核提权工具。通过 PI-futex 竞争覆盖 waiter，依次完成
---

# AGENTS.md

GhostLock：Android 内核提权工具。通过 PI-futex 竞争覆盖 waiter，依次完成
W1（SELinux）、W2（凭据/uid 0）、W3（seccomp），最后 handoff 给 root child 做
KernelSU 模块加载。内核按精确 `uname -r` 匹配 HOCON profile，未匹配即拒绝运行。

## 三层架构

| 层        | 位置                | 说明                                                                                       |
| --------- | ------------------- | ------------------------------------------------------------------------------------------ |
| Android   | `app/`              | Kotlin/Compose UI、HOCON profile 解析/合并/覆盖、Shizuku UserService、日志                 |
| Native    | `src/core/`         | C++23 攻击 runtime，产物 `build/native/ghostlock`                                          |
| Extractor | `tools/extract_rs/` | Rust；boot.img / OTA / URL → `--format conf`（flatten GLK profile）/ `--format json`（v1） |

- Native 不是 JNI：`libghostlock.so` 是可执行 ELF，由 Kotlin `ProcessBuilder` 启动。
  CLI：`--ghostlock-app-call`（stdin GLK1）/ `--load-prebuilt-profile <bin>`
  （无参数的 v1 `offsets.json` 入口已移除）。入口细节见
  `docs/analysis/native-entrypoint-plan.md`（git 历史）。
- 内置 profile 在 `app/src/main/assets/kernel_profiles/`：`index.conf` 索引、
  `<uname-r>.conf` 每 release 一份、`execution-*.conf` 公共/分 route 调参、
  `credential-6x.conf`、`kernelsnitch-6x.conf`。格式为 HOCON（支持 `include`）。
- 组件模型：frontend（`root_child`；`umh_forward` 占位不可用）× backend（`cve_2026_43499`；
  `cve_2026_64560` 占位不可用）× middleware（`select_stack` / `tcp_zerocopy` / `multicast_waiter`），
  由 `Pipeline<F,B,M>` 编译期固定。组合与可用性的唯一权威是 `route/component_catalog.hpp`，
  选择显式来自 profile/wire，不从 kernel 版本推断。

## 常用命令

```sh
# Native
make -C src ghostlock                 # NDK 构建（ANDROID_NDK_HOME 或 local.properties）
make -C src native-host-tests         # 主机单元测试（含固定向量/生命周期）
make -C src lint-tidy                 # clang-tidy，要求 0 findings
ANDROID_NDK_HOME=... make -C src      # NDK 未自动探测时的显式写法

# Android / 提取器
./gradlew :app:assembleDebug
./gradlew :app:testDebugUnitTest
./gradlew exportKernelProfiles        # 生成 GLK1 .bin 到 build/kernel-profiles/
(cd tools/extract_rs && cargo test --release)

# 攻击函数形状对比（攻击路径改动必须跑）
python3 tools/cmp_disasm.py <baseline-binary> build/native/ghostlock
```

## 改动流程：先设计，后改动（强制）

先读 `docs/development/design-philosophy.md`（设计思想：11 条原则，以 SWEBOK/ISO 12207/ISO 25010
等软件工程理论为依据，含理论锚点与项目实例）。
完整工程规范见 `docs/development/engineering-standards.md`（架构/数据流/控制流/数据结构/代码风格/
文档/验证的详细规则 + 外部标准来源）。**任何改动前必须先完成设计**：

1. **Explore**：先读代码与文档；用 `git log --all -- <path>` 查历史设计与 device-gate 证据
   （`docs/analysis/**` 全在 git 历史里）。
2. **Design**：S 级（注释/格式）直接改；M 级写清动机/影响文件/行为差异/验证计划；**L 级**
   （攻击关键路径、wire/profile 格式、跨 Native↔Kotlin 契约、公共数据结构、新增 route）
   必须先产出计划文档（模板见 `docs/development/documentation-standards.md`）并获用户认可，再写代码。
3. **Implement**：最小改动，只碰设计列出的文件；不动 `kernelsnitch/`、`LegacyProfileConverter.kt` 的 v1 转换与
   规范中的"明确保留"清单。
4. **Verify**：按级别跑满 §1.3 门槛并保留证据；汇报时给出命令与结果，不写"应该没问题"。

- 攻击路径改动 = `cmp_disasm`（8 函数）+ 真机门禁 + 门禁记录（格式见
  `docs/development/documentation-standards.md`），缺一不可。
- 大改动按批次推进，一个批次只做一类事，上一批验证通过再进下一批。

## 代码约定

- C++23、`-fno-rtti`、静态 libc++；`-Wall -Wextra -Wconversion -Wsign-conversion`，
  新代码不得引入告警。include 用相对 `src/core` 的路径（如 `#include "route/tcp_zerocopy_route.h"`）。
- 命名空间分层：`ghostlock::{route,session,memory,attack,race,support,profile}`；
  `kernelsnitch/` 是上游原样移植代码，不要顺手重写。
- 全局状态只允许 `g_exploit_session`（进程唯一 singleton）与启动期只读的
  `g_direct_map_end`；新代码不得再引入可变全局或引用别名。审计记录见
  `docs/analysis/native-global-state.md`（git 历史）。
- 组件结构 = identity（kind，声明型，必须 host 可编译）+ 执行 policy（frontend/backend 步骤或
  middleware route）。middleware policy 以编译期能力 + 静态 hook 表达，backend 步骤模板化在
  middleware 上直接调用（不用虚基类）；Route 类满足 `prepare → execute → disarm → destroy`，
  仅经 `status` 汇报。新增组件（middleware/backend/frontend）的完整触点清单见
  `docs/development/adding-a-component.md`。
- 双侧一致性（改了必须两边同步，测试会抓）：
  - Native `profile/model.h` 的 `RouteKind`/`kRouteCatalog` ↔ Kotlin `data/route/RouteKind.kt`
    （`route_catalog_test.cpp` / `RouteCatalogAgreementTest.kt` 断言同一列表）
  - `profile/binary.cpp` 字段表 ↔ Kotlin route config 的 `entries()/apply()/from()`，键名逐字一致
  - route 私有参数放 route 扩展节；只有共享代码会读的才进公共槽（顺序也必须一致）
- 配置权威是 GLK1 profile + HOCON。执行层不得读配置类环境变量，只允许进程/路径类
  （`GHOSTLOCK_HOME`、`TMPDIR`、`GHOSTLOCK_KSU_LOG`）。需要新状态就扩展 profile。
- **不要随意新增或叠加配置/profile 传输格式版本号**。本分支统一为 **v2**（对象分段：
  `header + sections[name → fields[name → u64]]`，presence 由键是否出现表达，值按位严格保存）。
  Kotlin 与 native 版本绑定，同一分支内直接替换，不做旧版本兼容；不同 branch 需要不同格式时，
  先改本节与 `docs/development/`，不要就地再起 v3/v4。
- v1（旧 `offsets.json`）**只在 Kotlin 侧**由 `LegacyProfileConverter.kt` 转换为 v2；
  native **不再解析 v1**（`src/core/legacy/` 的 JSON 路径已删除）。新 route/新字段不要改 v1 转换。

## 核心攻击代码审查（仅触及时执行）

以下审查只在改动触及核心攻击代码或其资源准备/回收路径时执行；普通 UI、文档、配置编辑器等未触及这些代码的改动不要求进行这组耗时检查。核心攻击代码包括 waiter/race/payload/route/exec 阶段代码，以及直接准备、持有、访问或回收其资源的代码。

- **所有权与生命周期追踪**：审查中按 Rust 式所有权思路列出每个关键对象、指针和资源的创建/获取起点、所有者、借用/访问者、访问区间、终结点及释放者。确认所有可能访问它的使用者都先于终结点停止访问；区分 owning、borrowed、shared 和 transferred ownership，并核对转移后旧所有者不再释放或访问。
- **UAF 检查**：逐条检查普通对象与资源路径，确认不存在 use-after-free；只有作为漏洞原语而被有意利用的目标对象生命周期例外，不得把该例外扩展到辅助对象、race 状态、waiter、缓冲区、映射或同步资源。
- **终结点与清理顺序**：确认销毁/回收发生在合理的生命周期边界，覆盖成功、失败、重试、取消和提前返回路径。重点核对 PI 相关内存与 waiter 生命周期：参与者停止访问、同步/解除关联、资源回收之间的既有先后关系不得因重构而改变；缺少明确终结点或顺序依据时不得合入。历史上曾因重构漏回收 PI 内存导致 panic，相关变更须特别检查资源回收位置和退出路径。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YuKongA/ghostlock-app](https://github.com/YuKongA/ghostlock-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
