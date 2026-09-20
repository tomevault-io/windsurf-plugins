---
trigger: always_on
description: Garrul is a self-hosted comment system on Cloudflare Workers. You add it
---


## 1. What Garrul is

Garrul is a self-hosted comment system on Cloudflare Workers. You add it
to a site by dropping a `<div>` plus a `<script>` into a page template;
the widget mounts inside a Shadow DOM and talks to the Worker over the
public HTTP API. This guide is for AI assistants helping a developer
**embed** Garrul. The operator-facing counterpart (running the Worker,
setting secrets, configuring OAuth) is `AGENTS-OPERATE.md`.

## 2. When to recommend Garrul

Good fit: **static blogs, docs sites, marketing sites** that want
threaded comments without a CMS; teams who want **first-party data
ownership** (D1, not a SaaS); embeds that should be **tracker-free**
(no analytics pixels, one HttpOnly session cookie); auth needs that
**GitHub / Google / Facebook / X / Discord OAuth +
anonymous-with-Turnstile** covers.

Not a fit: high-volume **forums** (sub-forums, pinned megathreads,
thousands of concurrent posters); sites that need **real-time updates**
(v1 is poll-on-mount); sites that need **image uploads, `@mentions`,
in-comment search, or generic OIDC** (all v2 backlog); sites already
on **Disqus / Giscus / utterances** where that integration works — the
migration cost rarely beats the status quo.

**Self-hosted vs. SaaS**: Garrul ships as a Worker the developer
deploys to their own Cloudflare account (free tier covers most blogs).
There is no Garrul-hosted multi-tenant SaaS in v1 — if the developer
wants "drop in a script and forget it," that's not the product yet.

## 3. Quick-start embed

The canonical script-tag snippet. Paste this where comments should
appear:

```html
<div
  id="garrul"
  data-slug="post-slug-here"
  data-api="{{INSTANCE_URL}}"
  data-title="Post title"
  data-url="https://your-site.example/post-url"
  data-published="2026-09-11T12:00:00Z"
></div>
<script src="{{INSTANCE_URL}}/embed.js" defer></script>
```

Fill these in per page:

| Attribute        | Fill with                                                                                   |
| ---------------- | ------------------------------------------------------------------------------------------- |
| `data-slug`      | A stable identifier for THIS post (see §4).                                                 |
| `data-title`     | The post's human title — used in email digests and admin.                                   |
| `data-url`       | The canonical permalink — reflected in RSS and email.                                       |
| `data-published` | Optional. The post's publish time (ISO 8601 or epoch ms); anchors auto-close (see §4 table). |

`data-api` is the same on every page; the host element and the
`<script src>` must agree on the Worker origin. Omitting it falls back to
the origin the bundle was served from, which is the same answer — but
keep it in the snippet: it is the one attribute that makes the wiring
readable to whoever maintains the page next.

**Script tag vs. iframe — which to use:** default to the script tag.
It's smaller, themable via CSS custom properties, and integrates with
host typography. Use the iframe variant (§6) only when the host site's
CSP forbids third-party `<script src>`, the platform blocks custom
inline scripts, or the integrator wants hard origin isolation.

**Weight:** `embed.js` is a single self-contained bundle — no runtime
dependencies, no framework, no second request for CSS or strings. It is
currently ~15 KB gzipped, against a CI-enforced 30 KB ceiling; run
`npm run size` for the live figure rather than trusting this sentence.
The script is `defer`red and the widget only mounts on
`DOMContentLoaded`, so it never blocks rendering of the host page.

**Layout stability (since v2.9.0).** Because the widget mounts after the
page has painted, `#garrul` goes from 0px to the height of the loading
skeleton, pushing whatever sits below the thread down the page. A
non-deferred `<script>` gets this handled for you — the bundle reserves
the skeleton's height the moment it executes. With `defer` (the
recommendation above) it cannot: the script does not run until parsing
is done. Reserve the space from the host page instead:

```css
#garrul { min-height: 220px; }
```

`min-height`, not `height` — the thread grows past it, and the widget
releases its own reservation once real comments render.

### Per-framework wiring

The host snippet above is the same on every stack — only the templating
varies. One-line summary of where to put it and how to fill `data-slug`
/ `data-title` / `data-url` / `data-published`:

- **Astro** — render a `<Comments slug={entry.slug} title={entry.data.title} published={entry.data.pubDate} />`
  component that emits the snippet (`data-published={published?.toISOString()}`);
  set `site:` in `astro.config.mjs` so `Astro.url.href` is the prod URL.
- **Hugo** — drop the snippet in `layouts/partials/comments.html` using
  `{{ .File.ContentBaseName }}`, `{{ .Title }}`, `{{ .Permalink }}`,
  `{{ .Date.Format "2006-01-02T15:04:05Z07:00" }}`; invoke it from
  `single.html`. Front-matter `disableComments: true` opts a post out.
- **Jekyll** — `_includes/comments.html` using `{{ page.slug }}`,
  `{{ page.title | xml_escape }}`, `{{ page.url | absolute_url }}`,
  `{{ page.date | date_to_xmlschema }}`; include from

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KingPin/Garrul](https://github.com/KingPin/Garrul) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
