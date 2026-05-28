---
trigger: always_on
description: 1. **Example Format**: The `example` field must start with `/` and be a working RSSHub route path (e.g., `/example/route`), not a full URL or source website URL.
---

## Review guidelines

### Route Configuration

1. **Example Format**: The `example` field must start with `/` and be a working RSSHub route path (e.g., `/example/route`), not a full URL or source website URL.

2. **Route Name**: Do NOT repeat the namespace name in the route name. The namespace is already defined in `namespace.ts`.

3. **Radar Source Format**: Use relative paths without `https://` prefix in `radar[]. source`. Example: `source: ['www.example.com/path']` instead of `source: ['https://www.example.com/path']`.

4. **Radar Target**: The `radar[].target` must match the route path. If the source URL does not contain a path parameter, do not include it in the target.

5. **Namespace URL**: In `namespace.ts`, the `url` field should NOT include the `https://` protocol prefix.

6. **Single Category**: Provide only ONE category in the `categories` array, not multiple.

7. **Unnecessary Files**: Do NOT create separate `README.md` or `radar.ts` files. Put descriptions in `Route['description']` and radar rules in `Route['radar']`.

8. **Legacy Router**: Do NOT add routes to `lib/router.js` - this file is deprecated.

9. **Features Accuracy**: Set `requirePuppeteer: true` only if your route actually uses Puppeteer. Do not mismatch feature flags.

10. **Maintainer GitHub ID**: The `maintainers` field must contain valid GitHub usernames. Verify that the username exists before adding it.

### Code Style

11. **Naming Convention**: Use `camelCase` for variable names in JavaScript/TypeScript. Avoid `snake_case` (e.g., use `videoUrl` instead of `video_url`).

12. **Type Imports**: Use `import type { ... }` for type-only imports instead of `import { ... }`.

13. **Import Sorting**: Keep imports sorted. Run autofix if linter reports import order issues.

14. **Unnecessary Template Literals**: Do not use template literals when simple strings suffice (e.g., use `'plain string'` instead of `` `plain string` ``).

15. **Avoid Loading HTML Twice**: Do not call `load()` from cheerio multiple times on the same content. Reuse the initial `$` object.

16. **Async/Await in Close**: When closing Puppeteer pages/browsers, use `await page.close()` and `await browser.close()` instead of non-awaited calls.

17. **No Explicit Null**: No need to explicitly set a property to `null` if it does not exist - just omit it.

18. **Valid Item Properties**: Only use properties defined in [lib/types. ts](https://github.com/DIYgod/RSSHub/blob/master/lib/types.ts). Custom properties like `avatar`, `bio` will be ignored by RSSHub.

19. **String Methods**: Use `startsWith()` instead of `includes()` when checking if a string begins with a specific prefix.

20. **Simplify Code**: Combine multiple conditional assignments into single expressions using `||` or `??` operators when appropriate.

### Data Handling

21. **Use Cache**: Always [cache](https://docs.rsshub.app/joinus/advanced/use-cache) the returned results when fetching article details in a loop using `cache.tryGet()`.

22. **Description Content**: The `description` field should contain ONLY the main article content. Do NOT include `title`, `author`, `pubDate`, or tags in `description` - they have their own dedicated fields.

23. **Category Field**: Extract tags/categories from articles and place them in the `category` field, not in `description`.

24. **pubDate Field**: Always include `pubDate` when the source provides date/time information. Use the `parseDate` utility function.

25. **No Fake Dates**: Do NOT use `new Date()` as a fallback for `pubDate`. If no date is available, leave it undefined. See [No Date documentation](https://docs.rsshub.app/joinus/advanced/pub-date#no-date).

26. **No Title Trimming**: Do not manually trim or truncate titles. RSSHub core handles title processing automatically.

27. **Unique Links**: Ensure each item's `link` is unique as it will be used as `guid`. Avoid fallback URLs that could cause duplicate `guid` values.

28. **Human-Readable Links**: The feed `link` field should point to a human-readable webpage URL, NOT an API endpoint URL.

### API and Data Fetching

29. **Prefer APIs Over Scraping**: When the target website has an API (often found by scrolling pages or checking network requests), use the API endpoint instead of HTML scraping.

30. **JSON Parsing**: When using `ofetch`, `JSON.parse` is automatically applied. Do not manually decode JSON escape sequences like `\u003C`.

31. **No Page Turning**: RSS feeds should only request the first page of content. Do not implement pagination parameters for users.

32. **Use Common Parameters**: Use RSSHub's built-in common parameters like [`limit`](https://docs.rsshub.app/guide/parameters#limit-entries) instead of implementing custom query parameters for limiting entries.

33. **No Custom Query Parameters**: Avoid using querystring parameters for route configuration. Use path parameters (`:param`) instead.

34. **No Custom Filtering**: Do not implement custom tag/category filtering in routes. Users can apply filtering using [common parameters](https://docs.rsshub.app/guide/parameters).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Qarzi-man/rsshub-news](https://github.com/Qarzi-man/rsshub-news) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
