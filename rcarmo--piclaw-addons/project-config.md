---
trigger: always_on
description: Validates that each addon can be imported without crashing.
---

# Developing piclaw add-ons

This guide covers how to create, test, and publish an extension for [piclaw](https://github.com/rcarmo/piclaw).

---

## Quick start

> **Install-path rule:** first-party `piclaw-addons` must install via **public GitHub-hosted tarball URLs** from `catalog.json`.
> Do **not** change docs, generated catalog entries, or runtime integration back to npmjs.org package specs or authenticated GitHub Packages reads.
> Runtime install/remove must remain zero-auth.


```bash
# 1. Create your addon directory
mkdir -p addons/my-addon/skills/my-skill

# 2. Write your entry point, package.json, and skill
# 3. Sync the catalog
bun run sync:catalog

# 4. Type-check
bunx tsc --noEmit

# 5. Push — CI handles the rest
git add addons/my-addon && git commit -m "feat: add my-addon" && git push
```

---

## Addon structure

> **Important:** standalone add-on packages must be self-contained.
> If an add-on is published as its own npm package (for example `@rcarmo/piclaw-addon-portainer`), it must not rely on repo-root files outside its package directory at runtime. Do not import `../../lib/compat/*` from a published standalone package unless those files are vendored into that package.

```
addons/<slug>/
├── index.ts          # Runtime entry point (default export)
├── web/
│   └── index.ts      # Optional browser-side settings pane / web entry
├── package.json      # Package manifest
├── skills/           # Optional: agent skills
│   └── my-skill/
│       └── SKILL.md
└── *.ts              # Supporting modules
```

---

## Entry point

The default export is a function that receives the `ExtensionAPI`:

```ts
import type { ExtensionAPI } from "@mariozechner/pi-coding-agent";
import { dirname, join } from "node:path";
import { fileURLToPath } from "node:url";

const baseDir = dirname(fileURLToPath(import.meta.url));

export default function myAddon(pi: ExtensionAPI) {
  // Register skills for agent discovery
  pi.on("resources_discover", () => ({
    skillPaths: [join(baseDir, "skills", "my-skill", "SKILL.md")],
  }));

  // Register a tool
  pi.registerTool({
    name: "my_tool",
    label: "my_tool",
    description: "What this tool does.",
    parameters: MyToolSchema,
    async execute(_toolCallId, params, _signal, _update, ctx) {
      return { content: [{ type: "text", text: "result" }] };
    },
  });
}
```

---

## package.json

```json
{
  "name": "@rcarmo/piclaw-addon-<slug>",
  "version": "0.1.0",
  "description": "One-line description",
  "type": "module",
  "main": "index.ts",
  "piclaw": {
    "type": "extension",
    "tags": ["relevant", "tags"]
  },
  "pi": {
    "extensions": ["index.ts"],
    "web": {
      "entries": ["web/index.ts"]
    },
    "skills": ["skills"]
  },
  "peerDependencies": {
    "@mariozechner/pi-coding-agent": "*",
    "@sinclair/typebox": "*"
  },
  "keywords": ["piclaw", "piclaw-addon"],
  "license": "MIT"
}
```

| Field | Required | Notes |
|---|---|---|
| `name` | ✓ | `@rcarmo/piclaw-addon-<slug>` |
| `version` | ✓ | Bump on every functional change |
| `description` | ✓ | Shown in the catalog and web UI |
| `piclaw.type` | ✓ | `"extension"` or `"skill"` |
| `piclaw.tags` | ✓ | Categorisation for search and display |
| `pi.extensions` | ✓ | Entry points — usually `["index.ts"]` |
| `peerDependencies` | ✓ | Must declare both `@mariozechner/pi-coding-agent` and `@sinclair/typebox` |


---

## Skills

A skill teaches the agent *when and how* to use your tools:

```
addons/<slug>/skills/<skill-name>/SKILL.md
```

Front matter:
```yaml
---
name: my-skill
description: What this skill teaches the agent
distribution: public
---
```

Register skills via `resources_discover`:
```ts
pi.on("resources_discover", () => ({
  skillPaths: [join(baseDir, "skills", "my-skill", "SKILL.md")],
}));
```

---

## Extension API reference

| Capability | Method |
|---|---|
| Register tools | `pi.registerTool({ name, parameters, execute })` |
| Lifecycle hooks | `pi.on("before_agent_start", fn)` |
| Resource discovery | `pi.on("resources_discover", fn)` |
| Interactive UI | `ctx.ui.select()`, `.confirm()`, `.input()` |
| Progress | `ctx.ui.setWorkingMessage(text)` |
| Status | `ctx.ui.setStatus(key, text)` |
| Widgets | `ctx.ui.setWidget(key, content, options)` |
| Toasts | `ctx.ui.notify(message, type)` |

### Tool parameters

Use `@sinclair/typebox`:

```ts
import { Type } from "@sinclair/typebox";

const Params = Type.Object({
  action: Type.Union([Type.Literal("get"), Type.Literal("list")]),
  id: Type.Optional(Type.String()),
});
```

### KV storage

Persist config or state:

```ts
import { createExtensionStorage } from "../../lib/compat/extension-kv.js";

const kv = createExtensionStorage("my-addon");
kv.set("config", value, "chat", chatJid);   // per-chat
kv.set("prefs", value, "global");            // cross-chat
```

### Settings panes and direct config API

For add-ons that expose a **Settings** pane:

#### Runtime side

Register config handlers directly from the runtime entry using the global registrar exposed by piclaw:

```ts
const registerAddonConfigApi = globalThis.__piclaw_registerAddonConfigApi;

registerAddonConfigApi?.("my-addon", "config", {
  get: async () => loadConfig(),
  set: async (payload) => {
    const next = saveConfig(payload);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rcarmo/piclaw-addons](https://github.com/rcarmo/piclaw-addons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
