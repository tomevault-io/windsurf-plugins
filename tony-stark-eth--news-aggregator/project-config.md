---
trigger: always_on
description: News Aggregator — self-hosted, AI-enhanced RSS/Atom aggregator. Symfony 8.0 + FrankenPHP + PostgreSQL.
---

# CLAUDE.md

## Project

News Aggregator — self-hosted, AI-enhanced RSS/Atom aggregator. Symfony 8.0 + FrankenPHP + PostgreSQL.

See `PITCH.md` for full project overview.

## Planning Archive

Initial development planning files (task plan, progress log, research findings) are archived in `docs/archive/`. These document the full build-out from Phase 1-13 and are kept for historical reference.

## Quick Start

```bash
make up          # Start containers
make quality     # Run all quality checks
make test        # Run all tests
make hooks       # Install git hooks
```

## All Make Targets

### Docker
| Target | Description |
|--------|-------------|
| `make build` | Build Docker images (no cache) |
| `make up` | Start containers (detached, wait for healthy) |
| `make down` | Stop and remove containers |
| `make start` | Build + start |
| `make restart` | Down + up |
| `make logs` | Follow all container logs |
| `make sh` | Shell into PHP container |
| `make worker-logs` | Follow Messenger worker logs |

### Symfony
| Target | Description |
|--------|-------------|
| `make sf c="<cmd>"` | Run any bin/console command |
| `make cc` | Clear Symfony cache |
| `make sf-migrate` | Run Doctrine migrations |

### Code Quality
| Target | Description |
|--------|-------------|
| `make quality` | Run all quality checks (ECS + PHPStan + Rector) |
| `make phpstan` | PHPStan static analysis (level max) |
| `make ecs` | ECS coding standards check |
| `make ecs-fix` | Fix ECS coding standards issues |
| `make rector` | Rector dry-run |
| `make rector-fix` | Apply Rector fixes |

### Testing
| Target | Description |
|--------|-------------|
| `make test` | Run all PHPUnit tests |
| `make test-unit` | Run unit tests only |
| `make test-integration` | Run integration tests only |
| `make infection` | Mutation testing (unit suite, 80/90% MSI) |
| `make coverage` | Generate HTML coverage report |

### TypeScript
| Target | Description |
|--------|-------------|
| `make ts-build` | Compile TypeScript via Bun |
| `make ts-watch` | Watch and compile TypeScript |

### Database
| Target | Description |
|--------|-------------|
| `make db-create` | Create database |
| `make db-drop` | Drop database |
| `make db-reset` | Drop + create + migrate |
| `make export-postgres` | Dump PostgreSQL to `backup/postgres_backup.sql` |
| `make import-postgres` | Restore from backup |

### Git
| Target | Description |
|--------|-------------|
| `make hooks` | Install git hooks from `.githooks/` |

## Domain Overview

```
src/
├── Article/         # Core: articles, scoring, deduplication, content fingerprinting
│   ├── Repository/  # ArticleRepositoryInterface + Doctrine implementation
│   ├── Mercure/     # MercurePublisherService (real + null impl), ArticleCreatedMercureSubscriber
│   ├── Message/     # EnrichArticleMessage, FetchFullTextMessage, RescoreArticlesMessage
│   ├── Service/     # ReadabilityExtractorService, ArticleContentFetcherService, DomainRateLimiterService
│   └── MessageHandler/ # FetchSourceHandler (Phase 1), FetchFullTextHandler (Phase 1.5), EnrichArticleHandler (Phase 2)
├── Enrichment/      # Rule-based + AI categorization/summarization/keywords/translation/sentiment (decorator pattern)
├── Source/          # Feed management, fetching (laminas-feed), health tracking
│   └── Repository/  # SourceRepositoryInterface + Doctrine implementation
├── Notification/    # Unified alert rules (keyword/AI/both) + Notifier dispatch
│   └── Repository/  # AlertRuleRepositoryInterface, NotificationLogRepositoryInterface
├── Digest/          # Periodic AI-generated editorial summaries
│   └── Repository/  # DigestConfigRepositoryInterface, DigestLogRepositoryInterface
├── Chat/            # Conversational RAG assistant (symfony/ai-agent + pgvector)
│   ├── Controller/  # ChatController, ChatHistoryController
│   ├── Service/     # ArticleChatService (agent orchestration), EmbeddingService, ArticleContextFormatter
│   ├── Store/       # ConversationMessageStore (DBAL-backed multi-turn persistence)
│   ├── Tool/        # ArticleSearchTool (hybrid semantic + keyword search, #[AsTool])
│   └── ValueObject/ # ChatResponse
├── User/            # Auth (symfony/security-bundle), per-user read state (UserArticleRead), bookmarks (UserArticleBookmark)
│   ├── Controller/  # ToggleBookmarkController, MarkAllReadController
│   └── Repository/  # UserRepositoryInterface, UserArticleReadRepositoryInterface, UserArticleBookmarkRepositoryInterface
└── Shared/
    ├── AI/          # ModelFailoverPlatform, ModelDiscoveryService, ModelQualityTracker
    ├── Search/      # SEAL + Loupe full-text search (zero infrastructure)
    ├── Entity/      # Category (shared lookup), Setting (key-value config overrides)
    ├── Repository/  # CategoryRepositoryInterface, SettingRepositoryInterface + Doctrine implementations
    ├── Service/     # SettingsService (hybrid env-var defaults + DB overrides), QueueDepthService
    ├── ValueObject/ # EnrichmentMethod (cross-domain)
    ├── Scheduler/   # MaintenanceScheduleProvider (daily reindex + cleanup)
    ├── Command/     # app:cleanup, app:search-reindex, app:check-sources, app:process-digests
    ├── Controller/  # DashboardController, HealthController, SettingsController

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tony-stark-eth/news-aggregator](https://github.com/tony-stark-eth/news-aggregator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
