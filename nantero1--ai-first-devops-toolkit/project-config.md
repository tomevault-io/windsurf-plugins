---
trigger: always_on
description: This lessons-learned file serves as a critical knowledge base for capturing and preventing mistakes. During development, document any reusable solutions, bug fixes, or important patterns. Consult it before implementing any solution.
---

*This lessons-learned file serves as a critical knowledge base for capturing and preventing mistakes. During development, document any reusable solutions, bug fixes, or important patterns using the format: Category: Issue → Solution → Impact. Entries must be categorized by priority (Critical/Important/Enhancement) and include clear problem statements, solutions, prevention steps, and code examples. Only update upon user request with "lesson" trigger word. Focus on high-impact, reusable lessons that improve code quality, prevent common errors, and establish best practices. Cross-reference with .cursor\memories.md for context.*

# Lessons Learned

*Note: This file is updated only upon user request and focuses on capturing important, reusable lessons learned during development. Each entry follows format: Priority: Category → Issue: [Problem] → Fix: [Solution] → Why: [Impact]. Use grep/automated tools to verify changes and prevent regressions.*

## Logging Best Practices

**Critical**: CLI Tool Logging Philosophy for CI/CD Environments
→ Issue: CLI tools need clean, predictable logging that serves both interactive users and CI/CD pipelines with clear level separation.
→ Solution: Follow strict PEP 282 pattern: INFO = user-facing progress and business events (branch selection, settings applied, major steps), DEBUG = technical details for developers, Rich output = actual results presentation separate from logging system.
→ Impact: Perfect CI/CD logs showing execution path and settings without noise, while preserving debugging capability.

**Critical**: Context-Dependent Logging Levels  
→ Issue: JSON parsing failure severity depends on context - ERROR when schema enforcement expected, DEBUG when optional fallback.
→ Solution: Map logging levels to user expectations: schema_model exists = ERROR (broken promise), no schema_model = DEBUG (expected behavior).
→ Impact: Proper error visibility for users, prevents confusion about system behavior.

**Important**: Eliminate Duplicate Success Messages in Execution Flow
→ Issue: Schema loading and ChatHistory creation logged at INFO level multiple times in single execution flow creating log noise and confusion.
→ Solution: Move technical success confirmations (schema creation, template rendering, chat history creation) to DEBUG level. Keep only business-relevant completion messages at INFO level.
→ Impact: Clean CI/CD output showing only meaningful progress, eliminates duplicate information, maintains debugging capability.

**Important**: Meta-Logging Should Be DEBUG Level
→ Issue: Logging initialization messages like "LLM Runner initialized with log level: INFO" are technical metadata, not user-facing progress.
→ Solution: Move logging configuration messages to DEBUG level - users care about what the tool does, not how it's configured to report.
→ Impact: Cleaner INFO output focused on actual business operations, follows CLI tool best practices.

**Enhancement**: Preserve User-Facing Rich Output Separate from Logging
→ Issue: Removing beautiful Rich console LLM response display during "logging improvements" degrades user experience.
→ Solution: Keep CONSOLE.print() for LLM responses completely separate from logging system - users expect to see their results beautifully formatted.
→ Impact: Maintains core value proposition of beautiful, readable output while having clean logging.

**Important**: Emoji Consistency Matches Log Level Severity
→ Issue: Using warning emojis (⚠️) in DEBUG messages creates confusion about severity.
→ Solution: Match emoji severity to log level: 🔄 for DEBUG fallbacks, ⚠️ for WARNING, ❌ for ERROR.
→ Impact: Clear visual communication of actual issue severity.

## Component Development

**Important**: Template Engine Unification Strategy    
→ Issue: Supporting both Handlebars and Jinja2 without breaking compatibility.    
→ Solution: Implement unified load_template() with file extension-based detection (.hbs, .jinja, .j2), separate loader functions, keep existing signatures.    
→ Impact: Enables multi-engine support, backward compatibility, and easy future extension.

**Critical**: Microsoft Semantic Kernel Integration Success Pattern    
→ Issue: Initially unclear how to make YAML model_id specifications actually control which Azure deployment is used by Semantic Kernel.    
→ Solution: Semantic Kernel selects services by service_id matching YAML execution_settings keys, not by model_id. Service's deployment_name determines actual Azure model called. Create services dynamically based on YAML model_id and use as deployment_name directly.    
→ Impact: Enables true YAML-driven model selection without hardcoded model lists, future-proof for any Azure deployment.

**Critical**: Semantic Kernel Service Registration Architecture  
→ Issue: YAML execution_settings.azure_openai.model_id was being ignored because service was pre-created with fixed deployment_name from environment.  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nantero1/ai-first-devops-toolkit](https://github.com/Nantero1/ai-first-devops-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
