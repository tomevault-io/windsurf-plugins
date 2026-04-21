---
trigger: always_on
description: - `src/index.ts` defines the worker and capabilities.
---

# Repository Guidelines

## Project Structure & Module Organization
- `src/index.ts` defines the worker and capabilities.
- `.examples/` has focused samples (sync, tool, automation, OAuth).
- Shared agent skills live in `.agents/skills/`. `.claude/skills` is kept as a compatibility symlink for Claude-specific discovery.
- Generated: `dist/` build output, `workers.json` CLI config.

## Worker & Capability API (SDK)
- `@notionhq/workers` provides `Worker`, schema helpers, and builders; the `ntn` CLI powers worker management.
- Capability keys are unique strings used by the CLI (e.g., `ntn workers exec tasksSync`).

```ts
import { Worker } from "@notionhq/workers";
import * as Builder from "@notionhq/workers/builder";
import * as Schema from "@notionhq/workers/schema";

const worker = new Worker();
export default worker;

// Declare a database
const tasks = worker.database("tasks", {
	type: "managed",
	initialTitle: "Tasks",
	primaryKeyProperty: "ID",
	schema: { properties: { Name: Schema.title(), ID: Schema.richText() } },
});

// Declare a pacer for the upstream API
const myApi = worker.pacer("myApi", { allowedRequests: 10, intervalMs: 1000 });

// Declare a sync that writes to the database
worker.sync("tasksSync", {
	database: tasks,
	execute: async (state) => {
		await myApi.wait();
		const items = await fetchItems(state?.page ?? 1);
		return {
			changes: items.map((i) => ({
				type: "upsert" as const, key: i.id,
				properties: { Name: Builder.title(i.name), ID: Builder.richText(i.id) },
			})),
			hasMore: false,
		};
	},
});

worker.tool("sayHello", {
	title: "Say Hello",
	description: "Return a greeting",
	schema: { type: "object", properties: { name: { type: "string" } }, required: ["name"], additionalProperties: false },
	execute: ({ name }, { notion }) => `Hello, ${name}`,
});

worker.oauth("googleAuth", { name: "my-google-auth", provider: "google" });
```

- All `execute` handlers receive a Notion SDK client in the second argument as `context.notion`.

- For user-managed OAuth, supply `name`, `authorizationEndpoint`, `tokenEndpoint`, `clientId`, `clientSecret`, and `scope` (optional: `authorizationParams`, `callbackUrl`, `accessTokenExpireMs`).
- After deploying a worker with an OAuth capability, the user must configure their OAuth provider's redirect URL to match the one assigned by Notion. Run `ntn workers oauth show-redirect-url` to get the redirect URL, then set it in the provider's OAuth app settings. **Always remind the user of this step after deploying any OAuth capability.**

### Sync

#### Databases, Pacers, and Syncs

Syncs write data into Notion databases. Databases are declared separately and referenced by handle:

```ts
// 1. Declare a database
const tasks = worker.database("tasks", {
	type: "managed",
	initialTitle: "Tasks",
	primaryKeyProperty: "Task ID",
	schema: {
		properties: {
			"Task Name": Schema.title(),
			"Task ID": Schema.richText(),
			Status: Schema.select([{ name: "Open" }, { name: "Done", color: "green" }]),
		},
	},
});

// 2. Declare a pacer for the upstream API
const myApi = worker.pacer("myApi", { allowedRequests: 10, intervalMs: 1000 });

// 3. Declare a sync
worker.sync("tasksSync", {
	database: tasks,
	schedule: "30m",
	execute: async (state) => {
		await myApi.wait();
		const { items, hasMore } = await fetchTasks(state?.page ?? 1);
		return {
			changes: items.map((item) => ({
				type: "upsert" as const,
				key: item.id,
				properties: {
					"Task Name": Builder.title(item.name),
					"Task ID": Builder.richText(item.id),
					Status: Builder.select(item.status),
				},
			})),
			hasMore,
			nextState: hasMore ? { page: (state?.page ?? 1) + 1 } : undefined,
		};
	},
});
```

Multiple syncs can write to the same database. Multiple syncs can share a pacer — the server apportions the budget evenly across all syncs that use it.

#### Pacers (Rate Limiting)

**Always declare a pacer** for any sync that calls an external API. Research the API's rate limits before implementing. If the limits are variable (e.g. Salesforce, where you can purchase more API calls), ask the user what budget to allocate.

- Call `await pacer.wait()` before **every** API request inside `execute`.
- The pacer ensures requests are evenly spaced over the interval window.
- If 4 syncs share a pacer with `allowedRequests: 100, intervalMs: 60_000`, each sync gets ~25 requests/minute.

```ts
const myApi = worker.pacer("myApi", { allowedRequests: 10, intervalMs: 1000 });

// Inside execute:
await myApi.wait();
const data = await fetchFromApi();
```

#### Choosing a Sync Strategy

**Simple replace sync** — For truly small data sources (<1k records) or APIs with no change-tracking support. One sync, replace mode. Every cycle returns the full dataset; records not returned are deleted via mark-and-sweep.

**Backfill + delta pair** — For everything else (recommended for most real integrations). Two syncs writing to the same database:
- **Backfill** (replace mode, `schedule: "manual"`): Paginates the entire upstream dataset. Triggered manually via CLI. Cleans up drift, backfills new schema properties, catches deletes the delta can't detect.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makenotion/workers-template](https://github.com/makenotion/workers-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
