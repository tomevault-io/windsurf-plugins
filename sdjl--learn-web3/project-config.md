---
trigger: always_on
description: 本文档介绍 `components` 目录下的所有公共组件，包括组件的作用、接口参数和使用示例。
---

# 公共组件文档

本文档介绍 `components` 目录下的所有公共组件，包括组件的作用、接口参数和使用示例。

## 目录结构

```
components/
├── layout/          # 布局相关组件
│   ├── Header.tsx
├── wallet/          # 钱包相关组件
│   ├── WalletConnection.tsx
│   └── WalletNotConnected.tsx
└── transaction/     # 交易相关组件
    ├── TransactionItem.tsx
    └── TransactionList.tsx
```

---

## Layout 组件

### Header

**文件路径**: `components/layout/Header.tsx`

**作用**: 
- 展示页面的主标题和副标题
- 提供页面功能的简要说明
- 统一的页面头部样式

**接口参数**:

```typescript
interface HeaderProps {
  /** 页面标签文本（显示在标题上方的小标签） */
  label: string;
  /** 主标题文本 */
  title: string;
  /** 页面描述文本 */
  description: string;
}
```

**使用示例**:

```typescript
import { Header } from "@/components/layout/Header";

<Header
  label="合约查询"
  title="查询合约信息"
  description="输入合约地址，查看合约的基本信息、余额和源代码。"
/>
```

---

## Wallet 组件

### WalletConnection

**文件路径**: `components/wallet/WalletConnection.tsx`

**作用**: 
- 显示 RainbowKit 的 ConnectButton，用于连接/断开钱包
- 展示当前连接状态、钱包地址、网络信息和余额
- 支持自定义标题和描述
- 支持指定必需的网络（如 Sepolia），并显示网络警告
- 自动处理钱包连接逻辑（浏览器插件和手机钱包）

**接口参数**:

```typescript
interface WalletConnectionProps {
  /** 标题文本（默认: "连接你的钱包"） */
  title?: string;
  /** 描述文本（默认: "RainbowKit 自带钱包列表、主题与 WC v2 支持。"） */
  description?: string;
  /** 是否显示完整信息（连接状态、地址、网络、余额）（默认: true） */
  showFullInfo?: boolean;
  /** 是否只在已连接时显示信息（默认: false） */
  showInfoOnlyWhenConnected?: boolean;
  /** 必需的网络（如果指定，会检查当前网络是否匹配） */
  requiredChain?: Chain;
}
```

**使用示例**:

```typescript
import { WalletConnection } from "@/components/wallet/WalletConnection";
import { sepolia } from "wagmi/chains";

// 基础用法
<WalletConnection />

// 自定义标题和描述
<WalletConnection
  title="连接钱包"
  description="请先连接钱包"
/>

// 简化模式（只显示网络和余额）
<WalletConnection
  showFullInfo={false}
  showInfoOnlyWhenConnected={true}
/>

// 指定必需的网络
<WalletConnection
  requiredChain={sepolia}
/>
```

**功能说明**:
- `ConnectButton` 会自动处理钱包连接：
  - 浏览器插件（如 MetaMask）→ 直接调用 `window.ethereum` 连接
  - 手机钱包（如 MetaMask Mobile）→ 生成二维码，通过 WalletConnect Cloud 中继
- 如果指定了 `requiredChain`，会检查当前网络是否匹配，不匹配时显示警告

---

### WalletNotConnected

**文件路径**: `components/wallet/WalletNotConnected.tsx`

**作用**: 
- 当用户未连接钱包时显示提示信息
- 引导用户连接钱包以使用相关功能

**接口参数**:

```typescript
interface WalletNotConnectedProps {
  /** 提示消息文本（默认: "请先连接钱包"） */
  message?: string;
}
```

**使用示例**:

```typescript
import { WalletNotConnected } from "@/components/wallet/WalletNotConnected";

// 使用默认消息
<WalletNotConnected />

// 自定义消息
<WalletNotConnected message="请先连接钱包以使用合约查询功能" />
```

---

## Transaction 组件

### TransactionItem

**文件路径**: `components/transaction/TransactionItem.tsx`

**作用**: 
- 显示单条交易记录的详细信息
- 支持不同的货币单位和显示模式
- 可配置的区块浏览器链接
- 支持显示发送/接收状态

**接口参数**:

```typescript
export interface TransactionItemProps {
  /** 交易记录数据 */
  transaction: Transaction;
  /** 格式化后的交易金额 */
  formattedValue: string;
  /** 货币符号（如 ETH、USDT） */
  currencySymbol: string;
  /** 格式化后的交易费用 */
  formattedFee: string;
  /** 费用货币符号（通常是 ETH，默认: "ETH"） */
  feeCurrencySymbol?: string;
  /** 区块浏览器基础 URL */
  blockExplorerUrl: string;
  /** 是否显示发送/接收状态（用于用户自己的交易，默认: false） */
  showDirection?: boolean;
  /** 用户地址（用于判断发送/接收） */
  userAddress?: string;
  /** 交易类型标签（如 "USDT 转账"） */
  typeLabel?: string;
  /** 是否显示发送地址和接收地址（默认根据 showDirection 决定，默认: false） */
  showBothAddresses?: boolean;
  /** 是否显示秒（时间格式，默认: false） */
  showSeconds?: boolean;
}
```

**使用示例**:

```typescript
import { TransactionItem } from "@/components/transaction/TransactionItem";
import type { Transaction } from "@/lib/services/etherscan";

const transaction: Transaction = { /* ... */ };

// 基础用法
<TransactionItem
  transaction={transaction}
  formattedValue="1.5"
  currencySymbol="ETH"
  formattedFee="0.001"
  blockExplorerUrl="https://etherscan.io"
/>

// 显示发送/接收状态
<TransactionItem
  transaction={transaction}
  formattedValue="100"
  currencySymbol="USDT"
  formattedFee="0.001"
  blockExplorerUrl="https://etherscan.io"
  showDirection={true}
  userAddress="0x..."
  typeLabel="USDT 转账"
/>

// 显示完整地址信息
<TransactionItem
  transaction={transaction}
  formattedValue="1.5"
  currencySymbol="ETH"
  formattedFee="0.001"
  blockExplorerUrl="https://etherscan.io"
  showBothAddresses={true}
  showSeconds={true}
/>
```

**显示内容**:
- 交易哈希（可点击跳转到区块浏览器）
- 交易金额（支持显示发送/接收状态）
- 发送地址和接收地址（可点击跳转）
- 交易费用
- 交易时间
- 区块号（可点击跳转）
- 交易状态（成功/失败）
- 交易类型标签（可选）

---

### TransactionList

**文件路径**: `components/transaction/TransactionList.tsx`

**作用**: 
- 提供统一的交易列表容器
- 处理加载、错误、空状态
- 支持刷新功能
- 自动去重交易记录

**接口参数**:

```typescript
export interface TransactionListProps {
  /** 交易记录列表 */
  transactions: Transaction[];
  /** 标题 */
  title: string;
  /** 加载状态（默认: false） */
  isLoading?: boolean;
  /** 错误信息 */
  error?: Error | null;
  /** 加载中的提示文本（默认: "加载交易数据中..."） */
  loadingText?: string;
  /** 空状态的提示文本（默认: "暂无交易记录"） */
  emptyText?: string;
  /** 错误状态的提示文本 */
  errorText?: string;
  /** 刷新函数 */
  onRefresh?: () => void;
  /** 格式化交易金额的函数 */
  formatValue: (tx: Transaction) => string;
  /** 货币符号 */
  currencySymbol: string;
  /** 格式化交易费用的函数 */
  formatFee: (tx: Transaction) => string;
  /** 费用货币符号（默认: "ETH"） */
  feeCurrencySymbol?: string;
  /** 区块浏览器基础 URL */
  blockExplorerUrl: string;
  /** 传递给 TransactionItem 的其他属性 */
  itemProps?: Omit<
    TransactionItemProps,
    | "transaction"
    | "formattedValue"
    | "currencySymbol"
    | "formattedFee"
    | "feeCurrencySymbol"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sdjl/learn-web3](https://github.com/sdjl/learn-web3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
