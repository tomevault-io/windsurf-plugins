---
trigger: always_on
description: Style guide for technical documentation with LLM-optimized structure
---


# Writing Guidelines for Humans and LLMs

## Core Style Principles

| Principle | Definition | Example: Good | Example: Bad |
|-----------|------------|--------------|-------------|
| Clear Language | Use simple, direct language with short words and sentences. Avoid unnecessary jargon. | "Click save to store your changes." | "Initiate the persistence process by activating the storage mechanism." |
| Developer-Focused | Write in a professional but approachable tone that addresses the reader directly. | "You can configure this option in the settings panel." | "Users might want to adjust configuration parameters." |
| Example-Driven | Include practical code examples for all key concepts. | "Use the `useState` hook: `const [count, setCount] = useState(0)`" | "State management is an important React concept." |
| Active Voice | Make the subject perform the action rather than receive it. | "React renders the component." | "The component is rendered by React." |
| Success-First | Show working examples before explaining theory. | "First, create a component: `function Button() {...}`. Now let's understand how it works..." | "The component lifecycle has several phases which you must understand before implementation..." |
| Transparent | Be honest about limitations and challenges. | "This approach works well for small datasets but may cause performance issues with larger ones." | "This is the optimal solution for data management." |
| Consistent Terms | Use the same terminology throughout to refer to the same concepts. | "Route" consistently or "endpoint" consistently, not both interchangeably. | Mixing "callback function" and "handler" for the same concept. |

## For LLM Content Generation

### Formatting Instructions

When generating content with an LLM, ensure the following:

1. **Structured Data**: Present information in well-defined structures like tables, numbered lists, and hierarchical headings
2. **Explicit Examples**: Always include contrastive examples (good vs. bad)
3. **Clear Boundaries**: Use explicit section markers and consistent formatting patterns
4. **Context-Awareness**: Begin with the most critical information for the document type
5. **Pattern Consistency**: Maintain consistent patterns throughout similar sections

### LLM Content Templates

For each document type, LLMs should structure content as follows:

```yaml
DocumentType: [Tutorial|HowTo|Reference|Explanation]
Audience: [Beginner|Intermediate|Advanced]
PrimaryGoal: "Single sentence describing document purpose"
Sections:
  - Name: "Introduction"
    Content: "Clear goal statement with outcomes"
  - Name: "Prerequisites"
    Content: "Bulleted list of requirements"
  - Name: "MainContent"
    Content: "Follows appropriate structure for document type"
  - Name: "Conclusion"
    Content: "Summary and next steps"
```

## Document Types and Their Styles

### Tutorials

**Purpose**: Guide beginners through their first complete experience
**Structure**:
- Start with a clear goal statement and outcomes
- Use numbered steps in chronological order
- Present minimal viable examples without alternatives
- Confirm success at each step

**Writing Rules**:
- Use first person plural ("Let's build") or second person ("You'll start by")
- Include complete, copy-pasteable code blocks
- Avoid introducing errors or debugging scenarios
- Use encouraging language with confirmation statements

**Example Format**:
```
## Getting Started with React

In this tutorial, you'll build your first React component. By the end, you'll have a working button component and understand basic React concepts.

### Prerequisites
- Node.js v18+
- Basic JavaScript knowledge

### Step 1: Create a new React project
Run the following command:

```bash
npx create-react-app my-first-app
cd my-first-app
```

You should see a new folder created with your React application.
```

### How-To Guides

**Purpose**: Provide task-oriented instructions for specific problems
**Structure**:
- Title using format "How to [accomplish specific task]"
- Numbered steps with clear outcomes
- Links to reference materials rather than full explanations

**Writing Rules**:
- Address the reader directly as "you"
- Assume basic familiarity with the technology
- Keep explanations focused only on completing the task
- Include prerequisites explicitly

**Example Format**:
```
## How to Add Authentication to a React Application

This guide shows you how to implement user authentication in an existing React application.

### Prerequisites
- A React application
- Basic understanding of React hooks

### Steps

1. Install the authentication library
   ```package-install
  @auth/react
   ```

2. Create an authentication context
   ```jsx
   function AuthProvider({ children }) {
     // Implementation details
     return <AuthContext.Provider value={auth}>{children}</AuthContext.Provider>;
   }
   ```
```

### Reference Documentation

**Purpose**: Provide comprehensive technical details about APIs, props, methods
**Structure**:
- Structured, consistent formatting for each item
- Complete coverage of parameters, return values, and types
- Tables, lists, and hierarchical organization

**Writing Rules**:
- Use neutral, precise language 
- Prioritize completeness over storytelling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [c15t/c15t](https://github.com/c15t/c15t) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
