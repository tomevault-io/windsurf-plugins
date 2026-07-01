---
trigger: always_on
description: **cdk-dia** is an automated diagram generator for AWS CDK applications. It reads a synthesized CDK cloud assembly (`tree.json`), builds an internal component tree representing AWS resources and their relationships, then renders visual diagrams (PNG via Graphviz or interactive HTML via Cytoscape).
---

# AGENTS.md — Guide for AI Agents Working on cdk-dia

## Project Overview

**cdk-dia** is an automated diagram generator for AWS CDK applications. It reads a synthesized CDK cloud assembly (`tree.json`), builds an internal component tree representing AWS resources and their relationships, then renders visual diagrams (PNG via Graphviz or interactive HTML via Cytoscape).

## Architecture

### Pipeline (5 stages)

```
tree.json  →  CDK Tree  →  Component Tree  →  Post-Processing  →  Rendered Diagram
```

1. **Parse** — `src/cdk/tree-json-loader.ts` reads `cdk.out/tree.json` into a `Tree`/`Node` model (`src/cdk/cdk-tree.ts`).
2. **Generate** — `src/diagram/aws/aws-diagram-generator.ts` traverses the CDK tree, creates `DiagramComponent` objects for CloudFormation resources, and builds a hierarchical component tree.
3. **Resolve Edges** — `src/diagram/aws/aws-edge-resolver.ts` scrapes CloudFormation properties for `Ref`, `Fn::GetAtt`, and `Fn::ImportValue` to create edges between components.
4. **Post-Process** — The generator applies filtering (include/exclude stacks, ignore), collapsing (CDK constructs, double clusters), asset removal, cross-stack edge simplification, and self-link removal.
5. **Render** — Either `src/render/graphviz/` (PNG via dot) or `src/render/cytoscape/` (interactive HTML).

### Key Files

| File | Purpose |
|------|---------|
| `bin/cli.ts` | CLI entry point (yargs) |
| `src/cdk-dia.ts` | Main orchestrator — ties parsing, generation, rendering |
| `src/cdk/cdk-tree.ts` | CDK tree data model (`Tree`, `Node`, `ConstructInfoFqn`) |
| `src/cdk/tree-json-loader.ts` | Loads and deserializes `tree.json` |
| `src/cdk/construct-decorator.ts` | `CdkDia.decorate()`, `CdkDiaDecorator`, `@DiagramOptions` |
| `src/diagram/aws/aws-diagram-generator.ts` | Core: tree traversal, component creation, all post-processing steps |
| `src/diagram/aws/aws-edge-resolver.ts` | Edge detection from CloudFormation intrinsic functions |
| `src/diagram/aws/aws-icon-supplier.ts` | Maps `AWS::*` resource types to icon paths |
| `src/diagram/aws/awsResouceIconMatches.json` | 643 resource type → icon mappings |
| `src/diagram/component/component.ts` | Abstract `Component` base class, `ComponentTags` enum |
| `src/diagram/component/customizable-attribute.ts` | `CollapssingCustomizer`, `IgnoreCustomizer` |
| `src/diagram/component/diagram-component.ts` | Concrete `DiagramComponent` |
| `src/diagram/component/root-component.ts` | Root of the component tree |
| `src/diagram/component-links.ts` | Bidirectional edge management |
| `src/render/graphviz/GraphvizGenerator.ts` | Converts diagram → ts-graphviz DOT model |
| `src/render/cytoscape/cytoscape-generator.ts` | Converts diagram → Cytoscape elements/styles |

### Exports

- `src/cdk/index.ts` — re-exports all CDK modules
- `src/diagram/index.ts` — re-exports all diagram modules

## Testing

### ALWAYS write and run tests

When making changes to this project, **you must write tests and verify they pass**. The existing test suite has strong snapshot coverage and graph integrity checks that catch regressions.

### How to run tests

```bash
# Compile TypeScript first (required — Jest runs against compiled JS)
npx tsc

# Run all tests
npm run test

# Run a specific test file
npx jest src/diagram/tests/generator.test.ts

# Update snapshots after intentional changes
npx jest --updateSnapshot
```

### Test framework

- **Jest** with **ts-jest** for TypeScript
- **jest-specific-snapshot** for named snapshot files (stored in `snapshots/`)
- Config: `jest.config.js` — test pattern: `**/src/**/tests/**/*.test.ts`
- Setup: `testSetup.js` — creates/cleans `test-generated/` directory before each run

### Test files and what they cover

| Test File | What It Tests |
|-----------|---------------|
| `src/cdk/tests/cdk-tree.test.ts` | CDK tree parsing from JSON fixtures; snapshot of parsed tree structure |
| `src/diagram/tests/generator.test.ts` | Diagram generation (snapshot), stack include/exclude filtering, ignore feature, graph integrity (all links exist in tree, all sub-components in tree, all links bidirectional) |
| `src/diagram/tests/aws-edge-resolver.test.ts` | Edge target scraping from CloudFormation props (`Fn::GetAtt`, `Ref`, `Fn::ImportValue`); uses custom Jest matcher `arrayToContainEdgeTarget` |
| `src/diagram/tests/stack-exports-container.test.ts` | Stack export parsing |
| `src/render/graphviz/tests/generator.test.ts` | Graphviz DOT output (snapshot); writes `.dot` files to `test-generated/` |
| `src/render/cytoscape/tests/generator.test.ts` | Cytoscape HTML output structure (index.html, icons/, js/, cy-elements.json, cy-styles.json) |

### Test patterns used in this project

**1. Parametrized snapshot tests over all fixtures:**
Most test suites iterate over `testCases` from `src/test-fixtures/testCases.ts` and compare output against named snapshots. When you add a new fixture, you get coverage across CDK parsing, diagram generation, Graphviz DOT, and Cytoscape rendering automatically.

**2. Graph integrity assertions:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pistazie/cdk-dia](https://github.com/pistazie/cdk-dia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
