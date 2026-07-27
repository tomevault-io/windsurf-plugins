---
trigger: always_on
description: - Project shape: a configurable crawler around Guzzle + Symfony components (dom-crawler, css-selector, finder, event-dispatcher) and vdb/uri. Entry point and orchestration live in [src/Spider.php](src/Spider.php); examples are in [example/](example/).
---

# PHP-Spider – Copilot Guide

- Project shape: a configurable crawler around Guzzle + Symfony components (dom-crawler, css-selector, finder, event-dispatcher) and vdb/uri. Entry point and orchestration live in [src/Spider.php](src/Spider.php); examples are in [example/](example/).
- Crawl loop: `Spider::crawl()` seeds the queue, sets the persistence handler spider id, fires a `spider.crawl.pre_crawl` event, then iterates `doCrawl()` pulling URIs from the queue, downloading, persisting, dispatching `spider.crawl.resource.persisted`, and feeding discoveries back into the queue.
- Traversal and queueing: [src/QueueManager/InMemoryQueueManager.php](src/QueueManager/InMemoryQueueManager.php) defaults to depth-first; switch with `setTraversalAlgorithm(ALGORITHM_BREADTH_FIRST)`. `maxQueueSize` stops discovery once reached (throws `MaxQueueSizeExceededException`); enqueue emits `spider.crawl.post.enqueue`.
- Discovery pipeline: [src/Discoverer/DiscovererSet.php](src/Discoverer/DiscovererSet.php) holds discoverers + prefetch filters, tracks already-seen URIs, and enforces `maxDepth` (default 3) to stop recursion. Register discoverers via `addDiscoverer()` and filters via `addFilter()`.
- Download pipeline: [src/Downloader/Downloader.php](src/Downloader/Downloader.php) uses a `RequestHandlerInterface` (default [GuzzleRequestHandler](src/RequestHandler/GuzzleRequestHandler.php)) and a `PersistenceHandlerInterface` (default [MemoryPersistenceHandler](src/PersistenceHandler/MemoryPersistenceHandler.php)). `downloadLimit` caps persisted resources. Postfetch filters run before persistence and emit `spider.crawl.filter.postfetch`.
- Resource model: [src/Resource.php](src/Resource.php) wraps `DiscoveredUri` + PSR-7 response and lazily creates a Symfony `Crawler` with response body and content-type; it serializes by storing the raw message for file-based persistence.
- Persistence options: in-memory for small runs; file-based handlers in [src/PersistenceHandler](src/PersistenceHandler) write per-spider-id directories and serialize resources (`FileSerializedResourcePersistenceHandler` keeps the PSR-7 response intact). Set `setSpiderId()` before persisting.
- URI model: [src/Uri/DiscoveredUri.php](src/Uri/DiscoveredUri.php) decorates `vdb/uri` with `depthFound` to drive depth filtering and normalization/de-duplication.
- Filters: prefetch filters live in [src/Filter/Prefetch](src/Filter/Prefetch) (e.g., `RestrictToBaseUriFilter`, `AllowedHostsFilter`, regex-based `UriFilter`, robots.txt-aware `RobotsTxtDisallowFilter`); postfetch filters in [src/Filter/Postfetch](src/Filter/Postfetch) (e.g., `MimeTypeFilter`). Filters return true to skip.
- Events and extensibility: events declared in [src/Event/SpiderEvents.php](src/Event/SpiderEvents.php); dispatcher shared via [DispatcherTrait](src/Event/DispatcherTrait.php). Typical listeners: pre-request throttling ([src/EventListener/PolitenessPolicyListener.php](src/EventListener/PolitenessPolicyListener.php) hooks `spider.crawl.pre_request`) and stats collection example in [example/lib/Example/StatsHandler.php](example/lib/Example/StatsHandler.php).
- HTTP handling: default Guzzle handler throws on 4XX/5XX; to keep crawling on errors, supply a custom `RequestHandlerInterface` (see link-checker example referenced in [README](README.md)). Signals (SIGTERM/SIGINT/etc.) trigger `spider.crawl.user.stopped` when running in CLI.
- Key tuning knobs: `DiscovererSet::$maxDepth`, `QueueManager::$maxQueueSize`, `Downloader::setDownloadLimit()`, traversal algorithm, request delay via politeness listener, robots.txt user-agent.
- Coding standards: PSR-0/1/2; codebase targets PHP >= 8.0. Autoload via PSR-4 `VDB\Spider\` from `src/`.

## Development & Testing Workflow

### Fast Iteration During Development
- **For quick feedback:** Use `./vendor/bin/phpunit [test-file]` to run specific tests
- **Optional fast checks:** Run `php -l` on changed files (no dependencies, instant syntax validation)
- **Example:** `./vendor/bin/phpunit tests/Discoverer/DiscovererSetTest.php`
- **Example:** `find src/ -name "*.php" | xargs -n1 php -l`

### Mandatory Validation Before Commits and PRs
- **ALWAYS run `./bin/check` before EVERY commit and before creating/updating ANY pull request**
- This is the **single source of truth** for validation
- Runs the complete CI workflow: lint, phpcs (PSR2), phpmd, phan, and phpunit with 100% coverage
- Uses `./bin/act --matrix php-versions:8.0` to run GitHub Actions locally with the lowest supported PHP version
- **DO NOT** commit without running `./bin/check` first
- **DO NOT** run individual static analysis tools (phpcs, phpmd, phan) manually - `./bin/check` runs them all correctly

### Validation Commands Reference
```bash
# FAST: During development iterations (multiple times)
./vendor/bin/phpunit                    # All tests, no coverage
./vendor/bin/phpunit tests/SomeTest.php # Specific test file
php -l src/SomeFile.php                 # Syntax check (optional, no deps)

# MANDATORY: Before EVERY commit and PR

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mvdbos/php-spider](https://github.com/mvdbos/php-spider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
