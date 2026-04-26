---
trigger: always_on
description: This codebase contains Scaffold-ETH 2 (SE-2), everything you need to build dApps on Ethereum. Its tech stack is NextJS, RainbowKit, Wagmi and Typescript. Supports Hardhat and Foundry.
---


This codebase contains Scaffold-ETH 2 (SE-2), everything you need to build dApps on Ethereum. Its tech stack is NextJS, RainbowKit, Wagmi and Typescript. Supports Hardhat and Foundry.

It's a yarn monorepo that contains following packages:

- FOUNDRY (`packages/foundry`): The solidity framework to write, test and deploy EVM Smart Contracts.
- NextJS (`packages/nextjs`): The UI framework extended with utilities to make interacting with Smart Contracts easy (using Next.js App Router, not Pages Router).


The usual dev flow is:

- Start SE-2 locally:
  - `yarn chain`: Starts a local blockchain network
  - `yarn deploy`: Deploys SE-2 default contract
  - `yarn start`: Starts the frontend
- Write a Smart Contract (modify the deployment script in `packages/foundry/script` if needed)
- Deploy it locally (`yarn deploy`)
- Go to the `http://locahost:3000/debug` page to interact with your contract with a nice UI
- Iterate until you get the functionality you want in your contract
- Write tests for the contract in `packages/foundry/test`
- Create your custom UI using all the SE-2 components, hooks, and utilities.
- Deploy your Smart Contrac to a live network
- Deploy your UI (`yarn vercel` or `yarn ipfs`)
  - You can tweak which network the frontend is pointing (and some other configurations) in `scaffold.config.ts`

## Smart Contract UI interactions guidelines
SE-2 provides a set of hooks that facilitates contract interactions from the UI. It reads the contract data from `deployedContracts.ts` and `externalContracts.ts`, located in `packages/nextjs/contracts`.

### Reading data from a contract
Use the `useScaffoldReadContract` (`packages/nextjs/hooks/scaffold-eth/useScaffoldReadContract.ts`) hook. 

Example:
```typescript
const { data: someData } = useScaffoldReadContract({
  contractName: "YourContract",
  functionName: "functionName",
  args: [arg1, arg2], // optional
});
```

### Writing data to a contract
Use the `useScaffoldWriteContract` (`packages/nextjs/hooks/scaffold-eth/useScaffoldWriteContract.ts`) hook.
1. Initilize the hook with just the contract name
2. Call the `writeContractAsync` function.

Example:
```typescript
const { writeContractAsync: writeYourContractAsync } = useScaffoldWriteContract(
  { contractName: "YourContract" }
);
// Usage (this will send a write transaction to the contract)
await writeContractAsync({
  functionName: "functionName",
  args: [arg1, arg2], // optional
  value: parseEther("0.1"), // optional, for payable functions
});
```

Never use any other patterns for contract interaction. The hooks are:
- useScaffoldReadContract (for reading)
- useScaffoldWriteContract (for writing)

### Reading events from a contract

Use the `useScaffoldEventHistory` (`packages/nextjs/hooks/scaffold-eth/useScaffoldEventHistory.ts`) hook.

Example:

```typescript
const {
  data: events,
  isLoading,
  error,
} = useScaffoldEventHistory({
  contractName: "YourContract",
  eventName: "GreetingChange",
  watch: true, // optional, if true, the hook will watch for new events
});
```

The `data` property consists of an array of events and can be displayed as:

```jsx
<div>
  {events?.map((event) => (
    <div key={event.logIndex}>
      <p>{event.args.greetingSetter}</p>
      <p>{event.args.newGreeting}</p>
      <p>{event.args.premium}</p>
      <p>{event.args.value}</p>
    </div>
  ))}
</div>
```

### Other Hooks
SE-2 also provides other hooks to interact with blockchain data: `useScaffoldWatchContractEvent`, `useScaffoldEventHistory`, `useDeployedContractInfo`, `useScaffoldContract`, `useTransactor`. They live under `packages/nextjs/hooks/scaffold-eth`.
## Display Components guidelines
SE-2 provides a set of pre-built React components for common Ethereum use cases: 
- `Address`: Always use this when displaying an ETH address
- `AddressInput`: Always use this when users need to input an ETH address
- `Balance`: Display the ETH/USDC balance of a given address
- `EtherInput`: An extended number input with ETH/USD conversion.

They live under `packages/nextjs/components/scaffold-eth`.

Find the relevant information from the documentation and the codebase. Think step by step before answering the question.

## 📚 核心文档参考规则

### llms-full-scaffold.txt - 必读文档
- **重要性**: `llms-full-scaffold.txt` 是本项目开发中**至关重要**的参考文档
- **使用原则**: 多阅读、勤阅读，在任何 Scaffold-ETH 2 相关的开发问题上优先参考此文档
- **避免幻觉**: 在回答问题前，务必查阅该文档确认准确性，避免编造不存在的 API 或用法
- **深度查询**: 如有必要，使用 DeepWiki MCP 工具查询 Scaffold-ETH 2 官方仓库 (https://github.com/scaffold-eth/scaffold-eth-2) 获取最新信息

示例用法:
```
# 查询 Scaffold-ETH 相关问题
使用 mcp_mcp-deepwiki_deepwiki_fetch 查询:
url: "scaffold-eth/scaffold-eth-2"
```

## 🎭 前端开发与调试规则

### Playwright MCP - 真实环境调试
- **调试策略**: 在进行前端开发和调试时，必须结合 Playwright MCP 进行**真实浏览器环境**的测试
- **质量标准**: 使用浏览器自动化工具进行"拳拳到肉"的调试，直到一切**看起来完美**
- **工作流程**:
  1. 启动开发服务器 (`yarn start`)
  2. 使用 `browser_navigate` 访问页面
  3. 使用 `browser_snapshot` 获取页面结构
  4. 使用 `browser_click`、`browser_type` 等工具模拟用户操作
  5. 使用 `browser_console_messages` 查看前端错误
  6. 使用 `browser_take_screenshot` 验证视觉效果
  7. 迭代修复直到功能完美

### 调试优先级
1. **先看文档** - 查阅 `llms-full-scaffold.txt`
2. **再写代码** - 基于文档编写符合规范的代码
3. **实时测试** - 用 Playwright 在浏览器中验证
4. **持续优化** - 根据实际表现迭代改进


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TreapGoGo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
