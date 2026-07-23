---
trigger: always_on
description: ReactUse is a collection of React hooks (`@reactuses/core`). The website is built with Astro (`packages/website-astro`).
---

# ReactUse - Project Guidelines

## Project Overview

ReactUse is a collection of React hooks (`@reactuses/core`). The website is built with Astro (`packages/website-astro`).

## Development

```bash
pnpm install      # install dependencies
pnpm lint         # eslint
pnpm test         # vitest
```

## AI Agent System (`.claude/`)

This repo carries a small, Chromium-inspired AI agent system under `.claude/` — sized for a
hooks library, not a 35M-line codebase. Map: [`.claude/README.md`](.claude/README.md).

**Engineering workflow** (apply to any hook/code task):

1. **Understand first.** Read the real source of the files you'll touch — and at least one
   similar existing hook — before writing. Don't infer behavior from names.
2. **Build**, reusing what exists. This codebase has a shared util layer and strong
   conventions; reuse before writing. Browser APIs **must** be SSR-safe.
3. **Test.** Jest, co-located `index.spec.ts` (`pnpm --filter @reactuses/core test <name>`).
4. **Verify.** `pnpm lint` + tests green. Stay on task — no unrelated drive-by edits.

**Where to look:**

- [`.claude/knowledge-base.md`](.claude/knowledge-base.md) — task → file/utility router. **Start here**; it tells you which existing code already solves your task.
- [`.claude/ai-policy.md`](.claude/ai-policy.md) — you own every line you ship; understand it before review.
- Skills (auto-activate): `new-hook`, `hook-test`, `hook-docs`, `pr-description`.
- Commands: `/new-hook`, `/pre-pr`, `/pr-desc`.

## Secrets (`.env`, gitignored)

`.env` holds API tokens — never commit (it is gitignored):

- `DEVTO_API_KEY` — dev.to publishing
- `HASHNODE_PAT` / `HASHNODE_PUBLICATION_ID` / `HASHNODE_PUBLICATION_HOST` — Hashnode publishing
- `NETLIFY_AUTH_TOKEN` — Netlify account PAT (DNS + site API). **reactuse.com DNS is Netlify-managed** (DNS zone under account `childrentime`). Use this token against `https://api.netlify.com/api/v1/dns_zones` to manage records — e.g. the `google-site-verification` TXT that verifies the `sc-domain:reactuse.com` GSC property (a domain property can only be verified by DNS TXT, not the `<meta>` tag in `BaseLayout.astro`).

## SEO / 搜索数据分析

用 Google Search Console 数据做优化分析：`python3 scripts/gsc-report.py [天数]`。
输出概览、趋势、Top 关键词/页面，以及「临门一脚关键词」「高曝光低点击页面」两类优化机会。
配置、用法、优化打法见 `GSC.md`。

## Website URL Structure

Hook documentation pages follow the pattern: `https://reactuse.com/{category}/{hookname}/`

Categories: `browser`, `effect`, `element`, `state`, `integrations`

**There is NO `/docs/` or `/hooks/` prefix.** The URL is derived from the file path under `packages/website-astro/src/content/docs/`.

**URLs are all lowercase.** The file is `useGeolocation.mdx` but the canonical URL is `https://reactuse.com/browser/usegeolocation/` (lowercase). Never link to the camelCase form — it creates duplicate pages in Google's index. A Netlify edge function 301-redirects any camelCase path to lowercase as a safety net, but links should be lowercase at the source.

## Blog Post Guidelines

### Writing blog posts

- Blog posts live in `packages/website-astro/src/content/blog/` (English), with translations in `blog-zh-hans/` and `blog-zh-hant/`.
- The legacy Docusaurus blog is at `packages/website-docusaurus/blog/`.

### Writing blog post titles (SEO — for NEW posts only)

> **Why this exists** (2026-06 growth analysis): blog posts pull huge search **impressions**
> but near-zero **CTR** because they rank #9–17 with weak titles — e.g.
> `react-hooks-vs-vue-composables` had **13.8k impressions but 0.2% CTR**. Meanwhile hook
> **doc** pages that match `useX` queries convert at **9–44% CTR**, and the developers who
> actually star the repo are **English-speaking / international** (only ~7% China). The funnel
> to GitHub stars is: English search → reactuse.com → star. **Do NOT rewrite old posts — apply
> this only to new ones.**

Title rules, in priority order:

1. **Lead with the concrete searched term** — a specific hook name or exact phrase
   (`useDebounce`, `useLocalStorage`, `useIntersectionObserver`) in the first ~60 chars.
   Vague/clever titles earn impressions but no clicks.
2. **Target a keyword you're already near page 1 on.** Before writing, run
   `python3 scripts/gsc-report.py 90` and pick a "临门一脚" query (rank 4–15, impressions ≥50);
   write the post to own it. A great post ranking #9 on a brand-new keyword wins nothing.
3. **Match search intent, not cleverness** — use real query shapes: `How to …`, `… in React`,
   `X vs Y`, a year (`2026`), `… with TypeScript`.
4. **One post = one primary keyword.** The page must read as *the* answer to that one query.
5. **Add a click-earning differentiator** — specificity / the problem solved
   (e.g. `…(SSR-safe, TypeScript)`).
6. **Write English-first.** zh-Hans / zh-Hant are translations; English is the star driver.

Examples:
- ✅ `React useDebounce Hook: Debounce State & Callbacks (2026)`
- ✅ `useLocalStorage in React: SSR-Safe Persistent State`
- ❌ `React Hooks vs Vue Composables` (broad/abstract → 13.8k impressions, 0.2% CTR in reality)
- ❌ `Mastering Reactivity Patterns` (no searchable keyword)

### Hook links in blog posts


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [childrentime/reactuse](https://github.com/childrentime/reactuse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
