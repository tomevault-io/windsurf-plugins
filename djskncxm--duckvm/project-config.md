---
trigger: always_on
description: 基于文本匹配的 VMP Handler 还原工具。通过解析 ARM64 trace 日志，利用 VMP dispatch 模式（`LDR [X24,...] + BR`）自动识别 handler 入口，遇到未知 handler 停下让人工命名，逐步还原完整 handler 序列。
---

# DuckReVM

基于文本匹配的 VMP Handler 还原工具。通过解析 ARM64 trace 日志，利用 VMP dispatch 模式（`LDR [X24,...] + BR`）自动识别 handler 入口，遇到未知 handler 停下让人工命名，逐步还原完整 handler 序列。

**核心原则**：
- 这个方案不考虑 B 指令，一份 trace 就是一条执行流，也就是一份算法，不再有其他情况
- 这是**解释器**不是文本解析器 — 解释 VM 指令语义，不要硬匹配 ARM64
- 每条指令位置是固定的，变了含义就不一样
- 必须理解 handler 本质，然后去读实际 trace 数据

## handler 格式

几乎都可以套用

```text
.text:00000000002B05F4 vm_handler_STORE_REG                    ; DATA XREF: .data.rel.ro:00000000003680E0↓o
.text:00000000002B05F4                 LDR             W8, [X0] ; vm_op_store: *(regs[base] + (int16_t)offset) = regs[value], PC += 1
.text:00000000002B05F8                 MOV             W13, #0x30 ; '0' ; W13 = 48 (每条指令元数据大小，用于计算下一条指令地址)
.text:00000000002B05FC                 LDR             W9, [X19] ; W9 = PC
.text:00000000002B0600                 LDR             X12, [X20] ; X12 = *instruction_stream = 指令元数据基址
.text:00000000002B0604                 AND             X10, X8, #0xFF ; X10 = BYTE0(op) = base寄存器索引
.text:00000000002B0608                 UBFX            X11, X8, #8, #8 ; X11 = BYTE1(op) = value寄存器索引
.text:00000000002B060C                 ADD             W9, W9, #1 ; W9 = PC + 1
.text:00000000002B0610                 SBFX            X8, X8, #0x10, #0x10 ; X8 = sign_extend(HIWORD(op)) = 有符号内存偏移
.text:00000000002B0614                 LDR             X10, [X23,X10,LSL#3] ; X10 = regs[base] (基址值)
.text:00000000002B0618                 NOP
.text:00000000002B061C                 SMADDL          X0, W9, W13, X12 ; X0 = (PC+1)*48 + metadata_base = 下一条指令元数据地址
.text:00000000002B0620                 LDR             X11, [X23,X11,LSL#3] ; X11 = regs[value] (要写入的值)
.text:00000000002B0624                 STR             W9, [X19] ; PC = PC + 1
.text:00000000002B0628                 STR             X11, [X8,X10] ; >>> 核心操作: *(regs[base] + offset) = regs[value] (STORE)
.text:00000000002B062C                 LDR             X8, [X0,#0x28] ; X8 = next_insn.metadata[0x28] = 下一个opcode索引
.text:00000000002B0630                 LDR             X8, [X24,X8,LSL#3] ; X8 = opcode_handler_table[next_opcode]
.text:00000000002B0634                 BR              X8      ; BR → 跳转下一个handler
```

## 已知陷阱

### 长度检查必须扣掉 end_addr

`interpret_handler()` 第 1084 行过滤掉了 `offset == cfg.end_addr` 的指令（即 BR），所以 `handler_insts.size()` 比 trace 中看到的指令数**少 1**。

简单 handler 的 `insts.size()` 通常在 9~15，长度检查用 `< 8` 即可。之前多次因为设 `< 16` 导致返回空串、handler 不出现在输出中。

### PAC/加密指针不是 bug

VMP 代码中常见高位非零的"巨值"指针（如 `0x9383fb018a0c23c2`）。这是 ARM64 PAC (Pointer Authentication) 的认证码或者 VMP 自己的指针加密，不是工具 bug。IR 注释应如实反映 trace 中的值。

### RegAccess 字段名

`RegAccess` 的寄存器名字段是 `reg_name`，不是 `reg`。

### STORE32 的 offset 是无符号提取

某些 compound handler 中的 STORE32 用 `lsl #0x20 + and 0xFFFF000000000000` 提取 offset（无符号），与其他 handler 的 `sign_extend_16` 不同。写解释器时需看 IDA 确认。


## 构建

```bash
./build.sh
# 或手动:
mkdir -p build && cd build && cmake .. -DCMAKE_BUILD_TYPE=Release && make -j$(nproc)
```

产物: `./build/duckrevm`

## 使用

**基础模式**（还原 handler 序列）：
```bash
./build/duckrevm trace_logs/code.log
```

工作流: 运行 → 遇到未知 handler 停下 → 去 IDA 分析该地址 → 添加到 `trace_logs/handlers_config.json` → 重新运行，重复直到还原完毕。

## 架构

- `src/trace_parser.cpp` - 解析 trace 文件，格式: `index : 0xaddr [0xoffset] "mnemonic operands" (registers)`
  - 自动提取寄存器读写信息到 `TraceInstruction.reads` 和 `TraceInstruction.writes`
- `src/handler_matcher.cpp` - 核心匹配引擎，识别 dispatch 模式、BL 折叠、嵌套 VMP 处理
- `src/output_writer.cpp` - 输出结果
- `src/handler_interpreter.cpp` - 具体的 handler 解释器插件
- `trace_logs/handlers_config.json` - handler 地址配置（含 start/end addr）

## 关键设计

- 使用 **offset 地址**匹配（非绝对地址），兼容不同基址
- **BL 折叠**: 遇到 BL 调用时用栈跟踪返回地址，折叠 native 函数内部指令，但保留嵌套 VMP handler
- **Dispatch 检测**: 前一条是 BR + 往前 5 条内有 `LDR Xn, [X24,...]` → 判定为 handler 入口
- **500 条指令上限**: 防止匹配跑飞，超限截断并警告
- **寄存器读写解析**: 解析时一次性提取所有 `(r)reg=value` 和 `(w)reg=value` 信息
- C++11，无第三方依赖，流式解析

## handlers_config.json 格式

```json
{
  "ctx": { "pc_reg": "x19", "ctx_base": "x23" },
  "handlers": [
    { "addr": "0x2c0cc0", "end_addr": "0x2c0d50", "name": "COMPOUND_STORE_INIT" }
  ]
}
```

`end_addr` 指定后精确匹配结束位置；未指定则扫描到 BR 指令结束。

## 解释器 (IR 生成)

接下来任务就是根据工具产出的未识别 handler 继续还原。

使用 IDA MCP 和 trace 日志进行，下面是完整工作流：

### 1. 运行工具找未识别 handler
```bash
./build/duckrevm trace_logs/code.log
```
工具会停在第一个未识别 handler 并输出：
```json
{"index": 10f93, "offset": 0x2b137c, "mnemonic":"ldr w8, [x19]"}
```

### 2. 提取 trace 指令序列
```bash
grep "0x2b137c" trace_logs/code.log -A 20
```
找到完整 handler 执行序列，识别 dispatch 边界（最后的 `br` 指令）。

### 3. 用 IDA MCP 分析语义
```python
mcp__ida-pro-mcp__disasm(addr="0x2b137c", max_instructions=20)
```
- 看 IDA 注释理解 handler 做了什么（如 `LOAD_INDIRECT`、`MOV_REG`）
- 关注关键指令：操作数读取、数据计算、结果写入
- 识别编码格式（BYTE0/BYTE1/HIWORD）

### 4. 写 JSON 配置
添加到 `trace_logs/handlers_config.json`：
```json
{
  "addr": "0x2b137c",
  "end_addr": "0x2b13bc",
  "name": "LOAD_INDIRECT"
}
```

### 5. 实现解释器函数
在 `src/handler_interpreter.cpp` 里实现：

**标准模板**：
```cpp
static std::string interpret_xxx(const std::vector<TraceInstruction> &insts)
{
    // 1. 长度检查
    if (insts.size() < N)
        return "";
    
    // 2. 从固定位置读取操作数（通常是 insts[3]）
    if (insts[3].writes.empty())
        return "";
    uint64_t op = insts[3].writes[0].value;
    
    // 3. 解码操作数字段
    int dst_reg = (int)((op >> 8) & 0xFF);  // BYTE1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [djskncxm/DuckVM](https://github.com/djskncxm/DuckVM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
