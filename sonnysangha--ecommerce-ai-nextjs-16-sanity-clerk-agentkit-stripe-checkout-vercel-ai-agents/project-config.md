---
trigger: always_on
description: Anything related to Sanity usage
---

# App SDK best practices

If you’ve worked with Sanity before, your experience querying the Content Lake is likely grounded in building Server-Side Rendered (SSR) or statically generated front-end applications designed for page load time performance.

Now, with the Sanity App SDK, you can build feature-rich content applications for authoring. However, this requires a different approach: swapping SSR thinking for Single-Page Application (SPA) best practices.

On top of this, if you’re used to writing React applications, some common patterns for building form-based user interfaces are best avoided when working with App SDK.

## What makes a great content application?

Content applications are defined as distinct, new experiences that give authors a focused environment to perform content operations. Instead of digging through a general-purpose CMS interface, authors work in a fit-for-purpose user interface to get the job done.

Content applications developed with the Sanity App SDK should be:

### Real-time

Any number of documents fetched and rendered into the user interface should continue to update as mutations happen to the source documents. Content applications should avoid concepts that handle stale data like "submit," “save” or "lock" buttons.

### Multiplayer

Two authors looking at the same document should be able to continually make and see edits without fear of overwriting one another’s work.

### Fast

Content rendered in the application should be locally cached, updated optimistically, and kept eventually consistent with the Content Lake.

### Accurate

There should never be stale data in an author's browser as they write content, nor after page load when fetched content is rendered. Updates should be written to and received directly from the Content Lake.

### This is all built-in to Sanity App SDK

These are the baseline expectations that Sanity’s engineers have had while developing Sanity Studio since 2017, and they’re now democratized for everyone to take advantage of via React Hooks in the Sanity App SDK.

## Get comfortable with more fetches

If you’ve built an SSR front end with Sanity before (such as in Next.js), you’ve likely created a Sanity Client and fetched all on-page content in a single query like this.

```tsx
// The SSR way: query and render "event" type documents

import { client } from "../sanity/client";

export async function Page() {
  const events = await client.fetch(
    `*[_type == "event"]`
  );

  return (
    <ul>
      {events.map((event) => (
        <li key={event._id}>{event.title}</li>
      ))}
    </ul>
  );
}
```

This can work great for SSR apps—where only the initial page load is important—since the grunt work of optimization is done behind the scenes, cached and delivered fast in a static format to your end users. But it falls short of a great SPA experience which may involve querying and editing an evolving number and type of documents, while keeping the user interface up to date in real-time.

### Prefer useDocuments over useQuery to fetch documents

Your natural inclination may be to use the App SDK hooks to recreate the "fetch everything in one query" pattern.

```tsx
// ❌ Do not simply swap client.fetch for useQuery
// It's too easy to over-fetch!

import { useQuery } from "@sanity/sdk-react";

export function Page() {
  const { data: events } = useQuery(
    `*[_type == "event"]`
  );

  if (!events) return null;

  return (
    <ul>
      {events.map((event) => (
        <li key={event._id}>{event.title}</li>
      ))}
    </ul>
  );
}
```

This list of documents will receive real-time updates—an upgrade from `client.fetch`—but may unknowingly fetch 1000’s of documents, each with 100’s of attributes.

> [!WARNING]
> Keeping raw GROQ queries performant
> The query in this particular example is problematic for performance. There’s no “array slicing” such as `[0..10]` to reduce the total number of documents returned, and no projection such as `{ title }` to reduce the number of attributes returned. 
> High performance is built-in when you use hooks like `useDocuments` and `usePaginatedDocuments` to return a filtered list of [document handles](/docs/app-sdk/document-handles), but your implementation will need to be more carefully considered when fetching by GROQ queries with `useQuery`.

`useQuery` exists to fetch content with a GROQ query should you need to—but makes it your responsibility to maintain your application’s performance. One particular example of where this may be useful is when a parent component needs all the details of child documents. 

In most cases, you should prefer `useDocuments` to fetch a list of document handles, and render components that do their own data fetching for more content.

> [!TIP]
> Document handles provide stable `key` values
> Among the benefits of fetching for and using [document handles](/docs/app-sdk/document-handles) is that they provide a stable `documentId` attribute which can be used as the `key` value when mapping over the response to render a list. 
> Stable unique identifiers are preferable to using the index when [rendering lists in a real-time React application](https://react.dev/learn/rendering-lists#keeping-list-items-in-order-with-key).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sonnysangha/ecommerce-ai-nextjs-16-sanity-clerk-agentkit-stripe-checkout-vercel-ai-agents](https://github.com/sonnysangha/ecommerce-ai-nextjs-16-sanity-clerk-agentkit-stripe-checkout-vercel-ai-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
