---
trigger: always_on
description: [**@cdklabs/generative-ai-cdk-constructs**](../../../../README.md)
---

[**@cdklabs/generative-ai-cdk-constructs**](../../../../README.md)

***

[@cdklabs/generative-ai-cdk-constructs](../../../../README.md) / [bedrock](../README.md) / Agent

# Class: Agent

Class to create (or import) an Agent with CDK.

## Cloudformation Resource

AWS::Bedrock::Agent

## Extends

- [`AgentBase`](AgentBase.md)

## Constructors

### Constructor

> **new Agent**(`scope`, `id`, `props`): `Agent`

#### Parameters

##### scope

`Construct`

##### id

`string`

##### props

[`AgentProps`](../interfaces/AgentProps.md)

#### Returns

`Agent`

#### Overrides

[`AgentBase`](AgentBase.md).[`constructor`](AgentBase.md#constructor)

## Properties

### actionGroups

> **actionGroups**: [`AgentActionGroup`](AgentActionGroup.md)[]

The action groups associated with the agent.

***

### agentArn

> `readonly` **agentArn**: `string`

The ARN of the agent.

#### Overrides

[`AgentBase`](AgentBase.md).[`agentArn`](AgentBase.md#agentarn)

***

### agentCollaboration?

> `readonly` `optional` **agentCollaboration?**: [`AgentCollaboratorType`](../enumerations/AgentCollaboratorType.md)

Agent collaboration type.

***

### agentCollaborators?

> `readonly` `optional` **agentCollaborators?**: [`AgentCollaborator`](AgentCollaborator.md)[]

Agent collaborators.

***

### agentId

> `readonly` **agentId**: `string`

The unique identifier for the agent

#### Overrides

[`AgentBase`](AgentBase.md).[`agentId`](AgentBase.md#agentid)

***

### agentVersion

> **agentVersion**: `string`

The version of the agent.

#### Overrides

[`AgentBase`](AgentBase.md).[`agentVersion`](AgentBase.md#agentversion)

***

### codeInterpreterEnabled

> `readonly` **codeInterpreterEnabled**: `boolean`

Whether the agent can generate, run, and troubleshoot code when trying to complete a task.

***

### customOrchestration?

> `readonly` `optional` **customOrchestration?**: [`CustomOrchestration`](../interfaces/CustomOrchestration.md)

Custom orchestration configuration.

***

### description?

> `readonly` `optional` **description?**: `string`

The description for the agent.

***

### forceDelete

> `readonly` **forceDelete**: `boolean`

Whether the resource will be deleted even if it's in use.

***

### foundationModel

> `readonly` **foundationModel**: [`IInvokable`](../interfaces/IInvokable.md)

The foundation model used for orchestration by the agent.

***

### guardrail?

> `optional` **guardrail?**: [`IGuardrail`](../interfaces/IGuardrail.md)

The guardrail associated with the agent.

***

### idleSessionTTL

> `readonly` **idleSessionTTL**: `Duration`

How long sessions should be kept open for the agent.

***

### instruction?

> `readonly` `optional` **instruction?**: `string`

The instruction used by the agent. This determines how the agent will perform his task.

***

### kmsKey?

> `readonly` `optional` **kmsKey?**: `IKey`

Optional KMS encryption key associated with this agent

#### Overrides

[`AgentBase`](AgentBase.md).[`kmsKey`](AgentBase.md#kmskey)

***

### knowledgeBases

> **knowledgeBases**: [`IKnowledgeBase`](../interfaces/IKnowledgeBase.md)[]

The KnowledgeBases associated with the agent.

***

### lastUpdated?

> `readonly` `optional` **lastUpdated?**: `string`

When this agent was last updated.

#### Overrides

[`AgentBase`](AgentBase.md).[`lastUpdated`](AgentBase.md#lastupdated)

***

### memory?

> `readonly` `optional` **memory?**: [`Memory`](Memory.md)

The memory configuration for the agent.

***

### name

> `readonly` **name**: `string`

The name of the agent.

***

### node

> `readonly` **node**: `Node`

The tree node.

#### Inherited from

[`AgentBase`](AgentBase.md).[`node`](AgentBase.md#node)

***

### orchestrationType?

> `readonly` `optional` **orchestrationType?**: [`OrchestrationType`](../enumerations/OrchestrationType.md)

The type of orchestration for the agent.

***

### promptOverrideConfiguration?

> `readonly` `optional` **promptOverrideConfiguration?**: [`PromptOverrideConfiguration`](PromptOverrideConfiguration.md)

Overrides some prompt templates in different parts of an agent sequence configuration.

#### Default

```ts
- No overrides are provided.
```

***

### role

> `readonly` **role**: `IRole`

The IAM role associated to the agent.

#### Overrides

[`AgentBase`](AgentBase.md).[`role`](AgentBase.md#role)

***

### shouldPrepareAgent

> `readonly` **shouldPrepareAgent**: `boolean`

Whether the agent will automatically update the DRAFT version of the agent after
making changes to the agent.

***

### testAlias

> `readonly` **testAlias**: [`IAgentAlias`](../interfaces/IAgentAlias.md)

The default test alias for this agent. This corresponds to the test alias
(`TSTALIASID`) that points to the working (`DRAFT`) version.

***

### userInputEnabled

> `readonly` **userInputEnabled**: `boolean`

Whether the agent can prompt additional information from the user when it does not have
enough information to respond to an utterance

## Accessors

### env

#### Get Signature

> **get** **env**(): `ResourceEnvironment`

The environment this resource belongs to.

For resources that are created and managed in a Stack (those created by
creating new class instances like `new Role()`, `new Bucket()`, etc.), this
is always the same as the environment of the stack they belong to.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awslabs/generative-ai-cdk-constructs](https://github.com/awslabs/generative-ai-cdk-constructs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
