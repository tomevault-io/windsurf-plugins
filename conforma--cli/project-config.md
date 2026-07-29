---
trigger: always_on
description: The Enterprise Contract CLI uses a flexible rule filtering system that allows you to filter Rego rules based on various criteria before evaluation. The system is designed to be extensible and composable, making it easy to add new filtering criteria.
---

# Pluggable Rule Filtering System

## Overview
The Enterprise Contract CLI uses a flexible rule filtering system that allows you to filter Rego rules based on various criteria before evaluation. The system is designed to be extensible and composable, making it easy to add new filtering criteria.

## Architecture

### Core Components
- **`PolicyResolver` interface**: Provides comprehensive policy resolution capabilities for both pre and post-evaluation filtering
- **`PostEvaluationFilter` interface**: Handles post-evaluation filtering and result categorization
- **`UnifiedPostEvaluationFilter`**: Implements unified filtering logic using the same PolicyResolver
- **Individual filter implementations**: Each filter implements the `RuleFilter` interface (legacy support)

### Current Filters
- **`PipelineIntentionFilter`**: Filters rules based on `pipeline_intention` metadata
- **`IncludeListFilter`**: Filters rules based on include/exclude configuration (collections, packages, rules)

## Interface Definitions

```go
// PolicyResolver provides comprehensive policy resolution capabilities.
// It handles both pre-evaluation filtering (namespace selection) and
// post-evaluation filtering (result inclusion/exclusion).
type PolicyResolver interface {
    // ResolvePolicy determines which packages and rules should be included
    // based on the current policy configuration.
    ResolvePolicy(rules policyRules, target string) PolicyResolutionResult
    
    // Includes returns the include criteria used by this policy resolver
    Includes() *Criteria
    
    // Excludes returns the exclude criteria used by this policy resolver
    Excludes() *Criteria
}

// PostEvaluationFilter decides whether individual results (warnings, failures,
// exceptions, skipped, successes) should be included in the final output.
type PostEvaluationFilter interface {
    // FilterResults processes all result types and returns the filtered results
    // along with updated missing includes tracking.
    FilterResults(
        results []Result,
        rules policyRules,
        target string,
        missingIncludes map[string]bool,
        effectiveTime time.Time,
    ) ([]Result, map[string]bool)

    // CategorizeResults takes filtered results and categorizes them by type
    // (warnings, failures, exceptions, skipped) with appropriate severity logic.
    CategorizeResults(
        filteredResults []Result,
        originalResult Outcome,
        effectiveTime time.Time,
    ) (warnings []Result, failures []Result, exceptions []Result, skipped []Result)
}

// RuleFilter decides whether an entire package (namespace) should be
// included in the evaluation set (legacy interface for backward compatibility).
type RuleFilter interface {
    Include(pkg string, rules []rule.Info) bool
}
```

## Current Implementation

### PolicyResolver Types

The system provides two main PolicyResolver implementations:

#### ECPolicyResolver
Uses the full Enterprise Contract policy resolution logic including pipeline intention filtering:

```go
type ECPolicyResolver struct {
    basePolicyResolver
    pipelineIntentions []string
    source             ecc.Source
    config             ConfigProvider
}

func NewECPolicyResolver(source ecc.Source, p ConfigProvider) PolicyResolver {
    intentions := extractStringArrayFromRuleData(source, "pipeline_intention")
    return &ECPolicyResolver{
        basePolicyResolver: basePolicyResolver{
            include: extractIncludeCriteria(source, p),
            exclude: extractExcludeCriteria(source, p),
        },
        pipelineIntentions: intentions,
        source:             source,
        config:             p,
    }
}
```

#### IncludeExcludePolicyResolver
Uses only include/exclude criteria without pipeline intention filtering:

```go
type IncludeExcludePolicyResolver struct {
    basePolicyResolver
}

func NewIncludeExcludePolicyResolver(source ecc.Source, p ConfigProvider) PolicyResolver {
    return &IncludeExcludePolicyResolver{
        basePolicyResolver: basePolicyResolver{
            include: extractIncludeCriteria(source, p),
            exclude: extractExcludeCriteria(source, p),
        },
    }
}
```

### Integration with Conftest Evaluator

The filtering is integrated into the `Evaluate` method in `conftest_evaluator.go`:

```go
func (c conftestEvaluator) Evaluate(ctx context.Context, target EvaluationTarget) ([]Outcome, error) {
    // ... existing code ...

    // Use unified policy resolution for pre-evaluation filtering
    var filteredNamespaces []string
    if c.policyResolver != nil {
        // Use the same PolicyResolver for both pre-evaluation and post-evaluation filtering
        // This ensures consistent logic and eliminates duplication
        policyResolution := c.policyResolver.ResolvePolicy(allRules, target.Target)

        // Extract included package names for conftest evaluation
        for pkg := range policyResolution.IncludedPackages {
            filteredNamespaces = append(filteredNamespaces, pkg)
        }
    }

    // ... conftest runner setup ...


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [conforma/cli](https://github.com/conforma/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
