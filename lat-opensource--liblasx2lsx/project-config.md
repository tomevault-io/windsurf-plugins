---
trigger: always_on
description: 提供 LASX（LoongArch SIMD）指令模拟的 C 库，通过 SIGILL 信号捕获和 LD_PRELOAD 实现。
---

# AGENTS.md - liblasx2lsx

提供 LASX（LoongArch SIMD）指令模拟的 C 库，通过 SIGILL 信号捕获和 LD_PRELOAD 实现。

## 项目结构

源代码按功能组织在 `src/` 目录下：

```
src/
├── include/          # 共享头文件
│   ├── debug.h
│   ├── loongarch-extcontext.h
│   ├── perf_idx.h
│   ├── perf_stats.h
│   ├── thread_data.h
│   ├── lasx_emu_private.h
│   ├── lasx_emu.h
│   ├── lasx_arith.h
│   ├── lasx_logic.h
│   ├── lasx_mem.h
│   ├── lasx_imm.h
│   ├── lasx_muldiv.h
│   ├── lasx_extract.h
│   ├── lasx_bitmanip.h
│   ├── lasx_cmp.h
│   ├── lasx_fbasic.h
│   ├── lasx_fma.h
│   ├── lasx_fcmp.h
│   ├── lasx_perm.h
│   ├── lasx_extadd.h
│   ├── lasx_shift.h
│   ├── lagoon.h
│   └── lasx_interpret.h
├── emu/              # 模拟核心
│   ├── sigill_hook.c      # SIGILL 信号处理器
│   ├── thread_data.c      # pthread 线程本地存储
│   ├── lasx_emu.c         # 核心模拟入口和指令分发
│   ├── lasx_arith.c       # 算术运算
│   ├── lasx_logic.c       # 逻辑运算
│   ├── lasx_mem.c         # 内存操作
│   ├── lasx_imm.c         # 立即数操作
│   ├── lasx_muldiv.c      # 乘除法
│   ├── lasx_extract.c     # 提取操作
│   ├── lasx_bitmanip.c    # 位操作
│   ├── lasx_cmp.c         # 比较操作
│   ├── lasx_fbasic.c      # 基础浮点
│   ├── lasx_fma.c         # 融合乘加
│   ├── lasx_fcmp.c        # 浮点比较
│   ├── lasx_perm.c        # 置换操作
│   ├── lasx_extadd.c      # 扩展加法
│   ├── lasx_shift.c       # 移位操作
│   ├── lasx_shift_n.c     # 窄化移位
│   ├── lasx_shift_ni.c    # 窄化立即数移位
│   └── perf_stats.c       # 性能统计
├── interpret/        # 解释层
│   ├── lasx_interpret.c
│   ├── lasx_arith_interpret.c
│   ├── lasx_logic_interpret.c
│   ├── lasx_mem_interpret.c
│   ├── lasx_imm_interpret.c
│   ├── lasx_muldiv_interpret.c
│   ├── lasx_extract_interpret.c
│   ├── lasx_bitmanip_interpret.c
│   ├── lasx_cmp_interpret.c
│   ├── lasx_fbasic_interpret.c
│   ├── lasx_fma_interpret.c
│   ├── lasx_fcmp_interpret.c
│   ├── lasx_perm_interpret.c
│   ├── lasx_extadd_interpret.c
│   └── lasx_shift_interpret.c
└── lagoon.c          # LoongArch 汇编器/JIT

tools/                # 工具脚本
├── add_lsx_intrinsics.py
├── check_lasx_opcodes.py
├── check_rmw.py
├── convert_to_intrinsics.py
├── find_missing_instructions.py
├── list_missing.py
├── verify_coverage.py
├── zl.py
├── find_missing.sh
└── run_parallel_tests.sh

tests/                # 测试程序
random_test/          # 随机测试框架
```

## 编译命令

```bash
make all          # 编译全部：liblasx2lsx.so + 测试 + 调试版
make              # 等同于 make all
make cross        # 交叉编译为 LoongArch（设置 CROSS_COMPILE 前缀）
make clean        # 清理编译产物
```

**自动行为：**
- x86_64 主机：自动使用 loongarch64-unknown-linux-gnu-gcc 交叉编译
- LoongArch 主机：默认使用 gcc 本地编译

**关键编译标志：**
- `-DCONFIG_LOONGARCH_NEW_WORLD` - 新版内核（x86_64交叉编译默认启用，LoongArch 本地自动检测）

## 使用方法

库通过 LD_PRELOAD 加载：

```bash
LD_PRELOAD=./liblasx2lsx.so your_program
```

## 关键约束（重要！）

### 1. Switch结构与OP前缀对应关系

**核心规则：`shift = 32 - OP编号`**

每个 `case OPxx_XXX:` 必须放在对应的 switch 语句中：

| OP前缀 | Switch位移 | 正确写法 |
|--------|-----------|---------|
| OP14 | `instr >> 18` | `switch (instr >> 18) { case OP14_...` |
| OP15 | `instr >> 17` | `switch (instr >> 17) { case OP15_...` |
| OP17 | `instr >> 15` | `switch (instr >> 15) { case OP17_...` |
| OP22 | `instr >> 10` | `switch (instr >> 10) { case OP22_...` |
| OP24 | `instr >> 8` | `switch (instr >> 8) { case OP24_...` |

**违反规则的后果：**
- 指令无法正确区分，导致错误的指令被错误处理
- 即使覆盖率显示100%，实际运行也会出错

### 2. 代码格式化

使用 clang-format 统一格式：

```bash
clang-format -i src/emu/lasx_emu.c
```

配置见 `.clang-format` 文件。

**不要用 Python 脚本手动转换格式**，容易出错。

### 3. 覆盖率验证

添加新指令后，验证 switch 结构正确性：

```python
import re
with open('src/include/lasx_emu_private.h') as f:
    priv = f.read()
opcode_defs = {m.group(1): int(m.group(2)) for m in re.finditer(r'#define (OP(\d+)_\w+)', priv)}

with open('src/emu/lasx_emu.c') as f:
    c = f.read()

# 验证各switch
for m in re.finditer(r'switch \(instr >> (\d+)\)', c):
    shift = int(m.group(1))
    expected_op = 32 - shift
    block = c[m.start():m.end()+500]
    cases = re.findall(r'case (OP\d+_\w+):', block)
    wrong = [x for x in cases if x in opcode_defs and opcode_defs[x] != expected_op]
    print(f"shift={shift} OP{expected_op}: {len(cases)}条, 错误{len(wrong)}")
```

### 4. 已知警告

编译时 `lasx_fbasic.c` 的 strict-aliasing 警告是原代码问题，不影响功能，无需修复。

## 测试规则

### 测试Targets

```bash
make test              # 运行正常测试（使用模拟库）
make test-native      # 无模拟运行，验证测试代码正确性
make test-debug       # 运行已知问题的测试（调试用）
make test-debug-native # 无模拟运行debug测试
```

### 测试目录结构

- `tests/` - 所有测试源文件
- 测试文件命名：`test_*.c`

### 添加新测试步骤

1. **创建测试文件**：在 `tests/` 目录下创建 `test_xxx.c`

2. **编写测试代码**：
   - 使用 LoongArch 内联汇编调用 LASX 指令
   - 内存布局：源A(32B) + 源B(32B) + 结果(32B)
   - 使用 `xvld` 加载，`xvst` 存储

3. **寄存器使用**：
   - `$xr0`, `$xr1`, `$xr2` 等向量寄存器
   - 指令格式：`xvld $xrN, offset(base)`
   - 指令格式：`xvadd.$xrN, $xrM, $xrK`

4. **添加到 Makefile**：
   - 正常测试：添加到 `TESTS` 变量
   - 已知问题测试：添加到 `TESTS_DEBUG` 变量

### 常见问题排查

#### 1. test-native 失败但 test 通过

**可能原因**：
- 测试预期值计算错误（测试代码bug）
- QEMU LASX 实现与硬件行为不同
- 指令语义理解错误

**排查步骤**：
```bash
# 1. 先用 test-native 验证是测试代码问题还是模拟器问题
make test-native

# 2. 如果 test-native 通过，说明模拟器实现有问题
# 3. 如果 test-native 也失败，检查预期值计算

# 4. 手动验证预期值
# 例如：xvandn.v 语义为 rd = rk & ~rj，不是 a & ~b
```

#### 2. 常见指令语义差异

| 指令 | 正确语义 | 常见错误 |
|------|---------|---------|
| `xvandn.v` | `rd = rk & ~rj` | `rd = rj & ~rk` |
| `xvorn.v` | `rd = rk \| ~rj` | `rd = rj \| ~rk` |
| `xvclo.h` | 输出 8-bit 计数/16-bit | 按 16-bit 全宽输出 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lat-opensource/liblasx2lsx](https://github.com/lat-opensource/liblasx2lsx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
