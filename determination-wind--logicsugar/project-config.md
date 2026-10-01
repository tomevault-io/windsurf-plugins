---
trigger: always_on
description: Mindustry Java mod that adds structured `for` / `while` / `switch` / function blocks to the
---

# LogicSugar — Agent Notes

Mindustry Java mod that adds structured `for` / `while` / `switch` / function blocks to the
logic editor while storing vanilla-compatible mlog. Workspace-wide rules (build modes, dev
identity `LogicSugar-dev / 0.0.0`, release safety) live in the parent `codex/AGENTS.md`;
this file only adds what is specific to this project.

## 兼容底线（项目所有者明文要求，改任何功能前先读）

**LogicSugar 的硬底线：多人联机环境下必须兼容原版客户端。** 保存到处理器的代码在任何
原版客户端上都要能解析、能运行；这是整个 mod 的存在前提，优先级高于一切新功能。

- **调试类功能只在单机启用**：凡是会改变保存产物语义的功能（目前是调试断言构建
  `AssertEmit=emit`），必须在代码层限定为 `!Vars.net.active()`（单机/编辑器）才生效——
  联机（已连接或自建）一律回落原版行为。纯展示类功能（如处理器状态指示、单位 flag 显示、变量复制按钮）
  不产生存档差异，不受此限。不提供改变处理器指令预算的能力：指令上限覆盖曾试做后被移除
  （commit ea97e00），处理器保存产物恒 ≤1000 条是硬不变式，勿再引入。
- **函数库不是处理器产物**：全局函数库 `functions.txt` 不算"保存到处理器的代码"，不受上面
  1000 条硬不变式约束；它有独立上限 `SugarFunctions.libraryInstructionLimit`（当前 10000 条
  语句）。库文本必须用 `SugarFunctions.readLibrary` 解析（临时抬高 `LExecutor.maxInstructions`
  后 `finally` 还原，绝不放开处理器检查），超限由 `libraryOverLimit` 在保存/打开编辑器时明确
  拒绝。使用库函数的处理器产物仍然 ≤1000 条，只嵌入被调用到的函数子集。
- **残余风险必须写进文档**：单机里创建的越界内容（>1000 条程序、带 assert 指令的调试
  构建）若之后被分享到多人环境，原版客户端仍会截断/清空/静默降级——代码无法阻止分享，
  只能靠设置描述与文档把后果讲清（见 bundle 的 maxInstructions/assertEmit 描述）。
- 新功能提案先按此底线分类：不碰保存产物 → 正常实现；碰保存产物 → 必须加联机门禁，
  并在 bundle 与 `docs/architecture.md` 说明单机限定。
- **上游同步基线**：断言/断点子系统对齐 cardillan/MlogAssertions **v0.11.1**（本地副本
  `../_upstream/MlogAssertions-pr`，`git fetch upstream` 更新）。上游的指令上限覆盖
  （`max-instructions`）**不得**移植；上游 v0.10/0.11 的 Vars / Memory / Properties 界面与
  快照子系统（`snapshot` 指令、`Snapshots` 数据层）本身不在同步范围内（它们不改变保存产物，
  属独立功能，若要移植需另议）。线格式例外有两处：`asserttype` 的 `null` 类型是 LogicSugar
  扩展（上游 `AssertionDataType` 不识别），以及上游 v0.10 起 `asserttype` 的 token 顺序为
  `<type> <value>`（旧序仍可读，保存统一写新序）——改这一块前先看
  `SugarAsserts.AssertTypeCard` 的注释与 `assertTypeTest`。

## Build & Test

```powershell
cd LogicSugar; ./gradlew check        # runs selfTest, ifElseTest, decompileTest, reconstructionTest, reconstructionMatrixTest, recoveryPredicateTest,
                                      # shortCircuitTest, crossLoaderTest, boxSelectTest, cfgTest, lintTest,
                                      # varClipboardTest, processorStatusTest, unitFlagsTest, assertTest, assertTypeTest, assertMessageTest, arrayTest,
                                      # arrayBulkTest, dataFrameworkTest, recordTest, containerTest, bitsetTest,
                                      # mapTest, setTest, listHeapTest, chainTest, dataSubsystemTest, dataCallTest, editHistoryTest,
                                      # bottomBarLayoutTest, escapePreviewTest, v160SensorAccessTest, funclibLimitTest, dataRuntimeTest,
                                      # exprTextImportTest, exprCardTest, conditionLabelTest, editorConflictTest, textWrapTest,
                                      # statementClipboardTest, originTest, counterJumpIndexTest, unitControlTest, spanTest
./gradlew check jar                   # build + dev jar at build/libs/ (copy to 构建/LogicSugar/LogicSugar-dev.jar)
```

The self-tests are `main()`-based JavaExec tasks (no JUnit runner). New regression coverage
should follow that convention and be added to `check.dependsOn`.

## Mod class loader vs. game classes (critical, easy to miss)

At runtime this mod's classes load through a **mod class loader** while `mindustry.logic.*`
game classes load through the app loader. Same package name, **different runtime packages**:

- `protected` / package-private members of game classes (`LStatement.field`,
  `LStatement.showSelect`, `LogicDialog.privileged`/`consumer`, …) may only be accessed
  1. from **inside our own subclasses** — subclass access to protected members is legal
     across loaders (this is why `SugarStatement.fieldsHint` works), or
  2. **reflectively** via `Field.setAccessible(true)` (see `SugarLogicDialog.privilegedField`
     for the established pattern).
- A static helper in `SugarStatements` (or any non-subclass of ours) calling those members
  **compiles fine** — javac only sees the source-level package match — and throws
  `IllegalAccessError` at runtime the first time the UI renders. This bit the first
  `addCompactOp` implementation (crash 2026-08-27).
- Rule: any helper that touches protected game-class members must be an instance method on
  the `SugarStatement` subclass (public if a static editor like
  `rebuildConditionEditor` needs to call it). Static code may only use public game API.
- `crossLoaderTest` simulates this loader split headlessly (child-first loader defines our
  classes, `LStatement` stays on the parent) and fails with guidance if the pattern
  regresses.

## Decompiler safety gate

`SugarDecompiler` may only return recovered Sugar source after recompiling it and comparing
the normalized instruction stream against the input (`verify`). Anything unrecognized falls
back to raw vanilla statements. When touching recovery logic, keep every new pattern behind
that gate; failure direction must always be "show more vanilla code", never "rewrite unknown
programs". Two properties of the gate are load-bearing:

- `verify` compiles each candidate across the full FuncMode x SwitchStrategy matrix, because
  the program may have been saved under different user settings.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeterMination-Wind/LogicSugar](https://github.com/DeterMination-Wind/LogicSugar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
