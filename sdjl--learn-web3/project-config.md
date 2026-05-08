---
trigger: always_on
description: 本文档介绍 `lib` 目录下的所有公共库文件和函数，包括 ABI 工具、配置文件、合约封装和第三方服务。
---

# 公共库文件文档

本文档介绍 `lib` 目录下的所有公共库文件和函数，包括 ABI 工具、配置文件、合约封装和第三方服务。

## 目录结构

```
lib/
├── abi/             # ABI 工具库
│   ├── types.ts          # ABI 相关类型定义
│   ├── utils.ts          # ABI 工具函数（使用 viem）
│   ├── parser.ts         # 事件日志解析器（使用 viem）
├── config/          # 配置文件
│   ├── addresses.ts      # 合约地址配置
│   ├── chains.ts         # 链配置
│   ├── etherscan.ts      # Etherscan API 配置
├── services/        # 第三方服务封装
│   └── etherscan.ts      # Etherscan API 服务
└── utils.ts         # 通用工具函数（Tailwind CSS）
```

---

## ABI 工具库 (lib/abi/)

ABI 工具库提供了处理智能合约 ABI（Application Binary Interface）的工具函数。
本模块使用 viem 库提供的函数来实现核心功能，避免重复实现。

### types.ts

**文件路径**: `lib/abi/types.ts`

**作用**: 
- 定义 ABI 相关的 TypeScript 类型
- 定义事件解析相关的类型

**导出类型**:

```typescript
/**
 * 事件 ABI 输入参数定义
 */
export interface EventAbiInput {
  indexed: boolean;  // 是否为 indexed 参数
  name: string;      // 参数名称，如 "from"、"to"、"value"
  type: string;      // 参数类型，如 "address"、"uint256"
}

/**
 * 事件 ABI 定义
 */
export interface EventAbiItem {
  anonymous: boolean;           // 是否为匿名事件
  inputs: readonly EventAbiInput[];  // 参数列表
  name: string;                 // 事件名称
  type: "event";                // 固定为 "event"
}

/**
 * 原始事件日志数据（来自 Etherscan API）
 */
export interface RawEventLog {
  topics: string[];      // topics[0] 是事件签名，后续是 indexed 参数
  data: string;          // 非 indexed 参数的 ABI 编码数据
  blockNumber: string;
  timeStamp: string;
  logIndex: string;
  transactionHash: string;
}

/**
 * 解析后的事件数据
 */
export interface ParsedEvent {
  eventName: string;
  transactionHash: string;
  blockNumber: string;
  timeStamp: string;
  logIndex: string;
  params: Record<string, string>;  // 参数名 -> 参数值
}
```

---

### utils.ts

**文件路径**: `lib/abi/utils.ts`

**作用**: 
- 提供事件签名计算（使用 viem 的 toEventSelector）
- 构建事件名称和 topic0 的映射关系
- 查找事件 ABI 定义

**导出函数**:

#### getEventTopic0

```typescript
/**
 * 获取事件的 topic0（事件选择器）
 * 使用 viem 的 toEventSelector 计算事件签名的 keccak256 哈希
 * 
 * @param eventAbi - 事件的 ABI 定义
 * @returns 事件签名的 keccak256 哈希值（十六进制字符串）
 */
export function getEventTopic0(eventAbi: EventAbiItem): string;
```

**使用示例**:

```typescript
import { getEventTopic0 } from "@/lib/abi/utils";

const transferAbi = {
  name: "Transfer",
  type: "event" as const,
  anonymous: false,
  inputs: [
    { indexed: true, name: "from", type: "address" },
    { indexed: true, name: "to", type: "address" },
    { indexed: false, name: "value", type: "uint256" },
  ],
};

const topic0 = getEventTopic0(transferAbi);
// 返回: "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef"
```

#### buildEventTopicsMap

```typescript
/**
 * 构建事件名称到 topic0 的映射表
 * 
 * @param eventAbiList - 事件 ABI 数组
 * @returns 事件名称到 topic0 的映射对象
 */
export function buildEventTopicsMap(
  eventAbiList: readonly EventAbiItem[]
): Record<string, string>;
```

#### buildTopicToEventNameMap

```typescript
/**
 * 构建 topic0 到事件名称的反向映射表
 * 用于根据 topic0 反查事件名称
 * 
 * @param eventAbiList - 事件 ABI 数组
 * @returns topic0 到事件名称的映射对象
 */
export function buildTopicToEventNameMap(
  eventAbiList: readonly EventAbiItem[]
): Record<string, string>;
```

#### getEventNames

```typescript
/**
 * 从事件 ABI 数组中提取所有事件名称
 * 
 * @param eventAbiList - 事件 ABI 数组
 * @returns 事件名称数组
 */
export function getEventNames(eventAbiList: readonly EventAbiItem[]): string[];
```

#### findEventAbi

```typescript
/**
 * 根据事件名称查找对应的 ABI 定义
 * 
 * @param eventAbiList - 事件 ABI 数组
 * @param eventName - 要查找的事件名称
 * @returns 找到的事件 ABI，如果未找到则返回 undefined
 */
export function findEventAbi(
  eventAbiList: readonly EventAbiItem[],
  eventName: string
): EventAbiItem | undefined;
```

---

### parser.ts

**文件路径**: `lib/abi/parser.ts`

**作用**: 
- 使用 viem 的 decodeEventLog 解析事件日志
- 将原始事件数据转换为可读格式

**导出函数**:

#### parseEventLog

```typescript
/**
 * 使用 viem 解析单个事件日志
 * 
 * @param log - 原始事件日志数据
 * @param eventAbiList - 合约的所有事件 ABI 列表
 * @param topicToEventName - topic0 到事件名称的映射表
 * @returns 解析后的事件对象，如果事件无法识别则返回 null
 */
export function parseEventLog(
  log: RawEventLog,
  eventAbiList: readonly EventAbiItem[],
  topicToEventName: Record<string, string>
): ParsedEvent | null;
```

#### parseEventLogs

```typescript
/**
 * 批量解析事件日志
 * 自动过滤掉无法识别的事件，可选按时间戳排序
 * 
 * @param logs - 原始事件日志数组
 * @param eventAbiList - 合约的所有事件 ABI 列表
 * @param sortDesc - 是否按时间戳倒序排列，默认 true
 * @returns 解析后的事件数组
 */
export function parseEventLogs(
  logs: RawEventLog[],
  eventAbiList: readonly EventAbiItem[],
  sortDesc?: boolean
): ParsedEvent[];
```

**使用示例**:

```typescript
import { parseEventLogs } from "@/lib/abi/parser";
import { USDT_EVENT_ABI } from "@/lib/abi/usdt";

// 解析事件日志
const events = parseEventLogs(response.result, USDT_EVENT_ABI, true);

for (const event of events) {
  console.log(`${event.eventName}: from=${event.params.from}, to=${event.params.to}`);
}
```

---

## Config 配置文件 (lib/config/)

### tokens.ts

**文件路径**: `lib/config/tokens.ts`

**作用**: 
- 集中管理所有代币名称常量
- 提供类型安全的代币名称引用
- 避免代币名称拼写错误
- 方便 TypeScript 类型检查

**导出内容**:

```typescript
/**
 * 代币符号常量
 * 使用 as const 确保类型安全
 */
export const TOKENS = {
  USDT: "USDT",
  ETH: "ETH",
} as const;

/**
 * 代币符号类型
 */
export type TokenSymbol = (typeof TOKENS)[keyof typeof TOKENS];

/**
 * 代币信息接口
 */
export interface TokenInfo {
  symbol: TokenSymbol;
  name: string;
  decimals: number;
}

/**
 * 代币信息配置
 */
export const TOKEN_INFO = {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sdjl/learn-web3](https://github.com/sdjl/learn-web3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
