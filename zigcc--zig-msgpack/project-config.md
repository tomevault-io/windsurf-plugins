---
trigger: always_on
description: 本文档为 AI 助手提供 zig-msgpack 项目的结构化指南，便于理解代码库并进行开发协助。
---

# AGENT.md - LLM 代码阅读规范

本文档为 AI 助手提供 zig-msgpack 项目的结构化指南，便于理解代码库并进行开发协助。

## 文档目的

- 为 LLM 提供项目快速索引
- 规范代码理解和修改流程
- 定义关键概念和术语
- 说明代码规约和最佳实践

---

## 1. 项目核心概念

### 1.1 项目定位
- **类型**：Zig 语言的 MessagePack 序列化/反序列化库
- **规范**：完整实现 MessagePack specification (https://msgpack.org)
- **特性**：支持所有 MessagePack 类型，包括 timestamp 扩展类型 (-1)

### 1.2 MessagePack 类型系统映射

| MessagePack 类型 | Zig 实现 | 说明 |
|-----------------|---------|------|
| nil | `Payload.nil: void` | 空值 |
| bool | `Payload.bool: bool` | 布尔值 |
| int | `Payload.int: i64` | 有符号整数 |
| uint | `Payload.uint: u64` | 无符号整数 |
| float | `Payload.float: f64` | 浮点数 |
| str | `Payload.str: Str` | UTF-8 字符串 |
| bin | `Payload.bin: Bin` | 二进制数据 |
| array | `Payload.arr: []Payload` | 数组 |
| map | `Payload.map: Map` | 键值对 |
| ext | `Payload.ext: EXT` | 扩展类型 |
| timestamp | `Payload.timestamp: Timestamp` | 时间戳（ext type -1） |

---

## 2. 文件结构

```
src/
├── msgpack.zig      # 核心实现（主文件）
├── test.zig         # 完整测试套件
├── bench.zig        # 性能基准测试
└── compat.zig       # 跨版本兼容层
```

### 2.1 各文件职责

#### `src/msgpack.zig` (核心实现)
- 定义 `Payload` 联合类型
- 实现 `Pack()` 泛型序列化器
- 提供包装类型：`Str`, `Bin`, `EXT`, `Timestamp`
- 导出工具函数：`wrapStr()`, `wrapBin()`, `wrapEXT()`
- 导出常量结构体：`FixLimits`, `IntBounds`, `FixExtLen`, `TimestampExt`, `MarkerBase`

#### `src/compat.zig` (兼容层)
- 提供 `BufferStream` 跨版本实现
- 处理 Zig 0.14-0.16 API 差异
- 导出 `fixedBufferStream` 兼容函数

#### `src/test.zig` (测试套件)
- 覆盖所有 MessagePack 类型
- 测试边界条件和错误处理
- 验证格式选择逻辑（最小编码原则）
- 包含 Fuzz 测试覆盖随机数据

#### `src/bench.zig` (性能基准测试)
- 基本类型序列化/反序列化性能测试
- 不同大小容器(数组/Map)的性能对比
- 嵌套结构和混合类型的实际场景测试
- 提供详细的吞吐量和延迟指标

---

## 3. 核心 API 规范

### 3.0 常量组织

### 9.4 性能优化指南

1. **使用内联函数**：频繁调用的小函数添加 `inline` 关键字
2. **利用泛型**：避免为每个类型重复相似代码
3. **使用 switch**：比 if-else 链更高效（编译器可优化为跳转表）
4. **减少分支**：简化控制流，提升分支预测准确性
5. **复用辅助函数**：如 `writeIntRaw`, `readIntRaw`, `writeDataWithLength`
库提供了组织化的常量结构体，方便使用和理解：

```zig
// MessagePack 格式限制
msgpack.FixLimits.POSITIVE_INT_MAX  // 127
msgpack.FixLimits.STR_LEN_MAX       // 31
msgpack.FixLimits.ARRAY_LEN_MAX     // 15
msgpack.FixLimits.MAP_LEN_MAX       // 15

// 整数类型边界
msgpack.IntBounds.UINT8_MAX   // 0xff
msgpack.IntBounds.UINT16_MAX  // 0xffff
msgpack.IntBounds.INT8_MIN    // -128

// 固定扩展类型长度
msgpack.FixExtLen.EXT4   // 4
msgpack.FixExtLen.EXT8   // 8

// Timestamp 相关常量
msgpack.TimestampExt.TYPE_ID        // -1
msgpack.TimestampExt.FORMAT32_LEN   // 4
msgpack.TimestampExt.NANOSECONDS_MAX // 999_999_999
```

### 3.1 Payload 创建方法

#### 基本类型（栈分配，无需 free）
```zig
Payload.nilToPayload() -> Payload
Payload.boolToPayload(val: bool) -> Payload
Payload.intToPayload(val: i64) -> Payload
Payload.uintToPayload(val: u64) -> Payload
Payload.floatToPayload(val: f64) -> Payload
Payload.timestampFromSeconds(seconds: i64) -> Payload
Payload.timestampToPayload(seconds: i64, nanoseconds: u32) -> Payload
```

#### 堆分配类型（需要 `payload.free(allocator)`）
```zig
Payload.strToPayload(val: []const u8, allocator: Allocator) !Payload
Payload.binToPayload(val: []const u8, allocator: Allocator) !Payload
Payload.extToPayload(t: i8, data: []const u8, allocator: Allocator) !Payload
Payload.arrPayload(len: usize, allocator: Allocator) !Payload
Payload.mapPayload(allocator: Allocator) Payload
```

### 3.2 Payload 操作方法

#### 数组操作
```zig
payload.getArrLen() !usize                           // 获取数组长度
payload.getArrElement(index: usize) !Payload         // 获取元素
payload.setArrElement(index: usize, val: Payload) !void  // 设置元素
```

#### Map 操作
```zig
payload.mapGet(key: []const u8) !?Payload           // 获取值（可能为 null）
payload.mapPut(key: []const u8, val: Payload) !void // 插入/更新键值对
```

#### 类型转换
```zig
 // 宽松转换（允许类型转换）
 payload.getInt() !i64   // uint 可转换为 i64（如果在范围内）
 payload.getUint() !u64  // 正数 int 可转换为 u64
 
 // 严格转换（不允许类型转换）
 payload.asInt() !i64     // 只接受 .int 类型
 payload.asUint() !u64    // 只接受 .uint 类型
 payload.asFloat() !f64   // 只接受 .float 类型
 payload.asBool() !bool   // 只接受 .bool 类型
 payload.asStr() ![]const u8   // 只接受 .str 类型
 payload.asBin() ![]u8         // 只接受 .bin 类型
 
 // 类型检查
 payload.isNil() bool       // 检查是否为 nil
 payload.isNumber() bool    // 检查是否为数字（int/uint/float）
 payload.isInteger() bool   // 检查是否为整数（int/uint）
```

### 3.3 序列化/反序列化

#### 创建 Pack 实例
```zig
const pack = msgpack.Pack(
    *BufferStream,          // WriteContext 类型
    *BufferStream,          // ReadContext 类型
    BufferStream.WriteError,
    BufferStream.ReadError,
    BufferStream.write,     // writeFn
    BufferStream.read,      // readFn
);

var p = pack.init(&write_buffer, &read_buffer);
```

#### 基本操作
```zig
try p.write(payload);                    // 序列化
const result = try p.read(allocator);   // 反序列化
defer result.free(allocator);           // 释放内存
```

---

## 4. 编码规范

### 4.1 最小编码原则

序列化器**必须**使用最小格式：

| 值范围 | 格式选择 |
|--------|---------|
| 0-127 | positive fixint (1 byte) |
| -32 to -1 | negative fixint (1 byte) |
| 128-255 | uint8 (2 bytes) |
| 字符串 0-31 字节 | fixstr |
| 数组 0-15 元素 | fixarray |
| Map 0-15 条目 | fixmap |

### 4.2 Timestamp 格式选择

```zig
// timestamp 32: nanoseconds == 0 && seconds 在 [0, 2^32-1]
// 格式: fixext4 + type(-1) + 4 bytes seconds

// timestamp 64: seconds 在 [0, 2^34-1] && nanoseconds <= 999999999
// 格式: fixext8 + type(-1) + 8 bytes (nano<<34 | seconds)

// timestamp 96: 其他情况（负秒数或大秒数）
// 格式: ext8 + len(12) + type(-1) + 4 bytes nano + 8 bytes seconds

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zigcc/zig-msgpack](https://github.com/zigcc/zig-msgpack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
