---
trigger: always_on
description: Runtime analysis bundle for Doctrine ORM, integrated into Symfony Web Profiler.
---

# Doctrine Doctor

Runtime analysis bundle for Doctrine ORM, integrated into Symfony Web Profiler.
90+ analyzers across 4 categories: Performance, Security, Integrity, Configuration.

- Namespace: `AhmedBhs\DoctrineDoctor`
- PHP 8.4+ / Symfony 6-8 / Doctrine ORM 3-4

## Commands

```bash
# Tests
make test                    # PHPUnit
composer test:unit           # unit tests only
make coverage                # HTML report in coverage/

# Quality
composer ecs                 # code style check
composer ecs:fix             # auto-fix code style
make phpstan                 # PHPStan level 8 (bleeding edge)
make rector                  # Rector dry-run
composer phpmd               # mess detection
composer deptrac             # architectural constraints
composer lint                # syntax check (parallel-lint)
composer markdown-lint       # docs markdown lint

# Combined
composer check               # lint + test + ecs + phpstan + phpmd + rector + deptrac
composer cs:fix              # ecs:fix + rector:fix
make qa                      # alias for check
```

## Code conventions

- `declare(strict_types=1)` required
- License header on every file (enforced by ECS)
- PSR-12, Yoda comparisons (`null === $value`)
- Trailing commas in multiline arrays, arguments, parameters
- `self::` in PHPUnit (not `static::`)
- Test names in snake_case, prefixed with `it_` (e.g. `it_detects_n_plus_one_queries`)
- PHPStan level 8, bleeding edge, `checkUninitializedProperties: true`
- Imports ordered: const, class, function (alphabetical)
- No `final` on classes
- Constructor promotion with `readonly`
- Concatenation with spaces (`$a . $b`, not `$a.$b`)

## Architecture (Deptrac)

```text
Domain        : ValueObject, DTO, Issue, Suggestion, Collection
Application   : Analyzer, Detector, Helper
Infrastructure: Factory, Strategy, Collector, Template, Generator, Utils, Service
Presentation  : Command
Configuration : DependencyInjection, Bundle
```

Dependencies only point inward: Domain imports nothing from Application/Infrastructure.

### Analyzer pattern (Strategy)

Two contracts:

- **`AnalyzerInterface`**: for query-based analyzers that need `QueryDataCollection`
- **`MetadataAnalyzerInterface`** (extends `AnalyzerInterface`): for metadata-based analyzers (Integrity, Configuration, Security) that work on Doctrine metadata or database connections. Uses `MetadataAnalyzerTrait` to bridge `analyze()` -> `analyzeMetadata()`.

```php
// Query analyzers
public function analyze(QueryDataCollection $queryDataCollection): IssueCollection;

// Metadata analyzers
public function analyzeMetadata(): IssueCollection; // via MetadataAnalyzerInterface + trait
```

- Auto-discovered via DI tag `doctrine_doctor.analyzer`
- Located in `src/Analyzer/{Performance,Security,Integrity,Configuration}/`
- Receive a decorated `EntityManager` (vendor entity filtering is transparent)
- Create Issues directly (`new IntegrityIssue(...)`, `new PerformanceIssue(...)`)
- Use config objects as default parameters instead of boolean flags (PHPMD compliance)
- Always return `IssueCollection::fromGenerator(fn() => yield ...)` for lazy evaluation

### Typical analyzer query pipeline

```php
$selects = $queryDataCollection->onlySelects();
$grouped = $selects->groupByNormalizedSql();
foreach ($grouped as $group) {
    if ($group->count() >= $this->threshold) {
        yield new PerformanceIssue(new IssueData(...));
    }
}
```

Key `QueryDataCollection` methods: `onlySelects()`, `onlyInserts()`, `groupByPattern()`, `groupByNormalizedSql()`, `excludePaths()`.

### Suggestion templates

- PHP files in `src/Template/Suggestions/{Category}/`
- Rendered via `PhpTemplateRenderer` (implements `TemplateRendererInterface`)
- Excluded from all checks (ECS, PHPStan, Deptrac, PHPMD)
- Must handle missing context keys gracefully
- Validated by `bin/validate-suggestion-templates.php` in CI:
  - Must use `ob_start()` / `ob_get_clean()`
  - Must return `['code' => $html, 'description' => $text]`
  - Must use `htmlspecialchars()` for output
  - No markdown syntax, no `eval()`

## Adding an analyzer

1. Create class in `src/Analyzer/{Category}/`
2. Implement `AnalyzerInterface` (query-based) or `MetadataAnalyzerInterface` (metadata-based)
3. Create suggestion template in `src/Template/Suggestions/{Category}/`
4. Register in `config/services.yaml` with tag `doctrine_doctor.analyzer`
5. Add tests in `tests/Unit/Analyzer/` or `tests/Analyzer/`
6. See `CONTRIBUTING_ANALYZER.md` for the full guide

Service registration — autowiring works if the analyzer only needs standard interfaces. For custom thresholds or services, declare arguments explicitly:

```yaml
AhmedBhs\DoctrineDoctor\Analyzer\Performance\YourAnalyzer:
    arguments:
        $threshold: '%doctrine_doctor.analyzers.your_analyzer.threshold%'
    tags:
        - { name: doctrine_doctor.analyzer }
```

## Domain model

### ValueObjects

- `Severity`: 5-level enum (info, low, medium, high, critical) with `getPriority()`, `isHigherThan()`
- `SuggestionType`: performance(), security(), integrity(), configuration(), bestPractice(), refactoring()
- `QueryExecutionTime`: factory methods `fromSeconds()`, `fromMilliseconds()`
- `IssueType`: enum mapping all issue types

### Issues


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ahmed-bhs/doctrine-doctor](https://github.com/ahmed-bhs/doctrine-doctor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
