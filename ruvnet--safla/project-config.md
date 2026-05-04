---
trigger: always_on
description: This project uses the SPARC (Specification, Pseudocode, Architecture, Refinement, Completion) methodology for systematic Test-Driven Development with AI assistance through Claude-Flow orchestration.
---

# Claude Code Configuration - SPARC Development Environment

## Project Overview
This project uses the SPARC (Specification, Pseudocode, Architecture, Refinement, Completion) methodology for systematic Test-Driven Development with AI assistance through Claude-Flow orchestration.

## SPARC Development Commands

### Core SPARC Commands
- `npx claude-flow sparc modes`: List all available SPARC development modes
- `npx claude-flow sparc run <mode> "<task>"`: Execute specific SPARC mode for a task
- `npx claude-flow sparc tdd "<feature>"`: Run complete TDD workflow using SPARC methodology
- `npx claude-flow sparc info <mode>`: Get detailed information about a specific mode

### Standard Build Commands
- `npm run build`: Build the project
- `npm run test`: Run the test suite
- `npm run lint`: Run linter and format checks
- `npm run typecheck`: Run TypeScript type checking

## SPARC Methodology Workflow

### 1. Specification Phase
```bash
# Create detailed specifications and requirements
npx claude-flow sparc run spec-pseudocode "Define user authentication requirements"
```
- Define clear functional requirements
- Document edge cases and constraints
- Create user stories and acceptance criteria
- Establish non-functional requirements

### 2. Pseudocode Phase
```bash
# Develop algorithmic logic and data flows
npx claude-flow sparc run spec-pseudocode "Create authentication flow pseudocode"
```
- Break down complex logic into steps
- Define data structures and interfaces
- Plan error handling and edge cases
- Create modular, testable components

### 3. Architecture Phase
```bash
# Design system architecture and component structure
npx claude-flow sparc run architect "Design authentication service architecture"
```
- Create system diagrams and component relationships
- Define API contracts and interfaces
- Plan database schemas and data flows
- Establish security and scalability patterns

### 4. Refinement Phase (TDD Implementation)
```bash
# Execute Test-Driven Development cycle
npx claude-flow sparc tdd "implement user authentication system"
```

**TDD Cycle:**
1. **Red**: Write failing tests first
2. **Green**: Implement minimal code to pass tests
3. **Refactor**: Optimize and clean up code
4. **Repeat**: Continue until feature is complete

### 5. Completion Phase
```bash
# Integration, documentation, and validation
npx claude-flow sparc run integration "integrate authentication with user management"
```
- Integrate all components
- Perform end-to-end testing
- Create comprehensive documentation
- Validate against original requirements

## SPARC Mode Reference

### Development Modes
- **`architect`**: System design and architecture planning
- **`code`**: Clean, modular code implementation
- **`tdd`**: Test-driven development and testing
- **`spec-pseudocode`**: Requirements and algorithmic planning
- **`integration`**: System integration and coordination

### Quality Assurance Modes
- **`debug`**: Troubleshooting and bug resolution
- **`security-review`**: Security analysis and vulnerability assessment
- **`refinement-optimization-mode`**: Performance optimization and refactoring

### Support Modes
- **`docs-writer`**: Documentation creation and maintenance
- **`devops`**: Deployment and infrastructure management
- **`mcp`**: External service integration
- **`swarm`**: Multi-agent coordination for complex tasks

## Claude Code Slash Commands

Claude Code slash commands are available in `.claude/commands/`:

### Project Commands
- `/sparc`: Execute SPARC methodology workflows
- `/sparc-<mode>`: Run specific SPARC mode (e.g., /sparc-architect)
- `/claude-flow-help`: Show all Claude-Flow commands
- `/claude-flow-memory`: Interact with memory system
- `/claude-flow-swarm`: Coordinate multi-agent swarms

### Using Slash Commands
1. Type `/` in Claude Code to see available commands
2. Select a command or type its name
3. Commands are context-aware and project-specific
4. Custom commands can be added to `.claude/commands/`

## Code Style and Best Practices

### SPARC Development Principles
- **Modular Design**: Keep files under 500 lines, break into logical components
- **Environment Safety**: Never hardcode secrets or environment-specific values
- **Test-First**: Always write tests before implementation (Red-Green-Refactor)
- **Clean Architecture**: Separate concerns, use dependency injection
- **Documentation**: Maintain clear, up-to-date documentation

### Coding Standards
- Use TypeScript for type safety and better tooling
- Follow consistent naming conventions (camelCase for variables, PascalCase for classes)
- Implement proper error handling and logging
- Use async/await for asynchronous operations
- Prefer composition over inheritance

### Memory and State Management
- Use claude-flow memory system for persistent state across sessions
- Store progress and findings using namespaced keys
- Query previous work before starting new tasks
- Export/import memory for backup and sharing

## SPARC Memory Integration

### Memory Commands for SPARC Development
```bash
# Store project specifications

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ruvnet/SAFLA](https://github.com/ruvnet/SAFLA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
