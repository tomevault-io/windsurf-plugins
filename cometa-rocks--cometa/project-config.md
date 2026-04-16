---
trigger: always_on
description: This is a full-stack application with Angular 15 frontend and Python/Django backend for automated testing and monitoring. The project includes specialized components for test automation, monitoring, scheduling, and AI-powered features.
---

# Cursor Rules for Cometa Project

## Project Overview
This is a full-stack application with Angular 15 frontend and Python/Django backend for automated testing and monitoring. The project includes specialized components for test automation, monitoring, scheduling, and AI-powered features.

## General Coding Standards

### Code Style
- Use consistent indentation (2 spaces for frontend, 4 spaces for backend)
- Follow the existing naming conventions in each module
- Write self-documenting code with clear variable and function names
- Add JSDoc comments for complex functions and classes
- Use consistent formatting across all file types

### File Organization
- Keep related files together in appropriate directories
- Use descriptive file names that reflect their purpose
- Group imports logically (Angular/core, third-party, local)
- Maintain clear separation between frontend and backend
- Use proper package structure for each component

### Error Handling
- Always handle errors gracefully
- Use try-catch blocks where appropriate
- Log errors for debugging purposes
- Provide user-friendly error messages
- Implement proper error boundaries

### Documentation
- Update README files when adding new features
- Document API endpoints thoroughly
- Include setup and deployment instructions
- Maintain changelog for major changes
- Document complex business logic

## Frontend (Angular 15) Rules

### Component Structure
- Use OnPush change detection strategy when possible
- Implement OnDestroy for components with subscriptions
- Use async pipe for observables in templates
- Keep components focused on single responsibility
- Follow Angular Material design patterns

### Service Guidelines
- Use dependency injection for services
- Make services stateless when possible
- Use BehaviorSubject for state management
- Implement proper error handling in HTTP calls
- Follow the existing RPC pattern for service communication

### Template Best Practices
- Use trackBy functions for *ngFor loops
- Avoid complex logic in templates
- Use safe navigation operator (?.) for null checks
- Keep templates readable and maintainable
- Implement proper accessibility features

### State Management
- Use NgRx store for global state management
- Implement proper actions, reducers, and selectors
- Use NgRx effects for side effects
- Keep state normalized and flat
- Use immer for immutable state updates

## Backend (Python/Django) Rules

### Python Standards
- Follow PEP 8 style guidelines strictly
- Use type hints where appropriate
- Write docstrings for all public functions and classes
- Use virtual environments for dependency management
- Implement proper error handling and logging

### Django Conventions
- Use Django ORM for database operations
- Implement proper model validation
- Use Django forms for data input
- Follow Django security best practices
- Implement proper API endpoints with DRF

### Cometa-Specific Features
- Implement proper test automation workflows
- Handle test scheduling and execution
- Implement monitoring and alerting systems
- Use WebSockets for real-time communication
- Implement proper file management and storage

## Testing Guidelines

### Frontend Testing
- Write unit tests for all components, services, and pipes
- Use Angular testing utilities and Jasmine
- Mock external dependencies properly
- Test both success and error scenarios
- Maintain good test coverage

### Backend Testing
- Write unit tests for all models, views, and services
- Use Django testing framework
- Mock external services and APIs
- Test database operations thoroughly
- Implement integration tests

### End-to-End Testing
- Use proper E2E testing frameworks
- Test critical user workflows
- Implement automated testing pipelines
- Test cross-browser compatibility
- Monitor test execution performance

## Security Considerations

### Data Protection
- Never commit sensitive information (API keys, passwords)
- Use environment variables for configuration
- Validate all user inputs thoroughly
- Implement proper authentication and authorization
- Use secure communication protocols

### API Security
- Implement proper API authentication
- Use rate limiting and request validation
- Implement CSRF protection
- Use secure headers and HTTPS
- Monitor for suspicious activity

### File Security
- Validate all file uploads
- Implement proper file permissions
- Use secure file storage backends
- Sanitize file content when necessary
- Implement proper file access controls

## Performance Guidelines

### Frontend Optimization
- Optimize bundle sizes and loading times
- Use lazy loading for feature modules
- Implement proper change detection
- Use virtual scrolling for large datasets
- Monitor and optimize render performance

### Backend Optimization
- Optimize database queries and indexing
- Implement proper caching strategies
- Use async operations when appropriate
- Monitor API response times
- Implement connection pooling

### Infrastructure
- Use containerization for deployment
- Implement proper load balancing
- Monitor resource usage and scaling
- Use CDN for static assets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cometa-rocks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
