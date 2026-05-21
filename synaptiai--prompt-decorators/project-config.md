---
trigger: always_on
description: APPLY software development decorators WHEN coding TO improve code quality and clarity
---


# Software Development Prompt Decorators

<version>1.0.0</version>

## Context
- Use these decorators when working on software development tasks
- Apply when generating code, designing systems, debugging issues, or reviewing code
- Use individually or combine multiple decorators for more specific results
- These decorators enhance the way the AI responds to coding-related requests

## Requirements
- Recognize and process all software development prompt decorators with the `+++` prefix
- Apply the appropriate transformation based on the decorator and its parameters
- Maintain the core query intent while applying decorator-specific modifications
- Support combinations of multiple decorators applied in sequence
- Follow the standard parameter parsing rules for all decorators

## Supported Decorators

### `+++Algorithm`
When this decorator is applied, the response implements specific algorithms with the desired complexity characteristics.

**Parameters:**
- `type (sorting | search | graph | string | numeric | ml | crypto)`: Algorithm category
- `complexity (constant | logarithmic | linear | linearithmic | quadratic | polynomial | exponential)`: Desired time complexity
- `approach (recursive | iterative | divide-conquer | dynamic | greedy)`: Algorithm design approach

**Example:**
```
+++Algorithm(type=graph, complexity=linear, approach=iterative)
Implement an algorithm to find the shortest path between two nodes in an unweighted graph.
```

### `+++APIDesign`
When this decorator is applied, the response designs API interfaces focusing on specific qualities.

**Parameters:**
- `style (rest | graphql | grpc | soap | websocket | webhook)`: API architectural style
- `focus (consistency | performance | developer-experience | backward-compatibility)`: Design priority
- `documentation (openapi | graphql-schema | protobuf | custom | style-appropriate)`: Documentation approach

**Example:**
```
+++APIDesign(style=graphql, focus=developer-experience, documentation=graphql-schema)
Design a GraphQL API for a content management system that prioritizes a great developer experience.
```

### `+++BugDiagnosis`
When this decorator is applied, the response systematically diagnoses software bugs using specified approaches.

**Parameters:**
- `method (bisection | logging | tracing | debugging | testing)`: Primary diagnostic method
- `level (symptoms | immediate-cause | root-cause)`: Depth of diagnosis
- `scope (code | configuration | environment | integration)`: Area to focus investigation

**Example:**
```
+++BugDiagnosis(method=bisection, level=root-cause, scope=integration)
Our login system intermittently fails when users attempt to authenticate with third-party providers.
```

### `+++CodeWalkthrough`
When this decorator is applied, the response provides a detailed explanation of code, following a specified pedagogical approach.

**Parameters:**
- `style (line-by-line | functional | conceptual | architectural)`: Explanation approach
- `audience (beginner | intermediate | advanced)`: Target knowledge level
- `highlight (patterns | gotchas | optimizations | all)`: Aspects to emphasize

**Example:**
```
+++CodeWalkthrough(style=functional, audience=intermediate, highlight=patterns)
Explain how this React component manages state and renders the UI.
```

### `+++DesignPattern`
When this decorator is applied, the response implements or explains software design patterns with appropriate applications.

**Parameters:**
- `pattern (singleton | factory | adapter | observer | strategy | command | facade)`: The design pattern to implement
- `language (python | javascript | typescript | java | csharp | go | rust)`: Programming language to use
- `variation (string)`: Specific variation of the pattern

**Example:**
```
+++DesignPattern(pattern=observer, language=javascript)
Create a notification system for an e-commerce application that alerts different parts of the UI when the cart changes.
```

### `+++TestCases`
When this decorator is applied, the response generates software test cases with specific coverage and methodologies.

**Parameters:**
- `type (unit | integration | functional | performance | security)`: Test category
- `coverage (happy-path | edge-cases | error-handling | all)`: Test coverage focus
- `framework (jest | pytest | junit | mocha | other)`: Testing framework to use

**Example:**
```
+++TestCases(type=unit, coverage=all, framework=pytest)
Write tests for this user authentication function.
```

### `+++SystemDiagram`
When this decorator is applied, the response creates a visual representation of system architecture or flow.

**Parameters:**
- `type (component | sequence | class | deployment | other)`: Diagram type
- `notation (uml | c4 | informal | boxes-arrows)`: Diagramming notation
- `detail (high-level | detailed)`: Level of detail to include

**Example:**
```
+++SystemDiagram(type=sequence, notation=uml, detail=detailed)
Create a diagram showing the authentication flow in our web application.
```

### `+++TechDebt`
When this decorator is applied, the response identifies and analyzes technical debt in code or architecture.

**Parameters:**
- `focus (code-quality | architecture | testing | documentation | other)`: Area of technical debt

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [synaptiai/prompt-decorators](https://github.com/synaptiai/prompt-decorators) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
