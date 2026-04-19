---
trigger: always_on
description: Development stack designed for AI agents to build Nostr-enabled Flutter applications. It includes a complete tech stack with Purplebase and Riverpod, documentation and recipes for common implementation scenarios.
---

# Purplestack

Development stack designed for AI agents to build Nostr-enabled Flutter applications. It includes a complete tech stack with Purplebase and Riverpod, documentation and recipes for common implementation scenarios.

## CRITICAL FOR AI ASSISTANTS - MUST READ FIRST

**Reference**. The AI assistant's behavior and knowledge is defined by this context file (aliased to other IDE-specific filenames), which serves as the system prompt. **ALWAYS** refer to this document and follow these rules and recommendations.

### Purplestack MCP Server

**CRITICAL**: This project relies on the purplestack MCP server for essential recipes and `models` package documentation. **YOU MUST VERIFY THE MCP SERVER IS WORKING BEFORE IMPLEMENTING ANY FEATURE**. If the purplestack MCP server cannot be called or returns 0 tools, there is a configuration issue. Fixes:
 - Run `flutter pub get`, then you must prompt the user to reload their IDE/environment
 - the `agent.json` (or `mcp.json` or equivalent) file may need to be modified to include the correct "cwd" pointing to the current project directory for the purplestack MCP server configuration, and/or the full path to the `dart` executable in the system

Available purplestack MCP tools:
- `list_recipes` - List all available implementation recipes  
- `read_recipe` - Read a specific recipe by name
- `search_recipes` - Search recipes by keyword
- `list_docs` - List all available documentation
- `read_doc` - Read specific documentation
- `search_docs` - Search documentation by keyword

**Usage Requirements:**
- **BEFORE IMPLEMENTING ANY FEATURE**: Check for relevant recipes using `search_recipes` **AND** API documentation using `search_docs`
- **For ANY code implementation**: Consult both recipes and documentation first
- **Always search first**: Use `search_recipes` and `search_docs` before writing any code. Always prioritize looking for documentation (`search_docs`) and recipes (purplestack MCP) before calling the nostr MCP
- If no recipes or docs are found for your use case, proceed with standard implementation
- Recipes are complete examples showing how to approach specific features

### Nostr MCP Server

The `nostr` MCP server provides Nostr protocol reference and documentation for understanding NIPs, event kinds, and protocol specifications. **It is only really useful when implementing a custom model**.

### CRITICAL: First prompt checklist

**First prompt definition**: A prompt is considered "first" if the app has not been renamed yet (i.e., app ID is still "com.example.purplestack"). This is determined by app state, not session state.

When handling a first prompt (as defined above), you MUST ALWAYS:

1. Run `fvm flutter pub get` (if fvm available), or `flutter pub get`
2. Initialize a fresh git repository **IF AND ONLY IF THE FIRST GIT COMMIT HASH IN THE REPOSITORY IS `c99f911fa1c2fd25e6a401f146ae5e0f53c459b1`:**
   - `rm -rf .git/` (remove existing git history)
   - `git init` (start fresh repository)
   - Make sure you are NOT deleting commits by the user!
3. Suggest the user an app name, app ID (e.g., `com.sample.app`) and description. Based on obtained information, you MUST call the `rename_app` tool
  - `dart tools/scripts/rename_app.dart --name "Your App Name" --app-id "com.sample.app"` – full reference available via purplestack MCP server: `search_recipes renaming`.
4. Summarize what you understood, and ask important questions to ensure all requirements are crystal clear before starting any code
5. Ask them which outputs they want. By default, this project builds an Android app, but Flutter supports iOS, MacOS, Linux and Windows, all of these are preconfigured. **Remove the whole folders** of platforms that the user does not care about.

**IF THE FLUTTER EXECUTABLE OR THE PURPLESTACK MCP TOOLS ARE NOT AVAILABLE YOU _MUST_ ABORT EVERYTHING _RIGHT NOW_. THAT DOES NOT MEAN LATER, IT MEANS NOW. NO "WE'LL TAKE CARE OF IT LATER". ABORT RIGHT NOW.**

## Technology Stack

- **Flutter**: Cross-platform UI framework
- **Dart**: Programming language
- **Riverpod / hooks_riverpod**: State management and dependency injection
- **Flutter Hooks**: React-style hooks for Flutter
- **models**: Domain models for Nostr events
- **Purplebase**: Local-first Nostr SDK with storage and relay pool, implementation of the `models` package interface
- **amber_signer**: NIP-55 Android signer integration
- **GoRouter**: Declarative routing
- **google_fonts**: Font management
- **fluttertoast**: Toast components
- **cached_network_image**: Image loading and caching
- **flutter_markdown**: Markdown rendering
- **auto_size_text**: Responsive text sizing
- **skeletonizer**: Skeleton loading states
- **percent_indicator**: Progress indicators
- **easy_image_viewer**: Image viewing
- **flutter_layout_grid**: Grid layouts
- **table_calendar**: Highly customizable, feature-packed calendar widget
- **dart_emoji**: Emoji support and parsing
- **any_link_preview**: Link preview generation
- **async_button_builder**: Async button states and interactions
- **path_provider**: Platform-specific directory paths

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/purplebase) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
