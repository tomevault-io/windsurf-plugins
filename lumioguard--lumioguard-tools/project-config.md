---
trigger: always_on
description: This file is for anyone changing the code, human or agent. It records the
---

# Working in this repo

This file is for anyone changing the code, human or agent. It records the
decisions that are expensive to rediscover: the invariants, and the failures that
put them there.

It is not the contributor guide. Setup, ports, testing and deploying live in
[CONTRIBUTING.md](CONTRIBUTING.md). What each tool does, and deliberately does
not do, lives in that tool's `README.md`, which is the authority on its
behaviour. Read the relevant one before changing behaviour, and update it in the
same change when behaviour moves.

- **Slopmeter** scores how closely a site's choices match the defaults generated
  sites ship with. See `tools/slopmeter/README.md`.
- **Leakpeek** reports what an app is exposing: keys in the bundle, a database
  with no row-level security, files served from the web root. See
  `tools/leakpeek/README.md`.
- **Citecheck** reports what stops a page being quoted by an answer engine: a
  body that is empty without JavaScript, crawlers turned away, no machine
  readable claims, nothing an answer can be lifted from. See
  `tools/citecheck/README.md`.

## The six that must not break

Everything else in this file is guidance. These six are the ones where breaking
them looks like nothing is wrong.

1. **The console never imports `core`.** That ships a whole detection engine to
   the browser, where anyone can read it. Anything both the engine and the
   surface need is domain vocabulary and belongs in `shared`. Where the surface
   needs a number the scorer owns, such as what a finding cost, the mapper puts
   it on the WIRE rather than the client recomputing it from a second copy of
   the table.
2. **`core` does no I/O.** No `fetch`, no filesystem, no clock. Fetching,
   screenshots and time belong to `api` services. This is what lets the suite run
   offline in seconds.
3. **The rule pack does not cross the wire.** Rule ids, categories and the
   catalogue never appear in a response, and no route returns them.
   `api/__tests__/wire-boundary.test.ts` exists because nothing breaks and no
   screen looks wrong when this leaks.
4. **Leakpeek reads, never writes.** `GET` only, through one request primitive,
   with no code path that writes. A finding that could only be proven by writing
   is reported unverified, never proven. The line between assessing a hole and
   exploiting one is the line between a read and a write.
5. **A report must not become the leak.** Evidence is redacted where it is
   produced: row counts and column names, never values; keys masked. Reports
   render on sites the reader does not own.
6. **The console serves a real document.** `#root` is filled at build time with
   the home page's own words, taken from `src/copy.ts` and the tool catalogue.
   Served empty it is `access.shell`, the blocker Citecheck ships in this repo,
   and the app looks perfect to every human who visits while scoring 40 on
   itself. Never hand-write that copy into `index.html`, and never hide the
   static document from anything that runs JavaScript: serving one thing to a
   browser and another to a crawler is the cloaking these tools report.

## Layout

```
apps/console                    the one front end
  build/                        what a crawler is served, written at build time
  build/pages/                  the prerendered explainers, one document each
  src/copy.ts                   the app's own words, read by the app and the build
  src/pages.ts                  the explainers, registered for the app and the build
  src/tools/catalogue.ts        each tool's id, label and summary as plain data
  src/tools/registry.ts         what a tool must provide to appear
  src/tools/<tool>/             its client, report panels, ink and descriptor
packages/shared                 wire contracts (zod) and shared domain vocabulary
packages/design-tokens          colour, type, radii, elevation, as a Tailwind plugin
packages/ui                     the drawn surface: components, theme, stylesheet
packages/api-core               transport, target resolution, reading hand-off
packages/web-core               browser-side transport and scan state
tools/<tool>/core               detection engine. No I/O
tools/<tool>/api                Cloudflare Worker over hono
```

Every tool is an engine and a Worker, and draws on the shared packages. They do
not depend on each other: a type both engines need belongs in `shared`, never
imported across tools.

Dependencies point one way. The console and `api` depend on `shared`; `api` also
depends on `core`; `core` depends only on `shared`.

**There is one front end, and a tool has none of its own.** Three meters on one
page ask the reader which number is the answer. The console draws one verdict,
the **worst** of the readings that ran, and each tool's own score sits with its
own section in that tool's own words.

A tool is added to the surface by a folder under `apps/console/src/tools/` and a
line in that folder's `index.ts`. Nothing else in the console names a tool, so
the picker, the consolidated score, the hand-off and the report all learn about
it at once. A tool that crawls also hands back its pages, and every crawl is
merged into ONE page list rather than a door each.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lumioguard/lumioguard-tools](https://github.com/lumioguard/lumioguard-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
