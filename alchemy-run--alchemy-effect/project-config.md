---
trigger: always_on
description: Alchemy Effect is an Infrastructure-as-Effects (IaE) framework that extends Infrastructure-as-Code (IaC) by combining business logic and infrastructure config into a single, type-safe program expressed as Effects.
---

# alchemy

Alchemy Effect is an Infrastructure-as-Effects (IaE) framework that extends Infrastructure-as-Code (IaC) by combining business logic and infrastructure config into a single, type-safe program expressed as Effects.

It includes a core IaC engine built with Effect. Effect provides the foundation for type-safe, composable, and testable infrastructure programs. It brings errors into the type-system and provides declarative/composable retry logic that ensure proper and reliable handling of failures.

# Concepts

- **Cloud Provider** - a cloud provider that offers a set of Services, e.g. AWS, Azure, GCP, Cloudflare, Stripe, Planetscale, Neon, etc.
- **Service** - a collection of Resources, Functions, and Bindings offered by a Cloud Provider.
- **Resource** - a named entity that is configuted with "Input Properties" and produces "Output Attributes". May or may not have Binding Contract.
- **Input Properties** - the properties passed as input to configure a Resource. Otherwise known as the "desired state" of the Resource.
- **Output Attributes** - the attributes produced by a Resource. Otherwise known as the "current state" of the Resource.
- **Stable Properties** - properties that are not affected by an Update, e.g. the ID or ARN of a Resource.
- **Function** (aka. **Runtime**) - a special kind of Resource that includes a runtime implementation expressed as a Function producing an `Effect<A, Err, Req>`. The `Req` type captures runtime dependencies, from which Infrastructure Dependencies are inferred.
- **Resource Provider** (see [Provider](./alchemy/src/Provider.ts))

A Resource Provider implements the following Lifecycle Operations:

- **Diff** - compares new props with old props and determines if the Resource needs to be updated or replaced. For updates, it can also specify a list of Stable Properties that will not be changed by the update.
- **Read** - reads the current state of a Resource and returns the current Output Attributes.
- **Pre-Create** - an optional operation that can be called to create a stub of a Resource before the actual create operation is called. This is useful for resolving circular dependencies since it allows for an empty resoruce to be created and then updated later with its dependencies (e.g. Function A and B depend on each other, so we can create a stub of Function A and then update it with the actual Function B later).
- **Create** - creates a new Resource. It must be designed as idempotent because it is always possible for state persistence to fail after the create operation is called. There are various techniques for resolving idempotency, such as deterministic physical name generation and resource tagging.
- **Update** - updates an existing Resource with new Input Properties.
- **Delete** - deletes an existing Resource. It must be designed as idempotent because it is always possible for state persistence to fail after the delete operation is called. If the resource doesn't exist during deletion, it should not be considered an error.
- **Capability** - a runtime requirement of a Function (e.g. require `SQS.SendMessage` on a `SQS.Queue`). Each Capability is split into two parts: a `Binding.Service` (runtime SDK wrapper) and a `Binding.Policy` (deploy-time IAM/binding attachment).
- **Binding.Service** - an Effect Service that wraps an SDK client and exposes a `.bind(resource)` method returning a typed callable for runtime use. Provided as a Layer on the **Function** Effect so it gets bundled into the Lambda/Worker. See [Binding](./alchemy/src/Binding.ts).
- **Binding.Policy** - an Effect Service that runs only at deploy time to attach IAM policies (AWS) or bindings (Cloudflare) to a Function's role/config. At runtime, `Binding.Policy` uses `Effect.serviceOption` so it gracefully becomes a no-op when the layer is not provided. Policy layers are provided on the **Stack** via `AWS.providers()()`, not on the Function.
- **Binding** - data attached to a Resource via `resource.bind(data)`. A Binding is a `{ context: PolicyContext, data: BindingData }` tuple that is collected on the Stack during plan/deploy. Bindings enable circular references between Resources — the `Binding.Policy` calls `ctx.bind({ policyStatements: [...] })` on the target Function, which records the binding data on the Stack. The Resource Provider then receives the resolved binding data in its `create`/`update` lifecycle operations via the `bindings` parameter.
- **Binding Contract** - the shape of data a Resource accepts from Bindings. For example, a Lambda Function accepts `{ env?: Record<string, any>, policyStatements?: PolicyStatement[] }` because it needs environment variables and IAM policies. A Cloudflare Worker accepts `{ bindings: Worker.Binding[] }` for its native binding system. The Binding Contract is declared as the fourth type parameter on the `Resource` interface. See [Lambda Function](./alchemy/src/AWS/Lambda/Function.ts) and [Cloudflare Worker](./alchemy/src/Cloudflare/Workers/Worker.ts).
- **Dependency** - Resources depend on other Resources through two mechanisms:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alchemy-run/alchemy-effect](https://github.com/alchemy-run/alchemy-effect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
