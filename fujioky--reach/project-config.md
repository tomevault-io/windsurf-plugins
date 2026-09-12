---
trigger: always_on
description: - **Build**: `npm run build` (Next.js 16 + Turbopack)
---

# Reach — Project Notes

## Build & Deploy

- **Build**: `npm run build` (Next.js 16 + Turbopack)
- **Deploy to Vercel**: `vercel build --prod && vercel deploy --prebuilt --prod`
- **Node runtime**: several `/api/*` routes require `export const runtime = 'nodejs'` because they rely on Node.js fetch streaming / S3 / PostgreSQL.

## Self-Authored Articles

The site publishes original articles alongside mirrors — see
`docs/article-publishing.md` for the full picture. Key facts for anyone touching
adjacent code:

- Articles live in `content_items` with `type='article'` and are reachable at
  the public `/p/<slug>` URL (no access control, no expiry) rather than the
  mirror's `/s/<token>`. Article-only columns (`slug`, `status`, `excerpt`,
  `cover_image_url`, `updated_at`, `comments_enabled`) are all nullable.
- **Any query over `content_items` now needs a type scope.** Use `isArticle` /
  `isNotArticle` from `lib/article/queries.ts`. `isNotArticle` deliberately
  keeps `type IS NULL` rows — early mirrors predate the column, and a bare
  `ne(type,'article')` silently drops them.
- Visitor comments go to `article_comments`, not `comments` (which stays the
  platform-scraped mirror table).
- The archive lives at `/post`; individual articles stay at `/p/<slug>` (a
  permanent redirect covers the old bare `/p`). Articles with `listed=false`
  are readable at their URL but excluded from the archive — use
  `listPublishedArticles(limit, includeUnlisted)` when that matters.
- Article slugs are random lowercase ASCII. They used to be derived from the
  title, which produced CJK slugs — and Next hands dynamic segments to the page
  **still percent-encoded**, so `测试1` arrived as `%E6%B5%8B%E8%AF%951`, missed
  the lookup, and the article 404'd while still listing on `/p`. `/p/[slug]`
  decodes params for that reason; don't remove it, older articles rely on it.
- Article media is uploaded browser → storage directly, never through a Route
  Handler (Vercel's ~4.5MB body cap). Videos need CORS on the R2/S3 bucket —
  the rule must include `AllowedHeaders: ["content-type"]`, or the preflight is
  rejected and every upload fails in the browser.
- Remote import ("从链接转存") is the only place a user-supplied hostname
  reaches the server's network stack. It must go through
  `lib/article/remote-fetch.ts`, which rejects non-http schemes and any host
  resolving to a private/loopback/link-local address, and re-validates every
  redirect hop. Do not swap it for a plain `fetch` with `redirect: 'follow'` —
  that validates only the first URL. The resolver walks up to 4 HTML pages
  (embedded media URL → meta-refresh/JS redirect → the AI fallback in
  `lib/article/ai-resolver.ts`, off by default); **anything the model returns is
  remote input and goes back through the same validation** before it is fetched.
- The transfer itself lives in `lib/article/remote-import.ts` and reports stages
  through a callback. `/api/article-import` streams those as NDJSON so the
  editor can show per-item progress; the `importRemoteMedia` Server Action calls
  the same function and just awaits the result. A stream that ends without a
  `done` event is a failure, not a success — that is what a Vercel function
  timeout looks like from the browser.
- `/api/article-media` is the single entry point for article media: it decides
  both access (public asset → open; otherwise a signed `?t=` token from the
  rendered page, or an admin session) and delivery (stream vs 302 to a
  presigned storage URL, per `article_media_direct`). Pages emit only that
  path — don't reintroduce URL rewriting on the page side.
- Direct delivery redirects to a **presigned** S3 GET, so the bucket does not
  need public read.
- The 「插入素材」 panel's 素材库 tab lists **every** article asset, not the open
  article's. That is the point: an asset uploaded into another article (or one
  whose article no longer references it) is otherwise unreachable. Nothing in
  the access path is scoped by owning article — `findGatedMediaIds` signs by
  media id and the route checks `shared`/token/session — so cross-article
  references render correctly. Don't add an owner check there without also
  killing this tab.
- Which assets are still in use is derived by scanning article text for
  `/api/article-media/<id>.<ext>`, not from a join table — the author edits
  Markdown freely and no API sees those edits. `lib/article/assets.ts` owns
  that; the cleanup path re-scans before deleting and skips uploads younger
  than 24h.
- Markdown bodies are rendered **without** raw HTML (no `rehype-raw`). Don't add
  it without a reason — it reopens the injection surface.
- Password gate (`lib/content/password.ts`) covers articles and mirrors: per-item
  mode none/inherit/custom, unlock scoped by password hash via an HMAC cookie.
  On mirrors it must run **before** `checkAccess`, which is what consumes a view
  and burns a one-shot link.
- `checkAccess` mutates (view count, burn); `peekAccess` is the read-only twin.
  Anything running per-request before the visitor sees the page — notably
  `generateMetadata` — must use peek, or it spends a view for the title alone.

## Video proxy health check

`probeVideoProxy` (`lib/health/checks.ts`) asks the proxy's own `/healthz` (then

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fujioky/reach](https://github.com/fujioky/reach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
