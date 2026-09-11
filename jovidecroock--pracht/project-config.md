---
trigger: always_on
description: Today, when an AI agent needs to do something on a website — book a slot, file a ticket, buy the thing — it does what a scraper does: load the page, read the DOM, guess which `<button>` is real, and click. That is slow, brittle, and anonymous. And the same guessing that fills a search box can also hit "delete account." The site owner cannot tell agents from humans, cannot say which operations are safe, and finds out what happened from the support queue.
---


## The Web Has Two Users Now

Today, when an AI agent needs to do something on a website — book a slot, file a ticket, buy the thing — it does what a scraper does: load the page, read the DOM, guess which `<button>` is real, and click. That is slow, brittle, and anonymous. And the same guessing that fills a search box can also hit "delete account." The site owner cannot tell agents from humans, cannot say which operations are safe, and finds out what happened from the support queue.

pracht's bet: your app already knows its own operations — it has just never written them down in a form a machine could trust. So you write each one down **once**:

```ts [src/capabilities/book-appointment.ts]
import { defineCapability } from "@pracht/capabilities";

export default defineCapability({
  title: "Book appointment",
  description: "Reserve an open slot with the given service and time.",
  input: { /* JSON Schema */ },
  output: { /* JSON Schema */ },
  effect: "write",
  middleware: ["auth"],
  expose: { http: true, webmcp: true, mcp: true },
  async run({ input, context }) { /* your business logic */ },
});
```

Register it in the same `defineApp()` manifest that already holds your routes, shells, middleware, and API routes, and it joins the app graph. One contract. pracht projects it everywhere.

---

## One Graph, Four Projections

The manifest is not a routing config that happens to be explicit. It is a description of the application, which the framework resolves once and then aims at four different audiences.

**Your own code.** The loader behind the booking page calls `invokeCapability("appointments.book", …)` — same schema validation, same named middleware, same pipeline. The human UI and the agent surface cannot drift apart, because they are the same function.

**The browser.** Your island's click handler calls the generated `capabilities.appointments.book({ … })` client, or a `<Form capability>` posts to it with no JavaScript at all. The capability module never ships to the client: only its name, endpoint, and effect class cross, and importing the module from client code fails the build.

**An agent standing in the user's tab.** With `expose.webmcp` and the capability named in that route's `capabilities` list, the page registers the operation as a [WebMCP](https://developer.chrome.com/docs/ai/webmcp) page tool. Navigating away replaces it with the destination route's tool set. The agent stops guessing at your DOM and instead reads: *"book_appointment — reserve an open slot. Input: service, time."* It acts as the signed-in user, in their session, and every check still runs on your server.

**An agent that never opens a browser.** With `expose.mcp`, the same contract is served as a tool on your app's own remote MCP endpoint — `initialize`, `tools/list`, `tools/call`, straight over HTTP. No SDK, no second server, no separate tool definitions to keep in sync. An MCP host points at `https://your-app/mcp` and gets the same validation, middleware, identity checks, and audit events every other caller gets.

Every projection runs one pipeline:

```text
input validation → middleware chain → run() → output validation → audit event
```

There is no second copy of the rules that could drift from the first. The full API — `defineCapability`, `expose`, effect classes, typed clients, `<Form capability>`, and the remote MCP transport — is on [Capabilities](/docs/capabilities).

---

## Discovery: Markdown and llms.txt

A projection nobody can find is not a projection. Two mechanisms make the graph discoverable, and both are opt-in.

### One URL, Two Representations

pracht can serve the same route as either a normal HTML page or raw Markdown. Browsers keep receiving rendered HTML; agents that explicitly ask for Markdown get the source document with no navigation chrome, hydration state, or scraped layout noise.

```sh
# Human-readable HTML
curl https://pracht.resynapse.dev/docs/routing

# Agent-readable Markdown
curl -H "Accept: text/markdown" https://pracht.resynapse.dev/docs/routing
```

Any route opts in by exporting a `markdown` string. When the incoming request prefers `text/markdown`, pracht returns that string before running the render pipeline:

```tsx [src/routes/pricing.tsx]
export const markdown = `# Pricing

- Starter: free
- Pro: usage-based
- Enterprise: contact sales
`;

export function Component() {
  return <PricingPage />;
}
```

Markdown route modules compiled by [`defineMarkdownCollection`](/docs/content) — which is how every page on this site is built — get that export generated for them, so a docs site becomes an agent-readable endpoint without writing anything.

If middleware generates the Markdown instead — one dynamic route module serving a large document corpus, say — declare it in route metadata:

```ts [src/routes.ts]
route("/guide/:version/:name", "./routes/guide.tsx", {
  markdown: true,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JoviDeCroock/pracht](https://github.com/JoviDeCroock/pracht) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
