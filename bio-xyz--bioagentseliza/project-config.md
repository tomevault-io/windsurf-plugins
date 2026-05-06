---
trigger: always_on
description: Repository: https://github.com/elizaos/eliza
---

# ElizaOS 1.0.0 Development Rules

Repository: https://github.com/elizaos/eliza

## Package structure

packages/core - @elizaos/core - the runtime and types
packages/client - The frontend GUI that is displayed by the CLI running
packages/app - The desktop and mobile application, built in Tauri, wrapping the core GUI and CLI
packages/cli - The CLI which contains the agent runtime and starts up the REST API, GUI, and loads agents and projects
-> This is what runs in most 'bun run test' and 'bun run start' cases, etc
-> 'elizaos' command is from here
packages/plugin-bootstrap - Default event handlers, actions and providers
packages/plugin-sql - DatabaseAdapter for Postgres and PGLite, soon others

There are others but they are not as important

## Core Development Principles

### 1. Flow - Always Plan First

- **Bug Fixes**: First identify the bug, research ALL related files, create complete change plan
- **Impact Analysis**: Identify all possible errors and negative outcomes from changes
- **Documentation**: Create thorough PRD and implementation plan BEFORE writing any code
- **Identify risks and approaches**: Thoroughly outline all risks and offer multiple possible approaches, choosing your favorite
- **Just do it**: Once the plan is in place, start writing code. Don't wait for response from the user.

### 2. No Stubs or Incomplete Code

- **Never** use stubs, fake code, or incomplete implementations
- **Always** continue writing until all stubs are replaced with finished, working code
- **No POCs**: Never deliver proof-of-concepts - only finished, detailed code
- **Iteration**: Work on files until they are perfect, looping testing and fixing until all tests pass

### 3. Test-Driven Development

- Models hallucinate frequently - thorough testing is critical
- Verify tests are complete and passing before declaring changes correct
- First attempts are usually incorrect - test thoroughly
- Write tests before implementation when possible

## Testing Infrastructure

### Command Structure

- **Main Command**: `elizaos test` (run from packages/cli)
- **Test Framework**: vitest
- **Subcommands**:
  - `component`: Run component tests using Vitest
  - `e2e`: Run end-to-end runtime tests
  - `all`: Run both component and e2e tests (default)

### Test Types

- **E2E Tests**:
  - Use actual runtime
  - Cannot use vitest state (interferes with internal elizaos vitest instance)
  - Test real integrations and workflows
- **Unit Tests**:
  - Use vitest with standard primitives
  - Test individual components in isolation

## Architecture Details

### Core Dependencies

- **Central Dependency**: Everything depends on @elizaos/core or packages/core
- **No Circular Dependencies**: Core cannot depend on other packages
- **Import Pattern**: Use @elizaos/core in package code, packages/core in internal references

### Key Files

- **Types**: `packages/core/src/types.ts` - All core type definitions
- **Runtime**: `packages/core/src/runtime.ts` - Main runtime implementation
- **Plugin Compatibility**: Shim everything through /specs (currently defaulting to v2)

### Abstraction Layers

- **Channel → Room Mapping**:
  - Discord/Twitter/GUI channels become "rooms"
  - All IDs swizzled with agent's UUID into deterministic UUIDs
  - Maintains consistency across platforms
- **Server → World Mapping**:
  - Servers become "worlds" in agent memory
  - Some connectors (MMO games) may use "world" on both sides
- **Messaging Server Abstractions**:
  - CLI uses: server, channel, user
  - Frontend client unaware of worlds/rooms
  - These are purely agent-side abstractions

### Service Architecture

- Services maintain system state
- Access pattern: `getService(serviceName)`
- Services can call each other
- Actions can access services

## Component Specifications

### Actions

**Purpose**: Define agent capabilities and response mechanisms

**Decision Flow**:

1. Message received
2. Agent evaluates all actions via validation functions
3. Valid actions provided to LLM via actionsProvider
4. LLM decides which action(s) to execute
5. Handler generates response with "thought" component
6. Response processed and sent

### Providers

**Purpose**: Supply dynamic contextual information - agent's "senses"

**Functionality**:

- Inject real-time information into agent context
- Bridge between agent and external systems
- Format information for conversation templates
- Maintain consistent data access

**Examples**:

- News provider: Fetch and format news
- Terminal provider: Game terminal information
- Wallet provider: Current asset information
- Time provider: Current date/time injection

**Execution**: Run during or before action execution

### Evaluators

**Purpose**: Post-interaction cognitive processing

**Capabilities**:

- Knowledge extraction and storage
- Relationship tracking between entities
- Conversation quality self-reflection
- Goal tracking and achievement
- Tone analysis for future adjustments

**Execution**: Run after response generation with AgentRuntime

### Tasks

**Purpose**: Manage deferred, scheduled, and interactive operations

**Features**:

- Queue work for later execution
- Repeat actions at defined intervals
- Await user input
- Implement multi-interaction workflows
- Task workers registered by name with runtime

### Plugins


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bio-xyz/BioAgentsEliza](https://github.com/bio-xyz/BioAgentsEliza) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
