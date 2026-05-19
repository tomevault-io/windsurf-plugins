---
trigger: always_on
description: - Run `make` to create a virtual environment and install dependencies.
---

# Contributor Guide

## Dev Environment Tips

- Run `make` to create a virtual environment and install dependencies.
- Activate the virtual environment with `source .venv/bin/activate` (Linux/Mac) or `.venv\Scripts\activate` (Windows).

## Testing Instructions

- Run `make format` to format the code.
- Run `make lint` to check for linting errors.
- Run `make test` to run the tests.

## Implementation Philosophy

This section outlines the core implementation philosophy and guidelines for software development projects. It serves as a central reference for decision-making and development approach throughout the project.

### Core Philosophy

Embodies a Zen-like minimalism that values simplicity and clarity above all. This approach reflects:

- **Wabi-sabi philosophy**: Embracing simplicity and the essential. Each line serves a clear purpose without unnecessary embellishment.
- **Occam's Razor thinking**: The solution should be as simple as possible, but no simpler.
- **Trust in emergence**: Complex systems work best when built from simple, well-defined components that do one thing well.
- **Present-moment focus**: The code handles what's needed now rather than anticipating every possible future scenario.
- **Pragmatic trust**: The developer trusts external systems enough to interact with them directly, handling failures as they occur rather than assuming they'll happen.

This development philosophy values clear documentation, readable code, and belief that good architecture emerges from simplicity rather than being imposed through complexity.

### Core Design Principles

#### 1. Ruthless Simplicity

- **KISS principle taken to heart**: Keep everything as simple as possible, but no simpler
- **Minimize abstractions**: Every layer of abstraction must justify its existence
- **Start minimal, grow as needed**: Begin with the simplest implementation that meets current needs
- **Avoid future-proofing**: Don't build for hypothetical future requirements
- **Question everything**: Regularly challenge complexity in the codebase

#### 2. Architectural Integrity with Minimal Implementation

- **Preserve key architectural patterns**: Example: MCP for service communication, SSE for events, separate I/O channels, etc.
- **Simplify implementations**: Maintain pattern benefits with dramatically simpler code
- **Scrappy but structured**: Lightweight implementations of solid architectural foundations
- **End-to-end thinking**: Focus on complete flows rather than perfect components

#### 3. Library Usage Philosophy

- **Use libraries as intended**: Minimal wrappers around external libraries
- **Direct integration**: Avoid unnecessary adapter layers
- **Selective dependency**: Add dependencies only when they provide substantial value
- **Understand what you import**: No black-box dependencies

### Technical Implementation Guidelines

#### API Layer

- Implement only essential endpoints
- Minimal middleware with focused validation
- Clear error responses with useful messages
- Consistent patterns across endpoints

#### Database & Storage

- Simple schema focused on current needs
- Use TEXT/JSON fields to avoid excessive normalization early
- Add indexes only when needed for performance
- Delay complex database features until required

#### MCP Implementation

- Streamlined MCP client with minimal error handling
- Utilize FastMCP when possible, falling back to lower-level only when necessary
- Focus on core functionality without elaborate state management
- Simplified connection lifecycle with basic error recovery
- Implement only essential health checks

#### SSE & Real-time Updates

- Basic SSE connection management
- Simple resource-based subscriptions
- Direct event delivery without complex routing
- Minimal state tracking for connections

### #Event System

- Simple topic-based publisher/subscriber
- Direct event delivery without complex pattern matching
- Clear, minimal event payloads
- Basic error handling for subscribers

#### LLM Integration

- Direct integration with PydanticAI
- Minimal transformation of responses
- Handle common error cases only
- Skip elaborate caching initially

#### Message Routing

- Simplified queue-based processing
- Direct, focused routing logic
- Basic routing decisions without excessive action types
- Simple integration with other components

### Development Approach

#### Vertical Slices

- Implement complete end-to-end functionality slices
- Start with core user journeys
- Get data flowing through all layers early
- Add features horizontally only after core flows work

#### Iterative Implementation

- 80/20 principle: Focus on high-value, low-effort features first
- One working feature > multiple partial features
- Validate with real usage before enhancing
- Be willing to refactor early work as patterns emerge

#### Testing Strategy

- Emphasis on integration and end-to-end tests
- Manual testability as a design goal
- Focus on critical path testing initially
- Add unit tests for complex logic and edge cases
- Testing pyramid: 60% unit, 30% integration, 10% end-to-end

#### Error Handling

- Handle common errors robustly
- Log detailed information for debugging
- Provide clear error messages to users
- Fail fast and visibly during development

### Decision-Making Framework


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/recipe-tool](https://github.com/microsoft/recipe-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
