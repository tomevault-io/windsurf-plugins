---
trigger: always_on
description: Rails Development Rules - The Rails Way with AI Agents
---

# Rails Development Rules - The Rails Way with AI Agents
***Built for: Solo developer + AI agents, Linear MCP integration, maximum joy***

## Core Philosophy
You are building a Rails 8+ application following The Rails Way™. Code should be:
- Convention over configuration
- Database-first design
- Progressive enhancement with Hotwire
- Zero-build frontend approach
- Test-driven with Minitest
- Optimized for AI agent collaboration
- Lean, readable, and maintainable

## Rails 8+ Stack Preferences
- **Authentication**: Rails built-in (`rails generate authentication`)
- **Background Jobs**: Solid Queue (default Rails 8)
- **Caching**: Solid Cache (database-backed, Redis when needed)
- **WebSockets**: Action Cable with Solid Cable adapter
- **Database**: PostgreSQL with Active Record
- **Frontend**: Hotwire (Turbo + Stimulus) + TailwindCSS
- **Asset Pipeline**: Propshaft (simpler, no-build approach)
- **Testing**: Minitest with fixtures (no RSpec, no factories)
- **Rich Text**: Action Text for content editing
- **File Uploads**: Active Storage with direct uploads
- **Deployment**: Kamal with Docker
- **Code Quality**: StandardRB for linting/formatting
- **Development Tools**: Bullet gem for N+1 detection, Annotate gem for schema docs
- **Error Tracking**: Rails built-in error reporter

## Detailed Guides
This document provides core principles. For detailed implementation guidance, see:

- **[core.md](rails/core.md)** - Rails 8 conventions and patterns
- **[models.md](rails/models.md)** - Active Record patterns and best practices
- **[controllers.md](rails/controllers.md)** - Controller design and RESTful patterns
- **[services.md](rails/services.md)** - Service objects and business logic
- **[testing.md](rails/testing.md)** - Testing philosophy and Minitest patterns
- **[security.md](rails/security.md)** - Security best practices and authorization
- **[performance.md](rails/performance.md)** - Database optimization and caching
- **[api.md](rails/api.md)** - API design and versioning (when needed)
- **[importmaps.md](rails/importmaps.md)** - JavaScript without build steps
- **[hotwire.md](rails/hotwire.md)** - Turbo & Stimulus patterns
- **[views.md](rails/views.md)** - View helpers and rendering
- **[styling.md](rails/styling.md)** - TailwindCSS integration
- **[background-jobs.md](rails/background-jobs.md)** - Solid Queue configuration
- **[deployment.md](rails/deployment.md)** - Kamal deployment guide
- **[mobile.md](rails/mobile.md)** - Hotwire Native for mobile apps (optional)

## File Organization & Naming

### Consistent Directory Structure
```
app/
├── controllers/
│   ├── concerns/
│   └── application_controller.rb
├── models/
│   ├── concerns/
│   └── application_record.rb
├── views/
│   ├── layouts/
│   ├── shared/
│   └── [resource_name]/
├── services/
├── jobs/
├── channels/
├── mailers/
└── helpers/
```

### Naming Conventions for AI Agents
- **Classes**: PascalCase, descriptive (`UserRegistrationService`, `InvoicePaymentProcessor`)
- **Files**: snake_case matching class name (`user_registration_service.rb`)
- **Methods**: snake_case, verb-first for actions (`process_payment`, `calculate_total`)
- **Variables**: snake_case, noun-first (`current_user`, `payment_amount`)
- **Constants**: SCREAMING_SNAKE_CASE (`MAX_RETRY_ATTEMPTS`, `DEFAULT_CURRENCY`)

### AI Agent File Patterns
Always organize files predictably:
```
app/models/user.rb                    # Model: singular
app/controllers/users_controller.rb   # Controller: plural + _controller
app/services/user_registration.rb     # Service: domain + action
app/jobs/send_welcome_email_job.rb    # Job: action + _job
app/views/users/index.html.erb        # View: controller/action
```

### AI-Friendly Documentation
- Include purpose statement for AI comprehension
- Reference Linear ticket context (ID-123)
- Document key dependencies and return values
- Specify potential errors and exceptions
- Keep documentation close to code

## Linear Integration (Project Management)

### Overview
This section contains Linear-specific integration patterns. The same principles can be adapted for other project management tools by replacing Linear ticket formats and magic words with platform-specific equivalents.

### Commit Message Format
- Use Conventional Commits format with issue references
- Structure: `type(scope): description` followed by body and footer
- Include ticket reference in commit body or footer
- Example:
  ```
  feat: add magic link authentication

  Implements passwordless login flow
  Fixes ID-123
  ```

### Issue Linking
- Use semantic keywords to manage issue state through commits
- **Closing keywords**: `close`, `closes`, `closed`, `closing fix`, `fixes`, `fixed`, `fixing`, `resolve`, `resolves`, `resolved`, `resolving`, `complete`, `completes`, `completed`, `completing` (auto-close issues)
- **Reference keywords**: `ref`, `refs`, `references`, `part of`, `related to`, `contributes to`, `toward`, `towards` (link without closing)
- Support multiple issues: `Fixes ID-123, ID-456`

### Branch Strategy
- Follow GitHub Flow with descriptive branch names
- Format: `type/ticket-id/brief-description`
- Example: `feat/id-123/magic-link-login`
- Keep branches short-lived and focused

### Documentation Integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [levifig/rails-instructions](https://github.com/levifig/rails-instructions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
