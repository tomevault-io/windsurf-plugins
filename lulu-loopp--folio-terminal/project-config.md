---
trigger: always_on
description: > 每个会话开工前读一遍。`docs/DESIGN.md` 是**做什么**的权威，这份是**怎么做**的权威。
---

# Folio 工程约定

> 每个会话开工前读一遍。`docs/DESIGN.md` 是**做什么**的权威，这份是**怎么做**的权威。
>
> **规则分三级，来源如实标注**——因为本文件自己要求"注释必须说真话"：
>
> - **【事故】** 这个项目真的踩过、在 `docs/reviews/` 里有案可查。违反过一次就付过一次返工。
> - **【预防】** M0 新立的原则，**没有**对应事故。它们是判断，不是血泪，可以讨论。
> - **【提示】** 审查时的参考，不是硬门。

## 零、最高原则

### 【事故】以规格和 upstream 契约为准推导，不要照复现步骤做局部修补

排第一，因为它造成过两次返工，一次比一次严重：

- 审核方给的一条复现步骤**本身就是错的**（称"APC 载荷里的 `ESC[3J` 会清空历史"，但按 DEC STD 070，载荷里的 ESC 本就终止 APC，那个 ED3 是真命令）。照它修 → 改坏 VT 状态机 → **未终止的 DCS/APC 永久卡死终端，连 RIS 都救不回**，比原 bug 严重得多。
- 另一次：复现步骤只用 `row 0`，锚点 rebase 就只修了 row 0，其他行静默漂移。

**正确做法**：回到 DESIGN.md / DEC STD 070 / upstream 源码推导出**正确行为**，修那个。修完复现步骤仍不过 → **说出来**，很可能是步骤错了。

第三轮 Codex 正是这么做的（面对那条错步骤，选择回到标准推导，让 OSC/APC 归于同一模型），这是它能通过的主要原因。

## 一、通用解

### 【事故】不要启发式

真实案例：曾用"前后网格切片相等"猜 resize 移除了哪些行——重复行（进度条、分隔线）就歧义。正确解是让 alacritty 直接上报。

**当"猜"能通过测试时，它仍然是错的**：它只是恰好没被你的样本证伪。

### 【预防】校验放在系统边界

PTY 字节流、用户输入、外部 API、配置文件是边界，要校验；内部模块之间靠类型系统。

来源是项目所有者的全局偏好（不在本仓库，Codex 看不到，所以写在这里），**不是审核事故**。**但当前代码与它冲突**：`bt-viewport:116`、`bt-transcript:165` 等公开构造器直接 `assert!` 调用方参数——那既不是边界，也和第四节的 panic 原则打架。M0 决定：改 `NonZero*` / `Result`，还是承认它们是边界。

## 二、vendor

### 【事故】`vendor/alacritty_terminal` 必须留在 `workspace.members`

**CI 用 `cargo metadata` 检查**（不是 grep——manifest 里这个路径出现两次，members 和 `[patch.crates-io]`，grep 会被 patch 行骗过，把 member 删了照样过）。

理由：vendor 的 180 个 upstream 测试是**我们补丁没破坏 VT 语义的关键回归防线**。它不在 members 里时 `cargo test --workspace` 根本不跑它——一个真实回归就是这样溜过两轮审核的（`tests::intermediate_reset_on_dcs_exit` 挂了没人知道）。

同时 CI 检查：路径指向 `vendor/`、版本精确等于 `0.26.0`（依赖也写 `=0.26.0`——否则 crates.io 上的 0.26.x 会 out-resolve 掉本地 patch，静默把捕获钩子从构建里拿掉）。

### 【事故】policy 不进 vendor，**也不进适配层**

什么该冻结、什么该丢弃是 `bt-term` 的判断；vendor 只如实上报发生了什么。三轮审核每轮都逐 hunk 核对过这条。

**但射程短了一格**：`TerminalAdapter::decorations_allowed()` / `mark_resize_quiescent()` 是**装饰调度策略**，却住在本该只做 alacritty 适配的 `TerminalAdapter` 里。policy 没进 vendor，但进了 **vendor 的门面**——R11' 想隔离的那层照样被污染了。

规则：**vendor 适配模块（`adapter.rs` / `cell_capture.rs`）不得依赖 `bt-doc` / `bt-detect` / `bt-viewport`**（CI 可 grep 它们的 `use`）。判据：适配层只回答"alacritty 发生了什么"，不回答"我们要拿它怎么办"。

### 【事故】upstream 的内存布局不得跨过适配层

冻结的转录里**不得**出现 upstream 的裸 bit / 裸 tag。现状违反：

- `bt-transcript:33` `flags: u16` 直接存 `cell.flags.bits()` ——**硬编码了 alacritty 的 bit 布局**
- `bt-transcript:34-35` 颜色是 `encode_color` 手打的 `0x01/0x02/0x03` tag，**全 workspace 没有解码器、没有测试**

§2 的"升级前 diff `shrink_lines`"**盖不到这里**：upstream 重排 `Flags` 的位，测试全绿，而历史的含义静默改变。

规则：转录层的每个编码类型必须是 **bt-transcript 自己定义**的（自己的 enum / bitflags），翻译和显式的位映射留在 `bt-term`。这样 upstream 改布局是**编译失败**而不是静默漂移。

**"只写数据"是可以 grep 的坏味道：任何 `encode_*` 没有配套的 `decode_*` + round-trip 测试，就是。**

### 【提示】补丁默认只在 `src/term/mod.rs`

当前补丁面：单文件 15 hunk +109/-4。越界不是禁止，但**需要专项说明**（upstream 结构变化可能逼你换地方）。每次改 vendor 更新报告里的补丁面统计。

### 【事故】升级 alacritty 前强制 diff `grid/resize.rs::shrink_lines`

`Term::resize` 镜像了它的公式。224 个测试能兜住行为，**兜不住上游公式静默漂移**。

## 三、测试

### 【事故】门测试从字节跑完整链路

首次交付把"零件级单测"误报成"三门全过"：24 个测试全是手工构造入参调一个函数，六个 crate **从未合并运行过**。规格的价值在协议的**接缝**上，而接缝正是没被测的部分。

**限于 gate / 集成测试**（G1/G2/G3 必须从"喂 VT 字节"到"装饰记录状态"，中间不许手工构造）。**这不否定单元测试**——零件级测试有它的价值，只是不能冒充门。

### 【事故】默认值会掩盖 bug —— 这是一个**族**，不是一个案例

所有锚点测试都用 `live_anchor(0, _)`，而 **row 0 恰好是唯一被旧代码覆盖的行**。于是"锚点从不 rebase"这个 BLOCKER 躲过了两轮审核和全部测试。

`live_anchor(0,_)` 被抓到，是因为它是**参数**。同族的另外三个躲过了三轮审核，因为它们是**字段默认值**：

| 恒定的默认值 | 掩盖了什么 |
|---|---|
| `live_anchor()` 的 `GridGeneration(1)` | 恒等于 session 初始 generation → 锚点 generation 永远"匹配"。配合"从不比较"，**generation 逻辑整个写反也不会红** |
| `LayoutKey` 的 `dpi/font/theme = 1000/1/1` | 4 个字段只有 1 个被测过。**把 LayoutKey 换成只剩 `width_cells` 的 struct，全部测试照样绿** |
| `CapturedRow::plain()` 的默认样式 | bt-term 所有测试只比 `.text` → `alacritty Cell → CapturedCell` 的翻译**从未从 VT 字节验证过**，样式是只写数据 |

规则：

1. **参数**：当它会改变控制流、命名空间、边界或生命周期时，取默认值/首个值的测试必须补非默认值的对偶用例。
2. **多字段的键**：每个字段都必须有一个能让它**单独失效**的测试。
3. **只写字段 = 死规格**：规格要求携带、但全 workspace 从未被读过的字段（现状：锚点 `generation`、`HistoryEntry.source`），**要么接线，要么从 DESIGN 里删**。每次 review 人工过一遍"新增字段有没有消费者"——`dead_code` 盖不到 pub 字段。

写测试时反问：**这个取值是不是恰好走了最简单的那条路径？**

### 【事故】断言要能失败——**射程覆盖构建配置本身**

分包不变性测试曾是**恒真**的：`feed()` 内部逐字节 `advance`，分包边界在构造上不可观测，那个断言**永远不可能红**。

**同一个 bug 已经出现三次，每次换个马甲**：

1. 恒真的分包断言
2. vendor 里那句"IL/DL 绝不触发"的假注释（测试恰好绕开了那个 bug）
3. **本规范的首版**：写了一大堆 lint，但 6 个 crate 都没写 `[lints] workspace = true`，**一条都没生效**，`clippy -D warnings` 是个永远绿的空门

所以这条规则的射程不止测试：**任何声称在挡住什么的东西，都必须先证明它会红。**

- CI 有一个**反向测试**：往 crate 里种一个 `todo!()`，clippy 必须红。护栏加进来的那天就要证明它会响。
- 写完断言问一句：**什么情况下它会红？** 答不上来就是假测试。
- 写完 CI job 问一句：**它挡过什么？** 答不上来就是空门。

### 【事故】驱动真实子进程的测试，超时按"孩子静默多久"算，不按墙钟总额

（2026-08-20，分支 `test-env-immunity`；技术细节见 `docs/DESIGN.md` §7.1.6c-3b 尾部。）

`bt-app` 的 `real_powershell_input_reaches_a_viewport_owned_frame` 与 `bt-pty` 的 `sidecar_resize_keeps_history_navigation_on_a_clean_prompt_line` 反复"随机"挂，一天里两个 agent 各自撞到。它们的等待全是**总额**：5s 等一行、10s 从头管到尾。这种超时量的不是被测的东西，是**这台机器当时有多忙**——闲时 32 次 0 失败，把二十四个自旋进程压满这台二十四线程的机器，同一支脚本立刻 8/8、13/16 全塌。改好之后把改前改后的两个测试二进制并排放进同一段负载里交替跑：**旧臂 7/8 挂，新臂 0/8**，新臂最慢一次熬到 48 秒才绿——那 48 秒正是"同样的字节，只是间隔更远"长的样子，也正是任何一个总额都买不到的东西。

规则：

1. **超时问"孩子停了没有"，不问"过去了多久"**：每读到一个字节预算就重置，另设一个远高于任何诚实开销的绝对天花板兜住"一直说话但永远说不到"。饿着的机器交付同样的字节，只是间隔更远，所以这个判据对负载免疫；真挂的孩子仍然在原来的秒数里红。
2. **不许把数字调大**。那买到的绿是把每一次真挂的代价一起乘上去，等于把不稳定藏起来。
3. **超时要报它等到了什么**：等了多久、其中静默多久、读了多少字节、握手到没到、屏幕长什么样。只说"失败"的超时会把下一个人送去查错的方向（这次就送错了：真凶是负载，被告是一条 env）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lulu-loopp/folio-terminal](https://github.com/lulu-loopp/folio-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
