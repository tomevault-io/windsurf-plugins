---
trigger: always_on
description: Zensor Portal UI is a React-based IoT device management interface for the Zensor system. It provides multi-tenant device management with real-time WebSocket data streaming and specialized irrigation control features.
---

# Zensor Portal UI - Project Intelligence

## Project Overview
Zensor Portal UI is a React-based IoT device management interface for the Zensor system. It provides multi-tenant device management with real-time WebSocket data streaming and specialized irrigation control features.

## Key Technical Patterns

### Component Architecture
- Use functional components with React hooks
- Prefer composition over inheritance
- Keep components focused and single-purpose
- Use custom hooks for reusable logic (e.g., useWebSocket)

### State Management
- Use React useState for local component state
- Avoid prop drilling - consider context for shared state
- Prefer derived state over storing computed values
- Handle loading, error, and success states consistently

### Styling Approach
- Use CSS variables for theming (defined in :root)
- Follow mobile-first responsive design
- Use CSS Grid and Flexbox for layouts
- Keep styles co-located with components when possible
- Use consistent spacing (8px grid system)

### API Integration
- Use environment variables for API configuration
- Handle errors gracefully with user-friendly messages
- Show loading states during API calls
- Implement retry logic for failed requests
- Use proper HTTP status code handling

## WebSocket Patterns

### Connection Management
- Use custom useWebSocket hook for connection management
- Implement automatic reconnection with exponential backoff
- Show connection status to users
- Handle connection errors gracefully

### Message Handling
- Parse JSON messages safely with error handling
- Update UI state based on message content
- Handle malformed messages gracefully
- Maintain state consistency during reconnections

### Relay Control (Specialized)
- Display relay values as "On" (green) or "Off" (red)
- Disable irrigation controls when relay is active
- Wait for first WebSocket message before enabling controls
- Monitor relay state changes for irrigation completion

## Scheduled Tasks Patterns

### API Integration
- Use `/v1/tenants/{tenant_id}/devices/{device_id}/scheduled-tasks` endpoints
- Implement full CRUD operations (GET, POST, PUT, DELETE)
- Handle pagination for task listing
- Use proper error handling for all operations

### Task Management
- Support cron expressions for scheduling (6 fields: second minute hour day month dayOfWeek)
- Convert duration from minutes (UI) to seconds (API)
- Implement active/inactive toggle for task status
- Provide human-readable display of cron expressions

### UI/UX Patterns
- Use form-based creation/editing with validation
- Provide preset examples for common schedules (Daily 6 AM, 12 PM, 6 PM)
- Show task status with visual indicators (active/inactive)
- Implement confirmation dialogs for destructive actions
- Use responsive design for mobile compatibility

### Command Structure
- Use irrigation commands with index, value, priority, and wait_for fields
- Structure: Two commands per irrigation task:
  - Activate: `{ index: 1, value: 1, priority: "NORMAL", wait_for: "0s" }`
  - Deactivate: `{ index: 1, value: 0, priority: "NORMAL", wait_for: "5m" }`
- Support multiple commands per scheduled task
- Handle command validation and error states

## UI/UX Patterns

### Navigation
- Use React Router for client-side routing
- Implement breadcrumb navigation for complex flows
- Provide clear visual feedback for current location
- Use consistent navigation patterns throughout

### Data Display
- Use card-based layouts for data presentation
- Show loading states during data fetching
- Display empty states when no data is available
- Use consistent typography and spacing

### Error Handling
- Show user-friendly error messages
- Provide retry options when appropriate
- Use consistent error state styling
- Log errors for debugging

### Responsive Design
- Design mobile-first
- Use CSS Grid for responsive layouts
- Ensure touch-friendly controls on mobile
- Test on various screen sizes

## Code Quality Standards

### JavaScript/React
- Use modern ES6+ features
- Prefer const over let, avoid var
- Use destructuring for props and state
- Implement proper error boundaries
- Use meaningful variable and function names

### CSS
- Use CSS variables for consistent theming
- Follow BEM-like naming conventions
- Keep selectors specific and avoid deep nesting
- Use semantic class names
- Optimize for performance

### File Organization
- Group related components together
- Use descriptive file names
- Keep components focused and single-purpose
- Separate concerns (UI, logic, styling)

### Documentation Organization
- **MANDATORY**: All documentation files must be placed in `./docs` folder
- Create comprehensive docs/README.md with navigation and categorization
- Organize documentation by type (Configuration, Development, API, Integration)
- Use consistent Markdown formatting and cross-referencing
- Never create documentation files in the root directory

## API Integration Patterns

### Endpoints
- `/v1/tenants` - Tenant management
- `/v1/tenants/{id}/devices` - Device listing
- `/v1/devices/{id}` - Device operations
- `/v1/devices/{id}/tasks` - Task management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zensor-iot/zensor-ui](https://github.com/zensor-iot/zensor-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
