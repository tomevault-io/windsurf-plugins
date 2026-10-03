---
trigger: always_on
description: Mecon 是一款基于 Kotlin Multiplatform 的专业音乐分析应用，集乐谱编辑、理论分析与 AI 辅助创作于一体。当前以桌面端（Compose for Desktop）为主，Web 五线谱编辑通过共享 KMP 会话提供。
---

# AGENTS.md - Mecon 开发规范

## 项目概述

Mecon 是一款基于 Kotlin Multiplatform 的专业音乐分析应用，集乐谱编辑、理论分析与 AI 辅助创作于一体。当前以桌面端（Compose for Desktop）为主，Web 五线谱编辑通过共享 KMP 会话提供。

**技术栈**：KMP 2.1.0 · Compose for Desktop · Gradle 9.0 · kotlinx.serialization + kaml · Coroutines + Flow

详细架构见 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)，完整文档索引见 [docs/README.md](docs/README.md)。

## 核心设计原则

1. **不可变数据**：所有数据类使用 `data class` / `value class`，字段用 `val`
2. **四层架构**：严格遵守 `Storage → Runtime → Computed → Render Geometry` 层级划分，Renderer 只负责排版，不生成乐谱元素——见下方 ⚠️
3. **类型安全**：用 `@JvmInline value class` 包装基本类型（`EventId` / `TrackId` 等），避免参数混淆
4. **避免重复逻辑**：映射逻辑放在与之最相关的位置（如枚举类型本身），其他位置委托访问；不适合直接复用时先询问是否重构
5. **序列化**：存储层数据类加 `@Serializable`；Storage 层只含源字段，不含对象引用

## ⚠️ 乐谱编辑功能必须同步接入多端

完整接入顺序与验收门禁见
[docs/score-editing-multiplatform.md](docs/score-editing-multiplatform.md)。以下为强制约束：

- **共享会话是唯一业务入口**：新增乐谱编辑能力先在
  `features/score-editing/src/commonMain/` 定义 `ScoreEditIntent` 并接入 `ScoreEditingSession`；
  需要新算法时放入 `core/.../engine/edit/`。桌面、Web、后续移动端只做平台 adapter，不得各写
  音乐规则、状态变换或 undo/redo。
- **Web 只保留轻量壳层**：React/JavaScript 可负责控件、文件/恢复、Canvas/SVG 命中、pointer
  像素到稳定 ID/音乐坐标的映射、瞬时 preview、键盘/Web MIDI adapter；所有持久化写入必须以
  普通 JSON intent 进入 Worker，再经 Kotlin/JS facade 调用 `ScoreEditingSession`。禁止在 Web
  直接改 `StorageScore`，或实现时值拆分、变音记号、符杠、连音组等业务逻辑。
- **UI 过滤不是业务校验**：平台可为体验过滤候选，但 session 必须独立校验 revision、目标存在性、
  参数与结构约束；像素、数组下标和帧内对象引用不得进入 intent。
- **一次功能必须完成整条链**：同步核对数据模型/兼容读取、core immutable edit、协议 codec、session
  effect 与历史边界、Computed 生成职责、renderer metadata/hit box、continuous/paginated splice、
  桌面 adapter、Web pointer/keyboard 入口和相关文档。修改 Storage 模型仍须先更新
  `docs/data_model/`。
- **跨端等价是完成条件**：JVM/JS 重放同一份 `features/score-editing/testdata/intent-trace.json`
  比较 score、revision、selection、effect、`nextInputPosition` 与 `scoreChanged`；覆盖 stale/no-op、
  失败原子性、单历史项、undo/redo 选择恢复和 render hint。**新能力必须往该 trace 追加步骤**，
  否则不受跨端保护；重刷只从 JVM 侧 `-Pscoreediting.trace.write=true`。Web 还须有真实
  Playwright pointer/keyboard 路径；涉及文件时用浏览器导出的 `.mecon` 做桌面回读门禁。
- **桌面普通记谱入口已收敛，禁止回退**：`ScoreSession` / `EditableScoreHost` 的音符、结构、表情、
  几何与选择编辑都应 dispatch `ScoreEditIntent`。`applyStorageEdit` 仍服务插件、配器、缩谱等尚未纳入
  score-editing 协议的文档域，不是新增记谱旁路的依据；新记谱能力一律走 `dispatchSharedEdit` + intent。
- 若某端明确不接入，必须在能力矩阵记录范围与原因；不得只完成桌面实现后静默遗漏 Web。

## ⚠️ 自由练习功能必须经共享会话

完整流程见 [docs/exploration/free-practice-extension-guide.md](docs/exploration/free-practice-extension-guide.md)，
Web 构建运行见 [docs/web-development.md](docs/web-development.md)。以下为强制约束：

- 自由练习的持久化操作、统一选择、历史、后台结果与 typed view 只在
  `features/free-practice/commonMain` 定义；Desktop/Web 均 dispatch `FreePracticeIntent`。
- 普通谱面编辑用 `FreePracticeIntent.Score` 包裹内层 `ScoreEditIntent`；复音上限、手工事件来源和
  workspace/score 原子历史由 `FreePracticeSession` commit policy 负责，平台不得补第二次提交。
- Web 完整谱面与自由练习必须复用 `@mecon/web-renderer/editor/react` 的 `ScoreEditor` 和
  `useScoreEditorController`。工具栏通过 profile/hidden/slot 配置，禁止在应用内复制 surface、controller、
  inspector 或音乐命令。
- 重 CPU 写作、教学目录和 finding 使用带 requestId/baseRevision/fingerprint 的独立后台 channel；
  结果必须回到 session 校验，React 不直接接收并合并领域结果。
- 新能力必须追加 `features/free-practice/testdata/practice-trace.json` 并让 JVM/Kotlin-JS 重放同一流程；
  当前该 trace 由开发者显式编辑并先经 JVM 校验，`-Pfreepractice.trace.write=true` 尚无生成器。
- **后台崩溃必须走共享失败通道**：后台 channel 抛异常 / worker 死亡时，平台只报错是不够的——请求
  仍挂在 session 上，工作台会永远停在 `RUNNING` 且拒绝交互。一律用
  `PracticeBackgroundFailure(requestId, reason)` 调
  `applyBackgroundFailure` / `applyTeachingCatalogFailure` / `applyFindingFailure`，由 session 回退到
  最后一次提交的状态并给出 `freePractice.*.failed`。桌面捕获 `Throwable`（`CancellationException`
  透传），Web 为每个 search worker 挂 `onerror` 并路由 error 消息。详见
  [docs/exploration/free-practice-auto-writing.md](docs/exploration/free-practice-auto-writing.md) §8.1。
- **共享代码里 `Map.Entry` 不得跨越 map 结构性修改**：Kotlin/JS 的 entry 是回指哈希表的活引用，
  `clear()`/插入新键之后再读 `key`/`value` 会抛
  “The backing map has been modified after this entry was obtained.”，而 JVM 完全正常——这类缺陷
  只在 Web 端暴露。`entries.toList()` / `sortedWith` / `asSequence()` 保留的都是活引用；先
  `map { it.key to it.value }` 拷贝再改表。见
  [docs/theory/dynamic-programming-solver.md](docs/theory/dynamic-programming-solver.md) §6.1。
- 钢琴卷轴当前明确保留桌面实现；除非任务明确把它纳入范围，不要借自由练习改动扩张或复制其旁路。

## ⚠️ Renderer 与 Computed 层职责划分

**Computed 层**决定"是否生成"每个乐谱元素（小节线、谱号、调号、临时记号等），输出 `ComputedBarline / ComputedClef / ComputedKeySignature / ComputedTimeSignature`。

**Renderer 层**只做排版：从 `ComputedScore` 读取元素，计算坐标与间距，生成 `RenderCommand`。

- ❌ 不在 Renderer 中判断"是否需要小节线"
- ❌ 不在 Renderer 中计算临时记号、符杠分组等音乐逻辑
- ✅ 发现 Renderer 中有元素生成逻辑 → 迁移到 `ComputeEngine`

### 区间符号排版与跨行吸附

- 房子、8va/8vb、发夹、渐变速度等区间符号必须进入
  `StaffAttachmentLayoutComputer` 的统一区间附件管线，参与横向碰撞、行优先级、系统拆段和
  staff extra extent 计算；禁止在 `StructuralElementRenderer` 中用固定 Y 单独绘制房子。
- 上方区间符号的行优先级由附件类型集中定义：房子位于其他区间符号最外层（最上方），
  不在各 Renderer 中各自添加常量偏移。
- 拖动区间符号或导航记号并吸附小节线时，先用各系统五线谱核心区
  （`staff centerY ± 2 staff spaces`）锁定指针所在行，再只在该系统的 measure bounds 中
  寻找最近边界；系统核心区必须投影到最终显示坐标后与原始指针比较，分页模式下禁止先把
  指针转换为全局谱面 Y 再反推系统。禁止使用包含附加符号/加线音的扩张
  `SystemNode.topY/bottomY` 判定行；扩张带可能互相重叠并导致吸附到相邻行。
- 导航记号跨系统拖动时，预览位移包含源系统到目标系统的 Y 距离；提交到目标小节线后必须
  扣除 `targetAnchorY - sourceAnchorY`，只持久化相对目标谱表的局部 `dy`。否则目标系统锚点
  和存储偏移会各应用一次系统距离，表现为向下/向上都多跳一行。
- 分页增量附件候选按系统的 1-based `measureRange` 分片。房子的索引键必须使用

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muse-garden/Mecon](https://github.com/muse-garden/Mecon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
