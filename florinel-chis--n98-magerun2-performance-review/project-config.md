---
trigger: always_on
description: This file provides guidance to Claude Code and its sub-agents when working with the Performance Review module for n98-magerun2.
---

# CLAUDE.md - Performance Review Module

This file provides guidance to Claude Code and its sub-agents when working with the Performance Review module for n98-magerun2.

## Quick Reference

### What This Module Does
A comprehensive performance analysis tool for Magento 2 that runs 11 different analyzer categories and generates actionable reports. Think of it as a "health check" for Magento installations.

### Architecture at a Glance

```
Command (PerformanceReviewCommand)
    ↓
Analyzers (11 core + custom)
    ↓
Issue Collection
    ↓
Report Generator
    ↓
Console Output
```

### Key File Locations

| Purpose | Path | Description |
|---------|------|-------------|
| **Main Command** | `src/PerformanceReview/Command/PerformanceReviewCommand.php` | Entry point, orchestrates all analyzers |
| **Analyzer Interface** | `src/PerformanceReview/Api/AnalyzerCheckInterface.php` | Contract for all analyzers (new interface) |
| **Issue Collection** | `src/PerformanceReview/Model/Issue/Collection.php` | Collects issues from analyzers |
| **Issue Builder** | `src/PerformanceReview/Model/Issue/IssueBuilder.php` | Fluent API for creating issues |
| **Report Generator** | `src/PerformanceReview/Model/ReportGenerator.php` | Formats and outputs reports |
| **Module Config** | `n98-magerun2.yaml` | Registers commands and autoloaders |
| **Core Analyzers** | `src/PerformanceReview/Analyzer/*.php` | 11 built-in analyzers |
| **Examples** | `examples/CustomAnalyzers/*.php` | Example custom analyzer implementations |

### Core Interfaces

```php
// Primary interface for new analyzers (v2.0)
interface AnalyzerCheckInterface {
    public function analyze(Collection $results): void;
}

// Optional: For analyzers needing configuration
interface ConfigAwareInterface {
    public function setConfig(array $config): void;
}

// Optional: For analyzers needing Magento dependencies
interface DependencyAwareInterface {
    public function setDependencies(array $dependencies): void;
}
```

### 11 Analyzer Categories

1. **config** - Configuration settings (mode, cache, session)
2. **database** - Database size, tables, URL rewrites
3. **modules** - Third-party modules and impact
4. **codebase** - Code organization, generated content
5. **frontend** - JS/CSS optimization, images
6. **indexing** - Indexers, cron jobs, queues
7. **php** - PHP version, memory, extensions
8. **mysql** - MySQL configuration and optimization
9. **redis** - Redis setup and configuration
10. **api** - API integrations and tokens
11. **thirdparty** - Known problematic extensions

### Example Analyzers (Reference Implementations)

The module includes production-ready example analyzers in `examples/CustomAnalyzers/`:

**UnusedIndexAnalyzer** (GOLD STANDARD REFERENCE)
- **Purpose**: Detects unused database indexes that waste storage and slow down writes
- **Interfaces**: All three (AnalyzerCheckInterface, ConfigAwareInterface, DependencyAwareInterface)
- **Testing**: 21 comprehensive unit tests covering all scenarios
- **Configuration**: Fully configurable thresholds (min_size_mb, high_priority_mb, medium_priority_mb)
- **Error Handling**: Comprehensive exception handling with fallback queries
- **Documentation**: Complete README and setup guide included
- **Status**: Production-ready, professionally coded

**Use UnusedIndexAnalyzer as your reference when creating custom analyzers.** It demonstrates all best practices.

**Files:**
- `examples/CustomAnalyzers/UnusedIndexAnalyzer.php` - Implementation
- `docs/examples/unused-index-analyzer/README.md` - Documentation
- `docs/examples/unused-index-analyzer/setup.md` - Setup guide
- `tests/Unit/Analyzer/UnusedIndexAnalyzerTest.php` - Test suite

**Other Examples:**
- `RedisMemoryAnalyzer.php` - Checks Redis memory usage and fragmentation
- `ElasticsearchHealthAnalyzer.php` - Monitors Elasticsearch cluster health

## Common Tasks

### Task 1: Create a New Custom Analyzer

**When to do this:** User wants to add a new performance check.

**Step-by-step:**

1. **Create the analyzer class:**
   ```php
   <?php
   namespace MyCompany\Analyzer;

   use PerformanceReview\Api\AnalyzerCheckInterface;
   use PerformanceReview\Model\Issue\Collection;

   class MyCustomAnalyzer implements AnalyzerCheckInterface
   {
       public function analyze(Collection $results): void
       {
           // Your analysis logic here
           if ($this->detectIssue()) {
               $results->createIssue()
                   ->setPriority('medium')  // high|medium|low
                   ->setCategory('Custom')
                   ->setIssue('Short description of the issue')
                   ->setDetails('Detailed explanation')
                   ->setCurrentValue('actual state')
                   ->setRecommendedValue('desired state')
                   ->add();
           }
       }

       private function detectIssue(): bool
       {
           // Your detection logic
           return true;
       }
   }
   ```

2. **Register in n98-magerun2.yaml:**
   ```yaml
   # Add autoloader if needed
   autoloaders_psr4:
     MyCompany\Analyzer\: 'path/to/MyCompany/Analyzer'

   # Register the analyzer
   commands:
     PerformanceReview\Command\PerformanceReviewCommand:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [florinel-chis/n98-magerun2-performance-review](https://github.com/florinel-chis/n98-magerun2-performance-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
