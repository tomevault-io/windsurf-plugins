---
trigger: always_on
description: TanStack Router: Setup and Architecture
---

# Overview

**TanStack Router is a router for building React and Solid applications**. Some of its features include:

- 100% inferred TypeScript support
- Typesafe navigation
- Nested Routing and layout routes (with pathless layouts)
- Built-in Route Loaders w/ SWR Caching
- Designed for client-side data caches (TanStack Query, SWR, etc.)
- Automatic route prefetching
- Asynchronous route elements and error boundaries
- File-based Route Generation
- Typesafe JSON-first Search Params state management APIs
- Path and Search Parameter Schema Validation
- Search Param Navigation APIs
- Custom Search Param parser/serializer support
- Search param middleware
- Route matching/loading middleware

To get started quickly, head to the next page. For a more lengthy explanation, buckle up while I bring you up to speed!

## "A Fork in the Route"

Using a router to build applications is widely regarded as a must-have and is usually one of the first choices you’ll make in your tech stack.

[//]: # 'WhyChooseTanStackRouter'

**So, why should you choose TanStack Router over another router?**

To answer this question, we need to look at the other options in the space. There are many if you look hard enough, but in my experience, only a couple are worth exploring seriously:

- **Next.js** - Widely regarded as the de facto framework for starting a new React project, it’s laser focused on performance, workflow, and bleeding edge technology. Its APIs and abstractions are powerful, but can sometimes come across as non-standard. Its extremely fast growth and adoption in the industry has resulted in a featured packed experience, but at the expense of feeling overwhelming and sometimes bloated.
- **Remix / React Router** - A full-stack framework based on the historically successful React Router offers a similarly powerful developer and user experience, with APIs and vision based firmly on web standards like Request/Response and a focus on running anywhere JS can run. Many of its APIs and abstractions are wonderfully designed and were inspiration for more than a few TanStack Router APIs. That said, its rigid design, bolted-on type safety and sometimes strict over-adherence to platform APIs can leave some developers wanting more.

Both of these frameworks (and their routers) are great, and I can personally attest that both are very good solutions for building React applications. My experience has also taught me that these solutions could also be much better, especially around the actual routing APIs that are available to developers to make their apps faster, easier, and more enjoyable to work with.

It's probably no surprise at this point that picking a router is so important that it is often tied 1-to-1 with your choice of framework, since most frameworks rely on a specific router.

[//]: # 'WhyChooseTanStackRouter'

**Does this mean that TanStack Router is a framework?**

TanStack Router itself is not a "framework" in the traditional sense, since it doesn't address a few other common full-stack concerns. However TanStack Router has been designed to be upgradable to a full-stack framework when used in conjunction with other tools that address bundling, deployments, and server-side-specific functionality. This is why we are currently developing [TanStack Start](https://tanstack.com/start), a full-stack framework that is built on top of TanStack Router and Vite.

For a deeper dive on the history of TanStack Router, feel free to read [TanStack Router's History](../decisions-on-dx.md#tanstack-routers-origin-story).

## Why TanStack Router?

TanStack Router delivers on the same fundamental expectations as other routers that you’ve come to expect:

- Nested routes, layout routes, grouped routes
- File-based Routing
- Parallel data loading
- Prefetching
- URL Path Params
- Error Boundaries and Handling
- SSR
- Route Masking

And it also delivers some new features that raise the bar:

- 100% inferred TypeScript support
- Typesafe navigation
- Built-in SWR Caching for loaders
- Designed for client-side data caches (TanStack Query, SWR, etc.)
- Typesafe JSON-first Search Params state management APIs
- Path and Search Parameter Schema Validation
- Search Parameter Navigation APIs
- Custom Search Param parser/serializer support
- Search param middleware
- Inherited Route Context
- Mixed file-based and code-based routing

Let’s dive into some of the more important ones in more detail!

## 100% Inferred TypeScript Support

Everything these days is written “in Typescript” or at the very least offers type definitions that are veneered over runtime functionality, but too few packages in the ecosystem actually design their APIs with TypeScript in mind. So while I’m pleased that your router is auto-completing your option fields and catching a few property/method typos here and there, there is much more to be had.

- TanStack Router is fully aware of all of your routes and their configuration at any given point in your code. This includes the path, path params, search params, context, and any other configuration you’ve provided. Ultimately this means that you can navigate to any route in your app with 100% type safety and confidence that your link or navigate call will succeed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WebMCP-org/webmcp-sh](https://github.com/WebMCP-org/webmcp-sh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
