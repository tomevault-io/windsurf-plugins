---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SwarmUI is a ViCE (Vibe Coding Environment) - a modern alternative to traditional IDEs that enables natural, conversational coding through AI integration. It's a Ruby on Rails 8.0.2 application using modern Rails conventions and the Hotwire stack (Turbo + Stimulus) for frontend interactivity.

### The ViCE Philosophy

SwarmUI is purpose-built for vibe coding mastery. Unlike traditional IDEs that bolt on AI features, SwarmUI provides the perfect environment for developers who:

- Have transcended traditional tool-based workflows
- Want to master the art of conversational development
- Need precisely the right tools without unnecessary complexity
- Seek a pure vibe coding experience

Key principles:
- **Minimal, essential tooling** - Just what you need for vibe coding, nothing more
- **Natural language first** - Express intent, not commands
- **Flow state optimization** - No interruptions, no friction
- **AI-native workflows** - Built for Claude collaboration from day one

This isn't an IDE with AI features added - it's a precision instrument for vibe coding practitioners.

## Key Technologies

- **Ruby**: 3.4.2
- **Rails**: 8.0.2
- **Database**: PostgreSQL
- **CSS Framework**: Tailwind CSS
- **JavaScript**: Import Maps (no Node.js build process)
- **Frontend Stack**: Hotwire (Turbo + Stimulus)
- **Asset Pipeline**: Propshaft
- **Background Jobs**: Solid Queue
- **Caching**: Solid Cache
- **WebSockets**: Solid Cable
- **Icons**: Heroicon (SVG icon library)

## Development Commands

NOTE: NEVER TRY TO RUN THE RAILS SERVER! If you need it restarted, ask the user.

### Initial Setup
```bash
bin/setup
```

### Running the Development Server
```bash
bin/dev
```
This starts both the Rails server and Tailwind CSS watcher.

### Database Commands
```bash
bin/rails db:create      # Create the database
bin/rails db:migrate     # Run migrations
bin/rails db:seed        # Seed the database
bin/rails db:prepare     # Create, migrate, and seed in one command
```

### Testing
```bash
bin/rails test           # Run all tests
bin/rails test test/models/user_test.rb  # Run specific test file
bin/rails test test/models/user_test.rb:42  # Run specific test by line number
```

### Code Quality
```bash
bin/rubocop -A           # Run RuboCop for code style checks and auto-fix violations
bin/brakeman             # Run security vulnerability scan
```

## Code Architecture

The application follows standard Rails conventions:

- **app/controllers/**: HTTP request handlers
- **app/models/**: ActiveRecord models for database interaction
- **app/views/**: ERB templates for HTML rendering
- **app/javascript/**: Stimulus controllers and JavaScript modules
- **app/assets/**: Static assets and stylesheets
- **config/**: Application configuration
- **db/**: Database schema and migrations
- **test/**: Minitest test files

### Testing Stack
- **Minitest**: Default Rails testing framework
- **Factory Bot**: Test data generation
- **Capybara**: Integration/system testing
- **WebMock/VCR**: HTTP request mocking

### Code Style
The project uses RuboCop with Shopify's style guide. Always run `bin/rubocop -A` and fix linting errors before committing changes.

## Development Workflow

1. NEVER START OR RESTART THE DEVELOPMENT SERVER
2. The server runs on http://localhost:3000 by default
3. Tailwind CSS compilation happens automatically
4. Rails automatically reloads on code changes
5. Use `bin/rails console` for interactive debugging

## Important Conventions

- Follow existing code patterns and Rails conventions
- Use Turbo for page updates instead of full page reloads
- Write Stimulus controllers for JavaScript behavior
- Keep controllers thin and models fat
- Write tests for all new functionality
- Use strong parameters in controllers
- Follow RESTful routing conventions

## Forms
- Use Simple Form for all forms in the application
- Simple Form has been configured to work with Tailwind CSS
- Refer to `llm_docs/simple_form_tailwind.md` for usage guidelines and examples
- The configuration uses custom wrappers for proper Tailwind styling

## UI Design Guidelines
- Always use Tailwind CSS 4 utility classes for styling in views and components.
- Follow the official Tailwind CSS 4 documentation that is in llm_docs/tailwind for syntax, features, and best practices.
- Prefer semantic HTML and accessibility best practices alongside Tailwind classes.
- Avoid custom CSS unless a design cannot be achieved with Tailwind utilities.
- For custom themes, define variables using `@theme` in app/assets/tailwind/application.css.

### Tailwind CSS 4 Documentation
The `llm_docs/tailwind/` directory contains the complete Tailwind CSS 4 documentation split into individual MDX files. 

**IMPORTANT**: Always consult `llm_docs/tailwind/INDEX.md` first to quickly find the right documentation file for any Tailwind utility class or concept.

When you need to:
- Look up a specific utility class → Read `llm_docs/tailwind/[utility-name].mdx` (e.g., `llm_docs/tailwind/background-color.mdx` for bg-* classes)
- Understand responsive design → Read `llm_docs/tailwind/responsive-design.mdx`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parruda/claude-swarm-ui](https://github.com/parruda/claude-swarm-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
