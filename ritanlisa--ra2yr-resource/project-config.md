---
trigger: always_on
description: 逆向工程完整的 `gamemd.exe` C++ 源码并重建为独立可执行文件。
---

# RA2YR ReSource — 从 gamemd.exe 重建红色警戒 2/尤里的复仇源码

## 项目目标

逆向工程完整的 `gamemd.exe` C++ 源码并重建为独立可执行文件。

目标文件：32 位 Windows PE，MSVC 6.0 编译，约 7.6MB，基址 0x400000，含 19,067 个函数。

| 阶段 | 目标 | 产出 |
|------|------|------|
| Phase 1 (当前) | 完整逆向重建，功能对等原版 | 可直接替换的 Win32 EXE (`gamemd.exe`) |
| Phase 2 (远期) | 现代化重构 (现代渲染 / AI Bot / 原生模组系统) | 跨平台现代游戏引擎 |

当前构建产物：可执行文件 `gamemd.exe`，CMake + C++20。

## 项目专属行为准则

1. **只操作指定的文件。** 不修改未要求修改的文件。
2. **`D:\RA2MD\` 只能写入 `hook_dll.dll`。** 游戏目录不能放置任何其他文件。
3. **`D:\RA2MD\` 只能有一个 `hook_dll.dll`。** Syringe 自动加载所有带 `.syhks00` section 的 DLL，多个变体会导致 hook 冲突。
4. **`gen/*.cpp` 不在 git 跟踪中。** 切换分支后必须 `python gen_reverse_hooks.py` 重新生成。
5. **`PostProcStub.asm` 不添加任何新 EXTERN。** 新符号会偏移 DLL 导出序数，导致所有 hook 静默失效。
6. **REVERSE 管道相关文件改动需用户审查。** 见下方"REVERSE 管道保护文件"列表。
7. **测试验收最低标准：进程存活 15 秒 + `comparisonResult.log` 生成。** 两者缺一不可。

### 故障排除最低标准

任何崩溃报告必须包含：

- [ ] EIP 对应的反汇编（用 .map / dumpbin）
- [ ] 完整栈 dump（至少 64 字节）
- [ ] 寄存器全状态
- [ ] 复现率（N 次测试中崩 M 次）
- [ ] 对照实验（上一稳定版本 vs 当前版本崩溃率对比）
- [ ] 自己改动的代码 diff（最近 3 个 commit）

缺任何一项，禁止使用"根因是 X"这类确定性结论。可以提推测，但必须明确标注为推测。

### REVERSE 管道保护文件

以下文件构成对拍验证管道的核心基础设施。**未经用户明确许可，不得修改其中任何一个。**
违反此规则修改这些文件并导致"编译通过"或"测试通过"的假象，属于蓄意绕过验证的行为。

#### 生成器（代码逻辑）

- `injectFunctionTest/gen_reverse_hooks.py`   — REVERSE 扫描、钩子体生成、幂等判定集成
- `injectFunctionTest/gen_re_impl.py`          — RE_* 函数模板（修改此文件而非 gen/re_impl.cpp）

#### 运行时基础设施

- `injectFunctionTest/PostProcStub.asm`        — slot stack push/pop + LogComparison 调用
- `injectFunctionTest/tls_storage.h`           — ShadowSlot 结构体（slot stack + depth + txn）
- `injectFunctionTest/shadow_txn.h` / `.cpp`   — VEH handler + ShadowTransaction
- `injectFunctionTest/hook_template.hpp`       — 共享格式定义（Hex8/Fmt/FmtArr/WrFile）
- `injectFunctionTest/hook_main.cpp`           — DllMain + ExeRun hook（g_owner_tid）
- `injectFunctionTest/render_hooks.cpp`        — DSurface::Blit hook（UI 追踪）
- `injectFunctionTest/headless_server.cpp`     — TCP :25400 调试接口

#### 生成产物（gitignore，每次构建重新生成）

- `injectFunctionTest/gen/reverse_hooks.cpp`   — 钩子入口 + write_entry + 格式化
- `injectFunctionTest/gen/reverse_check.cpp`   — completed 校验
- `injectFunctionTest/gen/re_impl.cpp`         — RE_* 函数包装器

#### 分析管道

- `injectFunctionTest/analyze_idempotent.py`   — SCC 不动点幂等判定
- `injectFunctionTest/ida_extract_purity.py`   — IDA 直接 .data 扫描（Phase 1）
- `injectFunctionTest/ida_extract_phase3.py`   — IDA vtable 调用解析（Phase 3）
- `injectFunctionTest/gen_annotations.py`      — 三层标注生成
- `injectFunctionTest/annotations.json`        — 标注数据（提交 git）
- `injectFunctionTest/purity_effects.json`     — 纯度数据（提交 git）

#### 构建 + 元数据

- `injectFunctionTest/CMakeLists.txt`          — HOOK_REGEN_GENERATED option
- `injectFunctionTest/functions.json`          — 19K 函数元数据（completed/done/idempotent）
- `src/core/reverse_marker.hpp`     — REVERSE 宏定义
- `functionComparison.md`                      — 对拍管线设计文档

#### 锁定理由

这些文件的任何"便利性修改"（如临时 return true、跳过校验、改默认值、删错误检查）都会导致：

1. **静默退化**：对拍系统表面上"正常工作"，实则不再检测差异
2. **假阳性通过**：未完成的 RE 实现被标记为 verified，掩盖 bug
3. **管道腐化**：后续分析依赖的元数据被污染，连锁影响所有函数判定

修改策略：如需调整，**必须先向用户说明改什么、为什么、影响哪些函数**，获批准后再实施。
未经许可的修改将被视为绕过验证，对应的测试结果无效。

## 构建命令

```bash
# Windows EXE (VS 2022, 32-bit)
cmake -B build_win -G "Visual Studio 17 2022" -A Win32
cmake --build build_win

# hook DLL (Release, 必须输出 .map 文件)
cmake -B build_hook -G "Visual Studio 17 2022" -A Win32 -DHOOK_OUTPUT_DIR=D:/RA2MD
cmake --build build_hook --config Release

# 生成 REVERSE 钩子代码
cd injectFunctionTest && python gen_reverse_hooks.py

# Linux 验证编译 (x86_64 ELF, 仅代码验证, 不能运行游戏)
cmake -B build_linux -G "Unix Makefiles"
cmake --build build_linux
```

## comparisonResult.log 调试工作流

这是 Inject/Replace 模式下验证 RE 实现正确性的核心流程。

### 日志结构

```text
============ Different Compares ============  ← RE ≠ 原版，需修复
  [function_sig-0xADDR]
  Call N: caller()<-ret_addr
    Input:  ...                          ← 输入参数
    Return: hook=false != original=true  ← 差异

================ Captures ================   ← Capture 模式（仅记录）

============= Same Compares ==============   ← RE = 原版，已验证
  [function_sig-0xADDR]
    Return: hook=original=X                 ← 一致

============== None Calls ================  ← 零次调用的函数
```

### 调试步骤

```text
1. 运行游戏（菜单进入 + 主界面 15s+）
   → comparisonResult.log 生成

2. 检查 Different Compares 下的函数
   → 提取输入样本（Input 行）

3. 对每个差异函数：
    a. IDA 反编译原版 → 对比 RE 实现
    b. 重点关注: outcode 顺序、边界条件、浮点运算差异
    c. 修改 gen_re_impl.py 的模板（不是 gen/re_impl.cpp，后者被 gitignore 会被覆盖）
    d. python gen_reverse_hooks.py && cmake --build build_hook

4. 当函数从 Different → Same 迁移：
    a. functions.json: hook.done = true
    b. 源文件: REVERSE 标记改为 "None"
    c. 提交
```

### completed 硬性规则

**`completed=true` 只能设置在完全逆向完成的函数上。**

| 字段 | 含义 | 设置时机 |
|------|------|---------|
| `completed` | RE 实现完整（代码层面） | IDA 反编译→完整实现→编译通过 |
| `done` | 对拍验证通过（行为层面） | comparisonResult.log Same Compares |

**强制约束**：

1. `completed=true` 必须满足：函数在 `src/` 下有完整 C++ 实现（非 stub/return 0/TODO），且编译 0 error 0 warning。
2. 任何 Inject/Replace 钩子若对应函数 `completed=false` → `gen_reverse_hooks.py` 报 ERROR，构建中止。
3. 设置 `completed=true` 前必须对照 IDA 反编译确认所有分支、边界、返回路径均已覆盖。
4. 禁止批量标记 `completed=true`——每个函数须逐个核实。
5. 违反以上规则导致的崩溃由标记者承担全责。

### 常见差异原因

| 原因 | 示例 | 修复 |
|------|------|------|
| RE stub 返回 0 | `return 0; // TODO` | 实现完整算法 |
| outcode 顺序/边界不同 | `clip_r-1` vs `clip_r` | 匹配 IDA 反编译 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ritanlisa/RA2YR_ReSource](https://github.com/Ritanlisa/RA2YR_ReSource) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
