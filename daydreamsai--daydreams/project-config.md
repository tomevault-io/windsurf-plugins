---
trigger: always_on
description: [**@daydreamsai/core**](./api-reference.md)
---


[**@daydreamsai/core**](./api-reference.md)

***

[@daydreamsai/core](./api-reference.md) / Agent

# Interface: Agent\<TContext\>

Defined in: [packages/core/src/types.ts:672](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L672)

Represents an agent with various configurations and methods for handling contexts, inputs, outputs, and more.

## Template

The type of memory used by the agent.

## Extends

- `AgentDef`\<`TContext`\>

## Type Parameters

### TContext

`TContext` *extends* [`AnyContext`](./AnyContext.md) = [`AnyContext`](./AnyContext.md)

The type of context used by the agent.

## Properties

### actions

> **actions**: [`Action`](./Action.md)\<`any`, `any`, `unknown`, [`AnyContext`](./AnyContext.md), `Agent`\<`TContext`\>, [`ActionState`](./ActionState.md)\<`any`\>\>[]

Defined in: [packages/core/src/types.ts:638](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L638)

An array of actions available to the agent.

#### Inherited from

`AgentDef.actions`

***

### container

> **container**: `Container`

Defined in: [packages/core/src/types.ts:595](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L595)

The container used by the agent.

#### Inherited from

`AgentDef.container`

***

### context?

> `optional` **context**: `TContext`

Defined in: [packages/core/src/types.ts:585](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L585)

The current context of the agent.

#### Inherited from

`AgentDef.context`

***

### debugger

> **debugger**: [`Debugger`](./Debugger.md)

Defined in: [packages/core/src/types.ts:590](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L590)

Debugger function for the agent.

#### Inherited from

`AgentDef.debugger`

***

### emit()

> **emit**: (...`args`) => `void`

Defined in: [packages/core/src/types.ts:706](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L706)

Emits an event with the provided arguments.

#### Parameters

##### args

...`any`[]

Arguments to pass to the event handler.

#### Returns

`void`

***

### events

> **events**: `Record`\<`string`, `z.ZodObject`\>

Defined in: [packages/core/src/types.ts:633](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L633)

A record of event schemas for the agent.

#### Inherited from

`AgentDef.events`

***

### exports?

> `optional` **exports**: [`ExportManager`](./ExportManager.md)

Defined in: [packages/core/src/types.ts:711](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L711)

Export manager for episodes

***

### exportTrainingData?

> `optional` **exportTrainingData**: `boolean`

Defined in: [packages/core/src/types.ts:650](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L650)

Whether to export training data for episodes

#### Inherited from

`AgentDef.exportTrainingData`

***

### inputs

> **inputs**: `Record`\<`string`, [`InputConfig`](./InputConfig.md)\<`any`, [`AnyContext`](./AnyContext.md), `Agent`\<`TContext`\>\>\>

Defined in: [packages/core/src/types.ts:623](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L623)

A record of input configurations for the agent.

#### Inherited from

`AgentDef.inputs`

***

### logger

> **logger**: `Logger`

Defined in: [packages/core/src/types.ts:570](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L570)

#### Inherited from

`AgentDef.logger`

***

### memory

> **memory**: [`MemorySystem`](./MemorySystem.md)

Defined in: [packages/core/src/types.ts:580](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L580)

The memory store and vector store used by the agent.

#### Inherited from

`AgentDef.memory`

***

### model?

> `optional` **model**: `LanguageModel`

Defined in: [packages/core/src/types.ts:605](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L605)

The primary language model used by the agent.

#### Inherited from

`AgentDef.model`

***

### modelSettings?

> `optional` **modelSettings**: `object`

Defined in: [packages/core/src/types.ts:610](https://github.com/dojoengine/daydreams/blob/612e9304717c546d301f9cac8c204de734cac957/packages/core/src/types.ts#L610)

Model settings for the agent.

#### Index Signature

\[`key`: `string`\]: `any`

#### maxTokens?

> `optional` **maxTokens**: `number`

#### providerOptions?

> `optional` **providerOptions**: `Record`\<`string`, `any`\>

#### stopSequences?

> `optional` **stopSequences**: `string`[]

#### temperature?

> `optional` **temperature**: `number`

#### topK?

> `optional` **topK**: `number`

#### topP?

> `optional` **topP**: `number`

#### Inherited from

`AgentDef.modelSettings`

***


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daydreamsai/daydreams](https://github.com/daydreamsai/daydreams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
