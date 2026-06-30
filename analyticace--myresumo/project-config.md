---
trigger: always_on
description: *My Github Username is:* **AnalyticAce**
---

# Copilot Instructions - Best Development Practices

*My Github Username is:* **AnalyticAce**

This guide establishes development standards and best practices for our tech stack: Python, FastAPI, HTML/CSS, Jinja2, Alpine.js, and AI integrations. Following these practices ensures maintainable, sustainable, and high-quality systems.

## General Instructions
- Always prioritize readability and clarity.
- For algorithm-related code, include explanations of the approach used.
- Write code with good maintainability practices, including comments on why certain design decisions were made.
- Handle edge cases and write clear exception handling.
- For libraries or external dependencies, mention their usage and purpose in comments.
- If possible use available tools to search libary documentation to write and suggestion updated code.
- Use consistent naming conventions and follow language-specific best practices.
- Write concise, efficient, and idiomatic code that is also easily understandable.

## Python Development

### Code Style & Structure
- Provide docstrings following PEP 257 conventions and use `ruff`
- Structure projects using a clear module hierarchy
- Use the typing module for type annotations (e.g., List[str], Dict[str, int]).
- Create meaningful docstrings (Google style recommended)
- Break down complex functions into smaller, more manageable functions.
- Maintain proper indentation (use 4 spaces for each level of indentation).
- Place function and class docstrings immediately after the `def` or `class` keyword.
- Use blank lines to separate functions, classes, and code blocks where appropriate.

### Python Best Practices
- Prefer explicit code over implicit
- Use virtual environments (`venv`, `poetry`, or `pipenv`)
- Implement comprehensive error handling with specific exception types
- Follow SOLID principles for OOP code
- Leverage dataclasses for data containers
- Use enums for related constants

### Testing and Edge Cases
- Write unit tests with pytest (aim for >80% coverage)
- Implement integration and end-to-end tests
- Use test fixtures for reusable test components
- Mock external dependencies and services
- Practice TDD where applicable
- Always include test cases for critical paths of the application.
- Account for common edge cases like empty inputs, invalid data types, and large datasets.
- Include comments for edge cases and the expected behavior in those cases.
- Write unit tests for functions and document them with docstrings explaining the test cases.


## FastAPI Development

### API Design
- Follow RESTful principles
- Use Pydantic models for request/response validation
- Implement proper status codes and error responses
- Version your APIs (path-based preferred: `/api/v1/...`)
- Organize endpoints using APIRouter for logical grouping

### FastAPI Features
- Leverage dependency injection for shared components
- Use background tasks for non-blocking operations
- Implement middleware for cross-cutting concerns
- Utilize FastAPI's built-in OpenAPI documentation
- Set up proper CORS handling

### Performance
- Use async/await for I/O-bound operations
- Implement caching for expensive operations
- Use connection pooling for database access
- Monitor endpoint performance
- Consider pagination for large result sets

## Frontend Development

### HTML Best Practices
- Use semantic HTML5 elements
- Ensure proper accessibility (ARIA attributes, proper heading structure)
- Validate markup with W3C validator
- Implement responsive design principles
- Keep markup clean and minimal

### CSS Structure
- Follow a naming convention (BEM recommended)
- Use CSS custom properties for theming
- Implement mobile-first responsive design
- Minimize specificity conflicts
- Consider utility-first approach for complex UIs

### Jinja2 Templating
- Create a base template with blocks for content sections
- Use macros for reusable components
- Keep logic in templates minimal
- Leverage template inheritance
- Use includes for partial templates
- Handle empty states gracefully

### Alpine.js Implementation
- Use for interactive components that don't require complex state management
- Follow progressive enhancement principles
- Keep Alpine components focused on a single responsibility
- Use x-data for component state
- Prefer declarative templates over imperative code

## AI Engineering

### Model Integration
- Abstract AI services behind clean interfaces
- Implement circuit breakers for external AI services
- Version your prompts and store them separately from code
- Log interactions with AI services for debugging
- Implement fallbacks for AI service failures

### Prompt Engineering
- Create clear, specific prompts with examples
- Use structured outputs when possible (JSON, etc.)
- Implement prompt versioning
- Test prompts with diverse inputs
- Document prompt design decisions

### Responsible AI
- Implement content filtering and safety measures
- Consider bias and fairness in AI implementations
- Provide clear indicators when content is AI-generated
- Implement user feedback mechanisms
- Set up monitoring for AI system outputs

## Version Control & Commit Practices

### Commit Message Structure
- Use a structured format: `<type>(<scope>): <subject>`
- Keep first line under 72 characters

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnalyticAce/MyResumo](https://github.com/AnalyticAce/MyResumo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
