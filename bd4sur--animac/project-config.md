---
trigger: always_on
description: 以下是Animac项目的通用研发规范。该规范尚未编写完毕，仍在持续补充，供AI编码代理参考。
---

# Animac 2026 通用研发规范

以下是Animac项目的通用研发规范。该规范尚未编写完毕，仍在持续补充，供AI编码代理参考。

## 项目概述

Animac（灵机）是一款Scheme解释器，支持Scheme语言的子集和某些自定义特性，并不完全遵守R5RS。

## 目录结构

- doc/：文档
- include/：C语言头文件
- src/：C语言实现（代码）文件
- test/：C语言测试实现文件
- typescript/：本项目早期原型项目，由TypeScript实现。除非明确要求，否则应忽略该目录，不要主动阅读 typescript 目录下的内容，以免干扰上下文。
- amalgamate.sh：Amalgamation 脚本，将解释器核心（伞形头文件收录范围，不含宿主相关文件）合并为单文件分发形态（见“架构设计”）。
- animac_core.h / animac_core.c：由 amalgamate.sh 生成的单文件头文件/实现（勿手工编辑）。

## 术语约定

- 逻辑长度称length，物理长度称size，容器最大容量称capacity。
- parameter形式参数称“引数”，argument实际参数称“参数”。
- Alpha-renaming过程，也就是通过换名来消除嵌套词法作用域中同名变量的混淆的过程，简称为ARN。
- 内置函数和运算符，值分类上属于变量，统称为“内建变量”builtin，不叫“primitive”。

## 环境与工具

- 禁止执行任何删除命令，如`rm`。
- 禁止通过Python、Node.js、bash等脚本间接执行具有删除副作用的函数或命令，例如`unlink`等。
- 禁止执行对Git状态产生修改的命令，如`git add`、`git push`、`git pull`、`git stash`等。但可以查看Git状态。如果确有需要，则必须停止对话，立刻向我请示如何处理。
- 当前开发环境是Windows系统，但部署了WSL（Ubuntu）。你可以使用WSL，通过WSL使用make、gcc等构建工具进行构建、测试。

## 架构设计

公开 API 伞形头文件（仿 CPython 的 Python.h）：`include/animac.h` 汇总解释器核心（基础设施、前端、运行时）的全部公开头文件。上层程序（REPL、解释器入口、debugger 等）只应直接包含 `animac.h`，另按需包含被明确排除的 host.h（宿主适配）、native_*.h（native 库）、highlight.h（终端呈现）。新增解释器核心头文件时，必须登记进 animac.h 对应分组。


关于解释器的工作目录、模块全局ID：

- 任何解释器实例都必须指定一个基准工作目录(base_dir)。
- 若工作在REPL模式下，则以终端cwd为base_dir，给模块一个临时文件名
- 若运行代码文件，则以该文件所在目录为base_dir，即该文件绝对路径的目录部分（约定不带斜杠）。
- 所有的import文件路径，要么是绝对路径，要么是相对于base_dir的相对路径。
- 链接器搜索import模块的算法：
  - 判断import文件路径是绝对路径还是相对路径。如果是绝对路径，直接读取。
  - 如果是相对路径，则将base_dir与相对路径拼接成绝对路径再读取。
- 模块ID的构造规则：
  - 将模块绝对路径中的斜杠替换为点、空白字符替换成下划线、冒号去掉。
  - 去掉第一个点；若文件名有.scm后缀则去掉
  - 例如："/home/a/b.scm" -> home.a.b
- 链接器（src/linker.c）不直接依赖宿主API获取模块源码（依赖倒置）：调用方通过 `am_link` 的引数注入 `am_linker_read_source_fn` 回调（见 include/linker.h），回调用传入的 allocator 分配返回的源码缓冲区（由链接器用 am_free 释放）。宿主侧的文件系统默认实现为 `am_host_read_source_from_file`（include/host.h，src/host.c 与 src/host_esp32.cpp 各有一份实现）。
- 垃圾回收（GC）统一由 gc 模块实现（include/gc.h、src/gc.c），层级位于 process/heap 之上、runtime 之下（不依赖 runtime.h）：对外函数——`am_gc_process`（分进程标记-清除）、`am_gc_compact`（全局标记-压缩：收集各 heap 表存活对象、调用压缩引擎、回写 heap 表指针）、`am_gc_collect`（对进程池的一轮编排，含 force_compact 参数，由 runtime 调用）、`am_gc_heap_watermark_level`（堆水位查询：0 正常 / 1 高水位 / 2 临界水位）。GC 配置宏集中在 gc.h：`AM_ENABLE_GC`、`AM_HEAP_COMPACT_INTERVAL`、`AM_GC_HEAP_HIGH_WATER_RATIO`(0.75)、`AM_GC_HEAP_CRITICAL_RATIO`(0.90)、`AM_GC_HEAP_FRAG_FLOOR_RATIO`(0.30)、`AM_GC_HEAP_FRAG_MIN_BLOCK_RATIO`(0.03125)、`AM_GC_PERIODIC_INTERVAL`(32)、`AM_GC_WATERMARK_CHECK_STRIDE`(256)。压缩职责按“引擎+钩子”拆分：allocator 提供纯物理压缩引擎 `am_allocator_heap_compact`（不感知逻辑堆，经 `am_allocator_relocate_fn` 回调按地址升序报告重定位）及 `am_allocator_host_malloc/realloc/free` 暂存分配接口；逻辑堆知识（存活判定、handle 表回写）全部在 gc 模块。allocator 不依赖 heap/map/object 等上层模块。
- GC 触发策略为“堆水位为主、周期兜底为辅”的三级触发（2026-07 起，取代原先的每轮事件循环定时 GC）：
  - L0（allocator 层）：`freelist_malloc` 分配失败时先经 `am_allocator_pool_auto_adjust` 向 VM 区让渡边界并重试（≤4 次），彻底失败置 `oom_flag`（经 `am_allocator_heap_take_oom_flag` 读取清除）；allocator 层级不允许触发 GC。
  - L1（runtime 层）：`am_runtime_tick` 内每 `AM_GC_WATERMARK_CHECK_STRIDE` 条指令及 tick 末尾检查水位（`runtime_gc_watermark_check`），级别 1 做标记-清除、级别 2 强制压缩；发现 oom_flag 也强制一轮 GC 以挽救其余进程。水位兼顺两个维度：用量比（used/capacity）与碎片（用量≥30% 且最大空闲块低于 max(容量×1/32, 近期最大分配请求)，防止 first-fit 碎片化失败；largest_request 由 freelist 记账、压缩后清零；碎片维度两阶段查询，低于下限不遍历空闲链表）。
  - L2（事件循环层）：每 `AM_GC_PERIODIC_INTERVAL` 轮事件循环执行一轮兜底 GC（0 表示禁用，即纯水位）。
  - allocator 新增查询：`am_allocator_heap_usage`（used/capacity/largest_free_block/largest_request）、`am_allocator_heap_take_oom_flag`。
- 内存池（src/allocator.c）不直接依赖宿主系统的 malloc/calloc/realloc/free（依赖倒置）：宿主在调用 `am_allocator_pool_create` 时通过 `am_allocator_host_vtable_t` 虚函数表（见 include/allocator.h）注入 `host_malloc`/`host_calloc`/`host_realloc`/`host_free` 四个实现（均为必需，任一为 NULL 则 create 失败）。池控制块、池底层内存及堆压缩的临时工作数组均经由该表分配；池仅保存指针不拷贝，宿主须保证其生命周期不短于池。宿主侧参考实现为 include/host.h 的 `am_host_malloc` 等四函数及默认实例 `am_host_default_vtable`（src/host.c 与 src/host_esp32.cpp 各有一份，ESP32 版映射到 SPIRAM 的 heap_caps_* 系列）。
- 运行时（src/runtime.c）不直接依赖宿主的输入输出回调、定时器与时间戳函数（依赖倒置）：宿主在调用 `am_runtime_create` 时通过 `am_runtime_vtable_t` 虚函数表（见 include/runtime.h）注入 6 个实现——事件回调 `on_tick`/`on_event`/`on_halt`/`on_error`（可选，为 NULL 则不触发）与时间函数 `now_ms`/`sleep_in_ms`（必需，为 NULL 则 create 失败）。runtime 仅保存 vtable 指针不拷贝，宿主须保证其生命周期不短于 runtime。桌面宿主的默认实现见 main.c 与 src/repl.c 中的 `g_host_vtable`（时间函数底层为 include/host.h 的 `am_current_timestamp_in_ms`/`am_sleep_in_ms`，src/host.c 与 src/host_esp32.cpp 各有一份实现）。

symbol是以其字面为ID的，相同拼写的symbol，无论在哪个上下文中都是同一个符号。因此AST合并时，字符串相同的symbol，就是同一个symbol。这与variable截然不同。

- 栈平衡（单值栈纪律，2026-07 起，方案详见 doc/todo/stack_bal.md 之“方案A：编译期单值纪律”）：编译器保证**任何表达式求值结束后恰好在 opstack 上留下 1 个值，任何语句序列执行完后栈深恢复到序列开始时的水平**。实现要点（均在 src/am_compiler.c，VM 指令语义不变）：
  - `begin`/lambda 体/`while` 体中，非末尾表达式的结果立即 `pop`；空 `begin`/空 lambda 体补 `push #undefined`；
  - 语句形式（`define`/`set!`/`display`/`newline`/`push`/`set_item!`/`while`/单臂 if 的假分支/无 else 且全部落空的 `cond`/空 `begin`）的值统一为 `#undefined`；
  - 语句型内建清单集中在 `compiler_statement_builtin_residue`（display/newline/list_push/set_item/read/write/fork），编译器在指令后补偿弹残留并 `push #undefined`（与 `AM_BUILTIN_OPCODE_MAP` 并列维护，勿散落各点）；
  - 编译期“零效应形式”（import/native/define-syntax/let-syntax/letrec-syntax/syntax-rules 与裸符号 break/continue）不产生值，由 `compiler_check_produces_value` 判定，语句边界不为它们发射 pop；
  - 纪律的连锁性质：函数调用净效应恒为 -argc+1；尾递归在 opstack 上恒界；while 无逐迭代栈累积；静态栈深分析随之精确（`compiler_stack_effect` 按运行时实际效应维护，call 后继深度 = 调用点深度 - argc + 1）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bd4sur/Animac](https://github.com/bd4sur/Animac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
