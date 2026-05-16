---
trigger: always_on
description: This module integrates FreeScout (help desk system) with GitHub to enable support teams to create, link, and track GitHub issues directly from support conversations. The module uses AI to generate issue content based on conversation threads and provides bidirectional synchronization between FreeScout and GitHub.
---

# FreeScout GitHub Integration Module

## Project Overview

This module integrates FreeScout (help desk system) with GitHub to enable support teams to create, link, and track GitHub issues directly from support conversations. The module uses AI to generate issue content based on conversation threads and provides bidirectional synchronization between FreeScout and GitHub.

## Module Structure Review (2025-07-24)

### Current Status
The module follows FreeScout's modular structure correctly with the following observations:

#### ✅ Properly Implemented:
1. **Module Configuration**: 
   - `module.json` correctly defines the module with proper namespace and service provider
   - `composer.json` has correct PSR-4 autoloading configuration
   - `start.php` follows the standard pattern

2. **Service Provider**: 
   - `GithubServiceProvider.php` properly extends Laravel's ServiceProvider
   - Correctly registers hooks using `\Eventy` for integration points
   - Implements settings management, asset loading, and sidebar integration

3. **Database Structure**:
   - Migration files are properly structured
   - Tables follow naming conventions (`github_issues`, `github_issue_conversation`, `github_label_mappings`)

4. **Directory Structure**:
   - Follows Laravel/FreeScout module conventions
   - All required directories are present (Config, Database, Entities, Http, Providers, Resources, Services)

#### ⚠️ Issues Found:

1. **Missing Entity Class**: 
   - The `GithubIssueConversation` entity class is missing despite having a migration for the `github_issue_conversation` table
   - This entity is needed for proper relationship management between issues and conversations

2. **Module Alias Constant**:
   - The Jira module defines a module alias constant (`define('JIRA_MODULE', 'jira')`) in the ServiceProvider
   - The GitHub module should follow this pattern for consistency

3. **Missing View Files**:
   - The `ajax_html` directory is empty - needs modal views for issue creation/linking
   - Based on the Jira module, we need at least `link_issue.blade.php`

4. **Service Provider Differences**:
   - Jira module includes helper constants for API methods and configuration
   - Jira module has static properties for caching user/meta data
   - Consider adding similar patterns for consistency

## Key Features

### Core Functionality
- **AI-Powered Issue Creation**: Generate GitHub issue titles and descriptions using AI analysis of support conversation threads
- **Intelligent Label Assignment**: Automatically assign GitHub labels based on FreeScout conversation tags or AI analysis of conversation content
- **Issue Linking**: Link existing GitHub issues to FreeScout conversations
- **Sidebar Integration**: Display related GitHub issues in conversation sidebar with status indicators
- **Bidirectional Sync**: Receive GitHub webhook notifications to update FreeScout when issue status changes
- **Conversation Status Sync**: Automatically update FreeScout conversation status when linked GitHub issues are closed

### User Experience
- **Modal-based UI**: Create/link issues via modal dialogs without leaving the conversation
- **Real-time Search**: Search existing GitHub issues with autocomplete
- **Visual Status Indicators**: Show issue status, labels, and assignees in sidebar
- **Automatic Notes**: Add system notes to conversations when issues are created or status changes

## Technical Architecture

### Based on FreeScout Jira Module Patterns
- **Laravel Modular Structure**: Service provider, entities, controllers, views
- **Event-Driven Integration**: Uses `\Eventy` hooks for FreeScout integration
- **AJAX-based Frontend**: Progressive enhancement with graceful degradation
- **Webhook Support**: GitHub webhooks for real-time status updates

### Database Schema
```sql
-- Main GitHub issues table (cached data)
github_issues:
- id (primary key)
- number (GitHub issue number)
- repository (owner/repo format)
- title (cached from GitHub)
- state (open/closed)
- labels (JSON array)
- assignees (JSON array)
- created_at, updated_at

-- Junction table for issue-conversation relationships
github_issue_conversation:
- id (primary key)
- github_issue_id (foreign key)
- conversation_id (foreign key)
- unique constraint on (github_issue_id, conversation_id)
```

## Implementation Progress (2025-07-25)

### ✅ Completed Implementation:

#### Phase 1: Module Foundation
1. **Module Structure Setup** ✓
   - Laravel modular structure created following FreeScout patterns
   - Service provider, configuration, and routing all properly implemented
   - Database migrations for entities created and functional
   - All required module files (module.json, composer.json, start.php) properly configured

2. **GitHub API Integration** ✓
   - Comprehensive GitHub API client implemented with authentication
   - Full CRUD operations for issues (create, read, update, search)
   - Advanced repository discovery (personal, organization, and installation repos)
   - Rate limiting and error handling implemented

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [verygoodplugins/freescout-github](https://github.com/verygoodplugins/freescout-github) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
