---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

This project uses Docker Compose for development environment and Make for common tasks:

### Environment Management
- `make up` - Start KurrentDB and PHP containers (builds if needed)
- `make down` - Stop all containers
- `make logs` - Follow container logs
- `make install` - Install Composer dependencies inside container

### Code Quality and Testing
- `make test` - Run PHPUnit tests with testdox output
- `make test-coverage` - Run tests with coverage report
- `make cs-fixer` - Fix code style using PHP-CS-Fixer
- `make cs-fixer-ci` - Check code style (dry-run with diff)
- `make phpstan` - Run static analysis (level 7)
- `make benchmark` - Run performance benchmarks
- `make before-push` - **Run before pushing commits** - executes cs-fixer, test, and phpstan in sequence to ensure code quality

### Running Individual Commands
All PHP commands are executed inside Docker containers using: `docker compose exec php <command>`

Examples:
- `docker compose exec php bin/phpunit tests/Integration/EventStoreTest.php` - Run specific test file
- `docker compose exec php bin/phpunit --filter test_it_should_read_stream_feed` - Run specific test method
- `docker compose exec php bin/phpstan analyse src/EventStore.php` - Analyze specific file
- `docker compose exec php bin/rector process src/EventStore.php --dry-run` - Preview Rector changes for a file

### Additional Make Targets
- `make rector` - Apply Rector refactoring to the codebase
- `make bash` - Open a bash shell inside the PHP container
- `make check-src-deps` - Verify all src dependencies are declared in composer.json

## Architecture Overview

This is a PHP 8.4+ library that provides a client for KurrentDB (formerly EventStoreDB) HTTP API for event sourcing applications.

### Core Components

**Main Entry Point:**
- `EventStoreFactory` / `EventStoreFactoryInterface` - Creates a fully wired `EventStore` instance; checks connection and assembles Symfony Serializer with all denormalizers
- `EventStore` - Facade implementing `EventStoreInterface` that delegates to three specialized services: `StreamReader`, `StreamWriter`, `StreamIteratorFactory`

**Event Handling:**
- `WritableEvent` - Events to be written to streams
- `WritableEventCollection` - Collections of events for atomic writes
- `StreamFeed\Event` - Events read from streams with metadata
- `WritableToStream` - Interface for objects writable to streams

**Stream Management:**
- `StreamFeed\StreamFeed` - Paginated stream representation with navigation links
- `StreamFeed\StreamFeedIterator` - Forward/backward stream iteration
- `StreamFeed\Entry` - Individual stream entries with event URLs
- `StreamFeed\EntryEmbedMode` - Controls event data embedding (NONE, RICH, BODY)
- `StreamFeed\LinkRelation` - Navigation relations (FIRST, LAST, NEXT, PREVIOUS)

**HTTP Layer:**
- Uses `FriendsOfOuro\Http\Batch\ClientInterface` from `friendsofouro/http-batch-contract`
- PSR-18 compliant HTTP client with batch request support
- `Http\Auth\Credentials` - Authentication handling extracted from URI
- `Http\ResponseCode` - HTTP response code constants
- Supports concurrent batch requests for improved performance

**Value Objects & Utilities:**
- `ValueObjects\Identity\UUID` - UUID handling
- `ExpectedVersion` - Stream version constants (ANY, NO_STREAM, etc.)
- `StreamDeletion` - Deletion modes (SOFT, HARD)

### Key Patterns

1. **Interface Segregation:** Core functionality split into focused interfaces:
   - `StreamReaderInterface` - Stream reading and event reading operations
   - `StreamWriterInterface` - Stream writing and deletion
   - `StreamIteratorFactoryInterface` - Stream iteration factories
   - `EventStoreInterface` - Combines all three interfaces above
2. **PSR Compliance:** Uses PSR-7 (HTTP messages), PSR-18 (HTTP client), and PSR-17 (HTTP factories)
3. **Optimistic Concurrency:** Stream operations include expected version checking
4. **Batch Operations:** Support for reading multiple events efficiently via `ClientInterface::sendBatch()`
5. **Stream Navigation:** AtomPub-style feed navigation with link relations
6. **Error Handling:** Specific exceptions for common scenarios with dedicated handlers
7. **Factory Pattern:** Dedicated factories for creating stream feeds and entries
8. **Serialization:** Symfony Serializer with custom denormalizers (`StreamFeedDenormalizer`, `EntryDenormalizer`, `LinkDenormalizer`, `EventDenormalizer`) and `WritableEventNormalizer` for serializing outbound events

### Testing Environment

- Uses Docker Compose with KurrentDB container for integration tests
- PHPUnit configuration in `phpunit.xml.dist` with strict settings
- Test suites: `unit` (tests/Unit) and `integration` (tests/Integration)
- Environment variable: `EVENTSTORE_URI=http://admin:changeit@eventstore.db:2113` (internal Docker network)
- Tests namespace: `KurrentDB\Tests\`
- Base test case: `tests/Integration/TestCase.php` for integration tests
- Xdebug enabled in Docker for coverage reports

### Code Standards

- PHP-CS-Fixer with Symfony and PSR-12 rules
- PHPStan static analysis at level 7 (strict mode)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FriendsOfOuro/kurrentdb-php-core](https://github.com/FriendsOfOuro/kurrentdb-php-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
