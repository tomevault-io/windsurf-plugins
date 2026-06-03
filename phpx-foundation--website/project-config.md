---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<project-guidelines>
PHP× is a global network of PHP and Laravel meetup groups. This is a multi-tenant Laravel application that hosts individual meetup group websites and serves as a central registry for PHP× groups worldwide.

The project originally heavily relied on the actions pattern, but has been refactored to use modern Filament conventions. Some actions still exist but are slowly being replaced.

See @README.md for project setup instructions.

## Architecture

### Multi-Tenant System
- Domain-based routing with `SetGroupFromDomainMiddleware` determines which group is being accessed
- Root domains (phpx.world, phpx.test) serve the main site with globe visualization
- Individual groups have their own subdomains/custom domains

### Key Models & Relationships
- **Group**: Represents a meetup group with domain, location, social links, Mailcoach/Bluesky integrations
- **Meetup**: Individual events with datetime, location, capacity, linked to Groups
- **Rsvp**: Links Users to Meetups for attendance tracking
- **GroupMembership**: Many-to-many between Users and Groups with roles (member, organizer, owner)

### Code Patterns
- **Snowflake IDs**: Custom ID generation with `glhd/bits` package
- **Tab Indentation**: This codebase uses tabs, not spaces (configured in .php-cs-fixer.dist.php)
- **Enums**: Extensive use of PHP enums for statuses and types

### Third-Party Integrations
- **Mailcoach SDK**: Each group can connect to their own Mailcoach instance for newsletters
- **Bluesky API**: Social media posting for meetup announcements
- **Laravel Forge SDK**: Domain management automation
- **Cloudflare**: CDN and security through `monicahq/laravel-cloudflare`

## Database & Testing
- **Production**: MySQL database
- **Testing**: SQLite in-memory (configured in phpunit.xml)
- **Migrations**: Comprehensive migration system with soft deletes on core models
- **Factories**: Model factories available for all core models
</project-guidelines>

<laravel-boost-guidelines>
=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to enhance the user's satisfaction building Laravel applications.

## Foundational Context
This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.3.25
- filament/filament (FILAMENT) - v3
- laravel/framework (LARAVEL) - v11
- laravel/prompts (PROMPTS) - v0
- laravel/socialite (SOCIALITE) - v5
- livewire/livewire (LIVEWIRE) - v3
- laravel/mcp (MCP) - v0
- laravel/pint (PINT) - v1
- laravel/sail (SAIL) - v1
- phpunit/phpunit (PHPUNIT) - v11
- alpinejs (ALPINEJS) - v3
- tailwindcss (TAILWINDCSS) - v3


## Conventions
- You must follow all existing code conventions used in this application. When creating or editing a file, check sibling files for the correct structure, approach, naming.
- Use descriptive names for variables and methods. For example, `isRegisteredForDiscounts`, not `discount()`.
- Check for existing components to reuse before writing a new one.

## Verification Scripts
- Do not create verification scripts or tinker when tests cover that functionality and prove it works. Unit and feature tests are more important.

## Application Structure & Architecture
- Stick to existing directory structure - don't create new base folders without approval.
- Do not change the application's dependencies without approval.

## Frontend Bundling
- If the user doesn't see a frontend change reflected in the UI, it could mean they need to run `npm run build`, `npm run dev`, or `composer run dev`. Ask them.

## Replies
- Be concise in your explanations - focus on what's important rather than explaining obvious details.

## Documentation Files
- You must only create documentation files if explicitly requested by the user.


=== boost rules ===

## Laravel Boost
- Laravel Boost is an MCP server that comes with powerful tools designed specifically for this application. Use them.

## Artisan
- Use the `list-artisan-commands` tool when you need to call an Artisan command to double check the available parameters.

## URLs
- Whenever you share a project URL with the user you should use the `get-absolute-url` tool to ensure you're using the correct scheme, domain / IP, and port.

## Tinker / Debugging
- You should use the `tinker` tool when you need to execute PHP to debug code or query Eloquent models directly.
- Use the `database-query` tool when you only need to read from the database.

## Reading Browser Logs With the `browser-logs` Tool
- You can read browser logs, errors, and exceptions using the `browser-logs` tool from Boost.
- Only recent browser logs will be useful - ignore old logs.

## Searching Documentation (Critically Important)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phpx-foundation/website](https://github.com/phpx-foundation/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
