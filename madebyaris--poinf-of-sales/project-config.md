---
trigger: always_on
description: Tech debt prevention patterns with consistency enforcement, code quality gates, and architectural governance
---


# 🏗️ Tech Debt Prevention & Code Quality Governance

## 🎯 Zero Tech Debt Philosophy

### Proactive Prevention Strategy
```typescript
// ✅ TECH DEBT PREVENTION: Systematic approach to code quality
namespace TechDebtPrevention {
  // Code quality metrics and thresholds
  interface QualityGates {
    code_coverage: { minimum: 85, target: 90 }
    complexity_score: { maximum: 10, target: 7 }
    duplication: { maximum: 3, target: 1 }
    performance: { api_response: '< 200ms', ui_render: '< 100ms' }
    security: { vulnerabilities: 0, code_quality: 'A' }
  }

  // Automated quality enforcement
  class QualityEnforcer {
    static enforcePreCommitQuality(): PreCommitHook {
      return {
        // Code format and style
        prettier_format: true,
        eslint_validation: true,
        typescript_strict_check: true,
        
        // Business logic validation
        business_rule_consistency: true,
        api_contract_validation: true,
        database_migration_safety: true,
        
        // Performance validation
        bundle_size_check: true,
        query_performance_validation: true,
        memory_leak_detection: true
      }
    }
  }
}
```

## 🔒 Consistency Enforcement Patterns

### 1. Architectural Consistency
```typescript
// ✅ CONSISTENCY: Standardized architectural patterns
class ArchitecturalConsistency {
  // Enforce consistent API patterns
  static createAPIEndpoint<TRequest, TResponse>(
    config: APIEndpointConfig<TRequest, TResponse>
  ): StandardAPIEndpoint<TRequest, TResponse> {
    return {
      // Standardized request validation
      validateRequest: (request: TRequest): ValidationResult => {
        const validator = this.createValidator(config.validation_schema)
        return validator.validate(request)
      },

      // Standardized business logic execution
      executeBusinessLogic: async (request: TRequest): Promise<TResponse> => {
        // Consistent error handling
        try {
          // Standardized logging
          Logger.info(`Executing ${config.endpoint_name}`, { request })
          
          // Business logic with consistent patterns
          const result = await config.business_logic(request)
          
          // Standardized success response
          return {
            success: true,
            message: config.success_message,
            data: result,
            timestamp: new Date().toISOString(),
            request_id: generateRequestId()
          }
        } catch (error) {
          // Standardized error handling
          return this.handleStandardError(error, config.endpoint_name)
        }
      },

      // Standardized response formatting
      formatResponse: (response: TResponse): StandardAPIResponse<TResponse> => {
        return {
          ...response,
          version: config.api_version,
          performance_metrics: this.getPerformanceMetrics()
        }
      }
    }
  }

  // Enforce consistent component patterns
  static createBusinessComponent<TProps>(
    config: ComponentConfig<TProps>
  ): React.FC<TProps> {
    return React.memo((props: TProps) => {
      // Standardized error boundary
      return (
        <ErrorBoundary fallback={config.error_fallback}>
          {/* Standardized loading states */}
          <Suspense fallback={config.loading_fallback}>
            {/* Standardized accessibility */}
            <div 
              role={config.accessibility.role}
              aria-label={config.accessibility.label}
              className={cn(config.base_classes, props.className)}
            >
              {/* Component content with consistent patterns */}
              {config.render(props)}
            </div>
          </Suspense>
        </ErrorBoundary>
      )
    }, config.memo_comparison || shallowEqual)
  }

  // Database query consistency
  static createDatabaseQuery<TParams, TResult>(
    config: QueryConfig<TParams, TResult>
  ): DatabaseQuery<TParams, TResult> {
    return {
      execute: async (params: TParams): Promise<TResult> => {
        // Standardized query performance monitoring
        const startTime = performance.now()
        
        try {
          // Standardized parameter validation
          this.validateQueryParams(params, config.param_schema)
          
          // Standardized query execution
          const result = await this.executeQuery(config.query, params)
          
          // Standardized performance logging
          const duration = performance.now() - startTime
          this.logQueryPerformance(config.name, duration, params)
          
          return result
        } catch (error) {
          // Standardized error handling
          this.handleQueryError(error, config.name, params)
          throw error
        }
      }
    }
  }
}
```

### 2. Code Pattern Enforcement
```typescript
// ✅ PATTERN ENFORCEMENT: Consistent code patterns across the system
class CodePatternEnforcement {
  // Standardized hook patterns
  static createBusinessHook<TData, TError = Error>(
    config: BusinessHookConfig<TData, TError>
  ): BusinessHook<TData, TError> {
    return function useBusinessData() {
      // Consistent state management
      const [state, setState] = useState<BusinessHookState<TData, TError>>({

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/poinf-of-sales](https://github.com/madebyaris/poinf-of-sales) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
