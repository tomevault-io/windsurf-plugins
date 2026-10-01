---
trigger: always_on
description: The `@kanjieteam/kjdraw/agent-tools` entry point exports `KJAgentToolSession`, which gives any tool-calling model a controlled way to work with a KJDraw document. It exposes JSON-serializable tool definitions for reading, querying, measuring, checking and proposing CAD changes. Proposal tools return exact geometry for review and do not edit the document.
---

:::en
## Build a reviewed CAD workflow {#agent-tools}

The `@kanjieteam/kjdraw/agent-tools` entry point exports `KJAgentToolSession`, which gives any tool-calling model a controlled way to work with a KJDraw document. It exposes JSON-serializable tool definitions for reading, querying, measuring, checking and proposing CAD changes. Proposal tools return exact geometry for review and do not edit the document.

The host creates the session for one authorized `KJDocument`, sends selected tool definitions to the model, dispatches model calls through `session.call(name, arguments)`, and keeps approval in its own trusted user interface. The same API works with different model providers; provider connection examples are covered in [Models and harnesses](https://kanjieteam.github.io/kjdraw/docs/latest/models/).

### Capabilities {#capabilities}

- Read the current revision, units, layers, layouts and bounded drawing geometry.
- Query a user selection, object types, layers, owner space or an XY region without sending the entire file.
- Measure explicit points and check stated geometric requirements against native CAD objects.
- Propose editable native geometry and common edits with a before/after preview.
- Apply an approved proposal as one undoable transaction, with revision and argument checks before commit.
- Add project-specific guidance through host-trusted `KJAgentCapabilityRegistry` manifests without adding executable code to the model tool path.

KJDraw enforces the published input schemas again when `call()` runs. Tool descriptions guide the model, while the CAD core remains responsible for geometry, document revisions, limits and commit behavior.

## Install and import {#install}

Install the package in the application that owns the drawing and review UI:

```sh
npm install @kanjieteam/kjdraw@next
```

Import the SDK from the package root and Agent tools from the `agent-tools` entry point:

```ts
import { createKJDrawSDK } from '@kanjieteam/kjdraw'
import { KJAgentToolSession } from '@kanjieteam/kjdraw/agent-tools'

const sdk = createKJDrawSDK()
const drawing = sdk.createDocument({ units: 'millimeter' })
const session = new KJAgentToolSession(sdk, drawing)
```

`session.definitions` contains the tools available to that session. Definitions include each tool's `name`, `description`, `effect` and `inputSchema`; unit fields are restricted to the document's canonical unit name, such as `millimeter`. Preserve these schema constraints when adapting them to a provider.

The package requires Node.js 22 or later for its Node examples. To verify the installed tool-session path without a model or API key, run:

```sh
node node_modules/@kanjieteam/kjdraw/examples/agent-tools.mjs
```

The example proposes a circle, simulates the host approval step, rejects a duplicate approval, reopens the native file and verifies Undo. It checks integration behavior rather than natural-language drawing quality.

## Shortest working flow {#quickstart}

This runnable example creates a session and asks for a circle proposal. The document remains unchanged while the proposal is waiting for review:

```ts
import { createKJDrawSDK } from '@kanjieteam/kjdraw'
import {
  KJAgentToolSession,
  type KJAgentGeometryPreview,
} from '@kanjieteam/kjdraw/agent-tools'

const sdk = createKJDrawSDK()
const drawing = sdk.createDocument({ units: 'millimeter' })
const session = new KJAgentToolSession(sdk, drawing)

const result = await session.call('cad_propose_circles', {
  expectedRevision: drawing.revision,
  units: 'millimeter',
  circles: [{ center: { x: 20, y: 20 }, radius: 3 }],
})
if (!result.ok) throw new Error(`${result.error.code}: ${result.error.message}`)

const proposal = result.value as {
  status: 'awaiting-host-approval'
  preview: KJAgentGeometryPreview
}
console.log(proposal.status, proposal.preview)
if (drawing.revision !== 0) throw new Error('A proposal must not edit the drawing')
```

Connect it to a model in four steps:

1. Give the provider adapter the selected entries from `session.definitions`.
2. Forward each model tool call to `session.call(name, arguments)` and return the result to the same model conversation.
3. When a proposal succeeds, show its exact arguments and `value.preview` to the reviewer.
4. After the host authenticates the reviewer and checks permission, call `session.approve(planId, reviewerId)` or `session.reject(planId, reviewerId)` from the host action.

Do not include `approve` or `reject` in the model's tool list. A reviewer ID string identifies the decision in KJDraw; authentication and authorization happen in the host application.

## Choose the right tool {#tool-selection}

Start with the narrowest tool that matches the task. The table lists the common entry points; inspect `session.definitions` for the complete tool set and its current schemas.

| Tool | Use it for |
| --- | --- |
| `cad_read_drawing` | Read the first bounded page of visible model-space objects, layers, units and revision |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KanJieTeam/kjdraw](https://github.com/KanJieTeam/kjdraw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
