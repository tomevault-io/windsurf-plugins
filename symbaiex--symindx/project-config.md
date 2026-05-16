---
trigger: always_on
description: This rule defines comprehensive patterns for leveraging Cursor's workflow automation capabilities, including background agents, action-based triggers, and multi-agent coordination for complex development tasks.
---

# Cursor Workflow Automation & Action Orchestration

This rule defines comprehensive patterns for leveraging Cursor's workflow automation capabilities, including background agents, action-based triggers, and multi-agent coordination for complex development tasks.

## Core Workflow Architecture

### Workflow Definition Patterns

**Declarative Workflow Files**
```yaml
# .cursor/workflows/ai-portal-validation.yaml
name: "AI Portal Validation Pipeline"
description: "Automated validation of new AI portal configurations"
triggers:
  - type: file_change
    patterns: ["mind-agents/src/portals/*/config.json"]
  - type: manual
    command: "validate-portal"
  - type: schedule
    cron: "0 2 * * *"  # Daily at 2 AM

actions:
  - name: "validate-config"
    agent: "config-validator"
    inputs:
      config_file: "${trigger.file_path}"
    timeout: 300
    
  - name: "test-connection"
    agent: "connection-tester"
    depends_on: ["validate-config"]
    inputs:
      portal_name: "${actions.validate-config.outputs.portal_name}"
    parallel: false
    
  - name: "generate-docs"
    agent: "doc-generator"
    depends_on: ["test-connection"]
    inputs:
      portal_config: "${actions.validate-config.outputs.config}"
      test_results: "${actions.test-connection.outputs.results}"

error_handling:
  retry_attempts: 3
  retry_delay: 30
  fallback: "notify-maintainers"
```

**Workflow Orchestration Rules**
```typescript
// .cursor/workflows/types.ts
interface WorkflowDefinition {
  name: string;
  description: string;
  triggers: WorkflowTrigger[];
  actions: WorkflowAction[];
  error_handling: ErrorHandlingConfig;
  monitoring: MonitoringConfig;
}

interface WorkflowAction {
  name: string;
  agent: string;
  inputs: Record<string, any>;
  outputs?: Record<string, any>;
  depends_on?: string[];
  parallel?: boolean;
  timeout?: number;
  retry_policy?: RetryPolicy;
}
```

### Action-Based Trigger System

**Event-Driven Automation**
```yaml
# .cursor/workflows/triggers/git-events.yaml
triggers:
  git_commit:
    patterns:
      - "feat(portals): *"
      - "fix(memory): *"
    actions:
      - validate-affected-systems
      - run-integration-tests
      - update-documentation
      
  git_merge:
    branches: ["main", "develop"]
    actions:
      - deploy-to-staging
      - run-performance-tests
      - notify-team
      
  file_change:
    patterns:
      - "mind-agents/src/characters/*.json"
    actions:
      - validate-character-schema
      - update-character-docs
      - test-character-behavior
```

**Development Lifecycle Integration**
```yaml
# .cursor/workflows/triggers/development.yaml
triggers:
  pr_opened:
    conditions:
      - label: "needs-review"
      - files_changed: "*.ts"
    actions:
      - code-quality-check
      - security-scan
      - documentation-check
      
  issue_labeled:
    labels: ["bug", "high-priority"]
    actions:
      - create-hotfix-branch
      - assign-emergency-team
      - schedule-investigation
      
  deployment_complete:
    environment: "production"
    actions:
      - run-smoke-tests
      - update-monitoring
      - notify-stakeholders
```

## Agent Coordination Patterns

### Multi-Agent Workflows

**Sequential Agent Pipeline**
```yaml
# .cursor/workflows/memory-optimization.yaml
name: "Memory System Optimization"
description: "Automated memory system maintenance and optimization"

agents:
  memory-analyzer:
    model: "gpt-4o"
    context: ["@mind-agents/src/memory/", "@docs/memory/"]
    capabilities: ["analysis", "reporting"]
    
  memory-optimizer:
    model: "claude-3.5-sonnet"
    context: ["@mind-agents/src/memory/", "@AI_MEMORY.md"]
    capabilities: ["code-modification", "optimization"]
    
  memory-tester:
    model: "gpt-4.1-mini"
    context: ["@mind-agents/src/__tests__/memory/"]
    capabilities: ["testing", "validation"]

workflow:
  - step: "analyze"
    agent: "memory-analyzer"
    task: "Analyze memory usage patterns and identify optimization opportunities"
    outputs: ["analysis_report", "optimization_candidates"]
    
  - step: "optimize"
    agent: "memory-optimizer"
    depends_on: ["analyze"]
    task: "Implement optimizations based on analysis report"
    inputs: 
      analysis: "${steps.analyze.outputs.analysis_report}"
      candidates: "${steps.analyze.outputs.optimization_candidates}"
    outputs: ["optimized_code", "change_summary"]
    
  - step: "test"
    agent: "memory-tester"
    depends_on: ["optimize"]
    task: "Test optimized memory system for correctness and performance"
    inputs:
      changes: "${steps.optimize.outputs.change_summary}"
    outputs: ["test_results", "performance_metrics"]
```

**Parallel Agent Execution**
```yaml
# .cursor/workflows/comprehensive-testing.yaml
name: "Comprehensive System Testing"
description: "Parallel testing across all SYMindX components"

parallel_groups:
  core_systems:
    - agent: "portal-tester"
      task: "Test all AI portal configurations"
      context: ["@mind-agents/src/portals/"]
      
    - agent: "memory-tester"
      task: "Test memory provider implementations"
      context: ["@mind-agents/src/memory/"]
      

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
