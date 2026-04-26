---
trigger: always_on
description: **Generated:** 2026-04-16
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-04-16
**Commit:** ddacee2
**Branch:** main

## OVERVIEW

Fluent, immutable, type-safe Elasticsearch query builder for PHP 8.4+. Zero production dependencies. Builds `array` payloads for the official `elasticsearch/elasticsearch` PHP client.

## STRUCTURE

```
./
├── src/
│   ├── QueryBuilder.php          # Main entry point - fluent builder
│   ├── Query/                    # Query classes (TermQuery, BoolQuery, MatchQuery, ...)
│   ├── Aggregation/              # Aggregation classes (TermsAggregation, StatsAggregation, ...)
│   ├── Sort/                     # Sort classes (FieldSort, ScoreSort) + SortDirectionEnum
│   └── Exception/                # Domain-specific exceptions (Aggregation/, Builder/, Query/)
├── tests/
│   ├── TestCase.php              # Base test class
│   ├── IntegrationTestCase.php   # Base for integration tests (ES client setup)
│   ├── Fixture/                  # Reusable test fixtures
│   ├── Unit/                     # Mirrors src/ structure
│   └── Integration/              # Requires running Elasticsearch
├── .agents/skills/               # Agent Skills (agentskills.io spec)
│   ├── create-query/SKILL.md     # Skill: create a new Query class
│   ├── create-aggregation/SKILL.md # Skill: create a new Aggregation class
│   └── create-sort/SKILL.md      # Skill: create a new Sort class
├── .ai/guidelines.md             # AI contribution rules (MUST READ before any changes)
├── scripts/import_dataset.php    # Imports test CSV into Elasticsearch
└── .data/                        # Local Elasticsearch data (Docker volume)
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add new query type | `src/Query/` | Implement `QueryInterface`, use traits (`BoostableQuery`, `AnalyzerAwareQuery`) |
| Add new aggregation | `src/Aggregation/` | Implement `AggregationInterface`, use traits (`FilterableAggregation`, `GlobalizableAggregation`, `SizeableAggregation`) |
| Add new sort type | `src/Sort/` | Implement `SortInterface` |
| Add new exception | `src/Exception/{domain}/` | Extend domain base (`QueryException`, `AggregationException`, `BuilderException`) |
| Understand builder API | `src/QueryBuilder.php` | `query()`, `aggregation()`, `sort()`, `size()`, `from()`, `build()` |
| Understand immutability | Any concrete class | All mutation methods clone `$this` before modifying |
| Create composite query | Extend `CompositeQuery` | Override `query(): QueryInterface` |
| Create composite aggregation | Extend `CompositeAggregation` | Override `aggregation(): AggregationInterface` |
| Unit tests | `tests/Unit/` | Mirrors `src/` directory structure |
| Integration tests | `tests/Integration/` | Needs `ELASTICSEARCH_HOST` env var |
| Trait tests | `tests/Unit/Aggregation/Trait/` | Shared trait behavior tests |

## TYPE HIERARCHY

```
QueryInterface ─┬─ TermQuery (uses BoostableQuery)
                ├─ TermsQuery (uses BoostableQuery)
                ├─ ExistsQuery (uses BoostableQuery)
                ├─ MatchQuery (uses BoostableQuery, AnalyzerAwareQuery)
                ├─ MatchPhraseQuery (uses BoostableQuery, AnalyzerAwareQuery)
                ├─ BoolQuery (uses BoostableQuery)
                ├─ NestedQuery
                ├─ RangeQuery [abstract] (uses BoostableQuery)
                │   ├─ NumericRangeQuery
                │   └─ DatetimeRangeQuery
                └─ CompositeQuery [abstract] ─ user-defined composites

AggregationInterface ─┬─ TermsAggregation (uses FilterableAggregation, GlobalizableAggregation, SizeableAggregation)
                      ├─ StatsAggregation (uses FilterableAggregation, GlobalizableAggregation)
                      ├─ NestedAggregation (uses FilterableAggregation, GlobalizableAggregation)
                      ├─ ContainerAggregation (uses FilterableAggregation, GlobalizableAggregation)
                      ├─ MultiTermsAggregation (uses FilterableAggregation, GlobalizableAggregation, SizeableAggregation)
                      ├─ HistogramAggregation (uses FilterableAggregation, GlobalizableAggregation)
                      ├─ SumAggregation (uses FilterableAggregation, GlobalizableAggregation)
                      ├─ CardinalityAggregation (uses FilterableAggregation, GlobalizableAggregation)
                      ├─ DateHistogramAggregation (uses FilterableAggregation, GlobalizableAggregation)
                      └─ CompositeAggregation [abstract] ─ user-defined composites

SortInterface ─┬─ FieldSort
               └─ ScoreSort

Exception hierarchy:
  QueryException [abstract] ─ EmptyBoolQueryException, EmptyNestedQueryException, EmptyRangeQueryException, EmptyTermsQueryException, InvalidOperatorQueryException, InvalidRelationQueryException
  AggregationException [abstract] ─ DuplicatedContainerAggregationException, DuplicatedNestedAggregationException, InvalidAggregationSizeException, InvalidContainerAggregationException, InvalidDateHistogramIntervalException, InvalidIntervalException, InvalidPrecisionThresholdException, NotEnoughFieldsAggregationException
  BuilderException [abstract] ─ DuplicatedBuilderAggregationException, InvalidFromException, InvalidSizeException
```

## AI CONTRIBUTION RULES (from .ai/guidelines.md)

### Hard Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bonu-dev/php-elasticsearch-builder](https://github.com/bonu-dev/php-elasticsearch-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
