---
trigger: always_on
description: Cursor rules for Hashgraph Online development with TypeScript, building AI agents on Hedera with RegistryBrokerClient.
---

# Hashgraph Online (HOL) Development Rules

You are an expert TypeScript developer building applications with Hashgraph Online (HOL) - the open-source SDK for AI agents and decentralized applications on Hedera.

## Technology Stack

**Core:**
- Language: TypeScript (strict mode)
- Runtime: Node.js 20+
- Package Manager: pnpm
- Testing: Jest with @swc/jest

**HOL SDK:**
- @hashgraphonline/standards-sdk - Core SDK for HCS standards and Registry Broker
- @hol-org/hashnet-mcp - MCP server for AI agent integration

**Frontend (when applicable):**
- Framework: Next.js 14+ (App Router)
- UI: shadcn/ui + Tailwind CSS
- Icons: react-icons (Lucide preferred)

## Coding Standards

### TypeScript Requirements
- NEVER use `any` - define proper interfaces
- NEVER use `as any` casting - use type guards
- ALWAYS define explicit return types
- ALWAYS use generics for flexible code
- Validate external data with type guards or zod

### File Naming
- Use kebab-case: `registry-client.ts`, `topic-manager.ts`
- Test files: `__tests__/registry-client.test.ts`
- Components: `registry-browser.tsx`

### Code Style
- Max 500 lines per file - split larger files
- No nested ternaries
- No inline comments - use JSDoc only
- No console.log - use Logger from standards-sdk
- Prettier formatting required

### React Patterns (when applicable)
- NO render functions like `renderContent()`
- NO inline callbacks in JSX
- NO hooks in loops/conditionals
- ALWAYS use separate child components
- ALWAYS define Props interfaces

## HOL SDK Usage

### RegistryBrokerClient - Initialization
```typescript
import { RegistryBrokerClient } from '@hashgraphonline/standards-sdk';

const client = new RegistryBrokerClient({
  baseUrl: 'https://api.hol.org',
  apiKey: process.env.HOL_API_KEY,
});
```

### RegistryBrokerClient - Search Agents
```typescript
import { RegistryBrokerClient, SearchParams, Logger } from '@hashgraphonline/standards-sdk';

const logger = new Logger({ module: 'AgentSearch', level: 'info' });
const client = new RegistryBrokerClient();

const searchParams: SearchParams = {
  q: 'weather',
  registry: 'hcs-10',
  limit: 10,
  page: 1,
};

const results = await client.search(searchParams);
logger.info('Search completed', { total: results.total });
results.hits.forEach(agent => {
  logger.debug('Agent found', { name: agent.name, description: agent.description });
});
```

### RegistryBrokerClient - Resolve Agent (UAID)
```typescript
import { RegistryBrokerClient, Logger } from '@hashgraphonline/standards-sdk';

const client = new RegistryBrokerClient();
const logger = new Logger({ module: 'AgentResolver', level: 'info' });

const agent = await client.resolveUaid('hcs10://0.0.123456/agent-name');
logger.info('Agent resolved', { 
  name: agent.name, 
  protocols: agent.protocols, 
  capabilities: agent.capabilities 
});
```

### RegistryBrokerClient - Register Agent
```typescript
import { RegistryBrokerClient, AgentRegistrationRequest, Logger } from '@hashgraphonline/standards-sdk';

const client = new RegistryBrokerClient();
const logger = new Logger({ module: 'AgentRegistration', level: 'info' });

const registration: AgentRegistrationRequest = {
  name: 'My Agent',
  description: 'A helpful AI agent',
  protocols: ['hcs-10'],
  capabilities: ['chat', 'search'],
  endpoint: 'https://my-agent.example.com',
};

const response = await client.registerAgent(registration, {
  autoTopUp: {
    accountId: process.env.HEDERA_OPERATOR_ID!,
    privateKey: process.env.HEDERA_OPERATOR_KEY!,
  },
});

if (response.success) {
  logger.info('Agent registered', { uaid: response.uaid });
}
```

### RegistryBrokerClient - Chat with Agent
```typescript
import { RegistryBrokerClient, StartChatOptions, Logger } from '@hashgraphonline/standards-sdk';

const client = new RegistryBrokerClient();
const logger = new Logger({ module: 'AgentChat', level: 'info' });

const options: StartChatOptions = {
  uaid: 'hcs10://0.0.123456/agent-name',
  auth: {
    accountId: process.env.HEDERA_OPERATOR_ID!,
    privateKey: process.env.HEDERA_OPERATOR_KEY!,
  },
};

const conversation = await client.chat.start(options);

const response = await conversation.send({
  plaintext: 'Hello, can you help me?',
});

logger.info('Agent response received', { sessionId: conversation.sessionId });
```

### RegistryBrokerClient - Get Stats
```typescript
import { RegistryBrokerClient, Logger } from '@hashgraphonline/standards-sdk';

const client = new RegistryBrokerClient();
const logger = new Logger({ module: 'RegistryStats', level: 'info' });

const stats = await client.stats();
logger.info('Registry stats', { totalAgents: stats.totalAgents, protocols: stats.protocols });

const registries = await client.registries();
registries.forEach(r => logger.debug('Registry', { name: r.name }));
```

### RegistryBrokerClient - Vector Search
```typescript
import { RegistryBrokerClient, VectorSearchRequest, Logger } from '@hashgraphonline/standards-sdk';

const client = new RegistryBrokerClient();
const logger = new Logger({ module: 'VectorSearch', level: 'info' });

const request: VectorSearchRequest = {
  query: 'find me an agent that can help with weather forecasts',
  limit: 5,
  filter: {
    registry: 'hcs-10',
    protocols: ['a2a'],

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
