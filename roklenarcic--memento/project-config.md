---
trigger: always_on
description: This document provides an analysis of the Memento caching library for AI assistants working on this codebase.
---

# Memento - Caching Library Analysis

This document provides an analysis of the Memento caching library for AI assistants working on this codebase.

## Project Overview

**Memento** is a Clojure function memoization library with advanced features beyond basic caching. It provides:
- Request/scoped caching (temporarily replace caches within a scope)
- Tagged eviction via secondary index (invalidate entries by tag + ID pairs)
- Selective caching (prevent caching of specific return values)
- Tiered/multi-cache support (combine local + upstream caches)
- Shared size-based eviction across multiple functions
- Cache events system
- Variable per-entry expiry

**Current version**: 2.0.68  
**Minimum Java**: 11+  
**Primary dependency**: Caffeine 3.1.8

## Directory Structure

```
memento/
├── src/memento/           # Clojure source files
│   ├── core.clj           # Main public API (307 lines)
│   ├── base.clj           # Core abstractions and protocols (60 lines)
│   ├── config.clj         # Configuration constants (178 lines)
│   ├── mount.clj          # MountPoint implementations (151 lines)
│   ├── caffeine.clj       # Caffeine cache implementation (145 lines)
│   ├── caffeine/config.clj # Caffeine-specific config keys
│   ├── guava.clj          # DEPRECATED - redirects to caffeine
│   ├── guava/config.clj   # DEPRECATED
│   ├── multi.clj          # Tiered cache implementations (47 lines)
│   └── ns_scan.clj        # Namespace scanning utility (47 lines)
│
├── java/memento/          # Java source files (performance-critical code)
│   ├── base/              # Core interfaces
│   │   ├── ICache.java    # Main cache interface
│   │   ├── Segment.java   # Function binding info
│   │   ├── CacheKey.java  # Composite key (id + args)
│   │   ├── EntryMeta.java # Cached value wrapper with metadata
│   │   ├── TagInvalidation.java # Active tag invalidation tracking across caches
│   │   ├── InvalidationClock.java # Global monotonic epoch source
│   │   └── Durations.java
│   ├── mount/             # Mount point implementations
│   │   ├── IMountPoint.java
│   │   ├── Cached.java    # Marker interface for cached fns
│   │   ├── CachedFn.java
│   │   └── CachedMultiFn.java
│   ├── caffeine/          # Caffeine implementation
│   │   ├── CaffeineCache_.java
│   │   ├── Expiry.java
│   │   ├── SecondaryIndex.java
│   │   └── SpecialPromise.java
│   └── multi/             # Multi-cache implementations
│       ├── MultiCache.java
│       ├── TieredCache.java
│       ├── ConsultingCache.java
│       └── DaisyChainCache.java
│
├── test/memento/          # Test files
│   ├── core_test.clj      # Main API tests (429 lines)
│   ├── caffeine_test.clj  # Cache builder tests (58 lines)
│   ├── multi_test.clj     # Multi-cache tests (67 lines)
│   ├── mount_test.clj     # Mount point tests (112 lines)
│   └── ns_scan_test.clj   # Namespace scanning tests (22 lines)
│
├── doc/                   # Documentation markdown files
└── deps.edn               # Project dependencies
```

## Architecture

### Key Concepts

1. **Cache**: An instance of `ICache` that stores key-value pairs. One cache can back multiple functions.

2. **MountPoint**: Connects a function to a cache. Contains:
   - Reference to the cache (or tag for dynamic lookup)
   - Segment information (function, key-fn, id, config)
   - Event handler

3. **Segment**: Metadata about a memoized function binding:
   - `f` - The original function
   - `keyFn` - Key transformation function
   - `id` - Identifier (typically var name)
   - `conf` - Mount configuration

4. **CacheKey**: Composite key used in cache storage:
   - `id` - Segment identifier
   - `args` - Transformed function arguments

5. **Tags**: Enable scoped caching and bulk operations:
   - Functions can have multiple tags
   - `with-caches` temporarily replaces cache for a tag
   - `memo-clear-tag!` invalidates by tag + ID

### Namespace Responsibilities

| Namespace | Purpose |
|-----------|---------|
| `memento.core` | Public API - all user-facing functions |
| `memento.base` | Core abstractions, `ICache` wrapper fns, `no-cache`, `new-cache` multimethod |
| `memento.config` | Configuration key constants and time units |
| `memento.mount` | `TaggedMountPoint` and `UntaggedMountPoint` records, tag management |
| `memento.caffeine` | `CaffeineCache` record implementing `ICache` |
| `memento.multi` | Registers tiered/consulting/daisy cache types |
| `memento.ns-scan` | Auto-attach caches to annotated vars |

### Java vs Clojure Split

Java is used for performance-critical paths:
- Reduces stack depth for cached calls
- Implements `ICache` and `IMountPoint` interfaces
- Handles concurrent load coordination (`SpecialPromise` and tag invalidation tracking)
- Secondary index for tag-based eviction

Clojure is used for:
- Public API (`memento.core`)
- Configuration and records
- Multimethod dispatch for cache types

## Cache Types

### Primary: Caffeine (`:memento.core/caffeine`)
- High-performance Java caching library
- Supports: size limits, TTL, fade (access-based), weak/soft references, statistics
- Default cache type

### None (`:memento.core/none`)
- No-op cache, doesn't cache anything
- Used when caching is globally disabled

### Multi-Cache Types


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RokLenarcic/memento](https://github.com/RokLenarcic/memento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
