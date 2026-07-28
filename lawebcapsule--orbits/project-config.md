---
trigger: always_on
description: A Agent is a specialized type of Workflow designed to orchestrate the lifecycle of real-world entities, such as cloud accounts, services, or data systems. Agents provide a declarative and persistent way to manage these entities through versioning, reconciliation, shared state, and scheduled verification.
---


# Agent documentation

A Agent is a specialized type of Workflow designed to orchestrate the lifecycle of real-world entities, such as cloud accounts, services, or data systems. Agents provide a declarative and persistent way to manage these entities through versioning, reconciliation, shared state, and scheduled verification.

Agents offer:

- Retroactive control loops;
- Shared workflow state via identity;
- Output reuse across executions;
- Hookable lifecycle stages: install, update, uninstall, and cycle.

## Write a Agent

A Agent is a `Workflow` with some specials behaviors.

### The `identity` of a agent

```ts
export class MyAgent extends Agent {
    IArgument: {
        accountId: string;
    };

    identity() {
        return this.argument.accountId;
    }
}
```

The identity() method defines a unique key for the agent. All agent instances with the same constructor and identity value will share the same database document, lifecycle state, and output.

This enables deduplication and stateful coordination between concurrent invocations.

#### Persistent storage for agents

Each agent is backed by a shared database document, accessible via the `agentDbDoc` property.
Some properties of the document are internal to the framework and should not be modified unless you know what you are doing.

#### Agent output

Each agent can define an output with the `setOutput` method.
Outputs are stored persistently and are available to other workflows or agent cycles.

```ts
export class MyAgent extends Agent {
    IArgument: {
        accountId: string;
    };

    identity() {
        return this.argument.accountId;
    }

    setOutput() {
        return {
            x: 1,
        };
    }
}
```

Outputs can then be consumed by other workflows or agents:

```ts
export class MySecondAgent extends Agent {
    IArgument: {
        accountId: string;
    };

    identity() {
        return this.argument.accountId;
    }

    setOutput() {
        return {
            x: 1,
        };
    }

    async defineUpdate() {
        const output = await this.do(
            'get-output',
            new MyAgent().setArgument({
                accountId: this.argument.accountId,
            }).getAgentOutput
        );
        await this.do('deploy', () => {
            return new Service(output).create();
        });
    }
}
```

### Agent lifecycle commands

Agents define lifecycle hooks. These correspond to specific `setCommand()` values and are executed based on state or schedule.

#### Install hook

```ts
export class MyAgent extends Agent {
    IArgument: {
        accountId: string;
    };

    identity() {
        return this.argument.accountId;
    }

    version = '1.0.0';

    async defineInstall() {
        const myAccount = new Account();
        await this.do('bootstrap', myAccount);
    }
}
```

`defineInstall` runs:

- when the agent has never been installed before;
- when the version field changes;
- when explicitly triggered with `.setCommand("install")`.

#### Update

```ts
export class MyAgent extends Agent {
    IArgument: {
        accountId: string;
    };

    identity() {
        return this.argument.accountId;
    }

    version = '1.0.0';

    async defineInstall() {
        const myAccount = new Account();
        await this.do('bootstrap', myAccount);
    }

    async defineUpdate() {
        const service = new Service().setArgument({
            accountId: this.argument.accountId,
        });
        await this.do('launchService', service);
    }
}
```

`defineUpdate` runs:

- every time a default execution runs;
- if you force the execution of the `update` step using `.setCommand('update')`

#### Uninstall

```ts
export class MyAgent extends Agent {
    IArgument: {
        accountId: string;
    };

    identity() {
        return this.argument.accountId;
    }

    version = '1.0.0';

    async defineInstall() {
        const myAccount = new Account();
        await this.do('bootstrap', myAccount);
    }

    async defineUpdate() {
        const service = new Service().setArgument({
            accountId: this.argument.accountId,
        });
        await this.do('launchService', () => {
            return service.create();
        });
    }

    async defineUninstall() {
        const service = new Service().setArgument({
            accountId: this.argument.accountId,
        });
        await this.do('launchService', () => {
            return service.delete();
        });
    }
}
```

`defineUninstall()` runs only if explicitly triggered with `.setCommand("uninstall")`

#### Cycle

Agents support a `defineCycle()` method, triggered periodically to verify or reconcile the external-world state.

```ts
export class MyAgent extends Agent {
    IArgument: {
        accountId: string;
    };

    identity() {
        return this.argument.accountId;
    }

    version = '1.0.0';

    async defineInstall() {
        const myAccount = new Account();
        await this.do('bootstrap', myAccount);
    }

    async defineUpdate() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LaWebcapsule/orbits](https://github.com/LaWebcapsule/orbits) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
