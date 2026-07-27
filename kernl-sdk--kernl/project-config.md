---
trigger: always_on
description: Defined in: [packages/kernl/src/agent.ts:34](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent.ts#L34)
---


# Class: Agent\<TContext, TOutput\>

Defined in: [packages/kernl/src/agent.ts:34](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent.ts#L34)

## Extends

- `BaseAgent`\<`TContext`, `TOutput`\>

## Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| `TContext` | `UnknownContext` |
| `TOutput` *extends* `AgentOutputType` | `TextOutput` |

## Implements

- `AgentConfig`\<`TContext`, `TOutput`\>

## Constructors

### Constructor

```ts
new Agent<TContext, TOutput>(config: AgentConfig<TContext, TOutput>): Agent<TContext, TOutput>;
```

Defined in: [packages/kernl/src/agent.ts:52](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent.ts#L52)

#### Parameters

| Parameter | Type |
| ------ | ------ |
| `config` | `AgentConfig`\<`TContext`, `TOutput`\> |

#### Returns

`Agent`\<`TContext`, `TOutput`\>

#### Overrides

```ts
BaseAgent<TContext, TOutput>.constructor
```

## Properties

| Property | Modifier | Type | Default value | Description | Overrides | Inherited from | Defined in |
| ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| <a id="description"></a> `description?` | `readonly` | `string` | `undefined` | - | - | `AgentConfig.description` `Agent`.[`description`](#description) | [packages/kernl/src/agent/base.ts:58](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent/base.ts#L58) |
| <a id="guardrails"></a> `guardrails` | `readonly` | \{ `input`: `InputGuardrail`[]; `output`: `OutputGuardrail`\<`AgentOutputType`\>[]; \} | `undefined` | A list of checks that run in parallel to the agent's execution on the input + output for the agent, depending on the configuration. | - | - | [packages/kernl/src/agent.ts:45](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent.ts#L45) |
| `guardrails.input` | `public` | `InputGuardrail`[] | `undefined` | - | - | - | [packages/kernl/src/agent.ts:46](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent.ts#L46) |
| `guardrails.output` | `public` | `OutputGuardrail`\<`AgentOutputType`\>[] | `undefined` | - | - | - | [packages/kernl/src/agent.ts:47](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent.ts#L47) |
| <a id="id"></a> `id` | `readonly` | `string` | `undefined` | - | - | `AgentConfig.id` `Agent`.[`id`](#id) | [packages/kernl/src/agent/base.ts:56](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent/base.ts#L56) |
| <a id="instructions"></a> `instructions` | `readonly` | (`context`: [`Context`](Context.md)\<`TContext`\>) => `string` \| `Promise`\<`string`\> | `undefined` | - | - | `AgentConfig.instructions` `BaseAgent.instructions` | [packages/kernl/src/agent/base.ts:59](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent/base.ts#L59) |
| <a id="kernl"></a> `kernl?` | `protected` | [`Kernl`](Kernl.md) | `undefined` | - | - | `Agent`.[`kernl`](#kernl) | [packages/kernl/src/agent/base.ts:51](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent/base.ts#L51) |
| <a id="kind"></a> `kind` | `readonly` | `"llm"` | `"llm"` | - | `BaseAgent.kind` | - | [packages/kernl/src/agent.ts:41](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent.ts#L41) |
| <a id="memory"></a> `memory` | `readonly` | `AgentMemoryConfig` | `undefined` | - | - | `AgentConfig.memory` `Agent`.[`memory`](#memory) | [packages/kernl/src/agent/base.ts:64](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent/base.ts#L64) |
| <a id="model"></a> `model` | `readonly` | [`LanguageModel`](../../protocol/interfaces/LanguageModel.md) | `undefined` | - | `AgentConfig.model` `BaseAgent.model` | - | [packages/kernl/src/agent.ts:42](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent.ts#L42) |
| <a id="modelsettings"></a> `modelSettings` | `readonly` | [`LanguageModelRequestSettings`](../../protocol/interfaces/LanguageModelRequestSettings.md) | `undefined` | Configures model-specific tuning parameters (e.g. temperature, top_p, etc.) | - | - | [packages/kernl/src/agent.ts:43](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent.ts#L43) |
| <a id="name"></a> `name` | `readonly` | `string` | `undefined` | - | - | `AgentConfig.name` `Agent`.[`name`](#name) | [packages/kernl/src/agent/base.ts:57](https://github.com/kernl-sdk/kernl/blob/91f1cdb0bdd9506521d48da419c35fdb4b5a7eab/packages/kernl/src/agent/base.ts#L57) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kernl-sdk/kernl](https://github.com/kernl-sdk/kernl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
