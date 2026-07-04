---
trigger: always_on
description: Guidance for Claude Code when working with which-key-lazy.
---

# CLAUDE.md

Guidance for Claude Code when working with which-key-lazy.

## Commands

```bash
# Development
./gradlew compileKotlin                # Fast compile check
./gradlew runIde                       # Launch sandbox IDE with plugin loaded
./gradlew buildPlugin                  # Build distribution zip (build/distributions/)

# Testing
./gradlew test                         # Run all tests
./gradlew test --tests "*.DefaultConfigTest"           # Run one test class
./gradlew test --tests "*.IdeaVimRcParserTest.parses space leader"  # Run one test method
./gradlew compileTestKotlin            # Compile tests without running

# Verification
./gradlew verifyPlugin                 # Verify plugin structure
./gradlew verifyPluginProjectConfiguration  # Check build config

# Utilities
./gradlew tasks                        # List all available tasks
./gradlew dependencies                 # Show dependency tree
```

Add `--console=plain` for cleaner output. Add `JAVA_HOME` export if Gradle can't find JDK.

## Project Overview

**Which Key LazyVim-style** is a JetBrains IDE plugin that replicates the `which-key.nvim` popup from Neovim/LazyVim. It integrates with IdeaVim to show leader-key bindings hierarchically when the user presses a prefix key and pauses.

Activated via `set which-key` in `.ideavimrc`. No separate keybinding needed.

## Architecture

### Data Flow

```
Keystroke → WhichKeyActionListener (AnActionListener)
         → MappingLookup.getNestedEntries(keySequence, mode)
           ├── tryLeaderLookup (pre-built tree from IdeaVimApiReader)
           └── queryAllMappings (runtime query: user mappings + VIM_ACTIONS)
         → WhichKeyPopupManager.showPopup(entries)
         → WhichKeyPopup → WhichKeyPanel (renders)
```

### Package Layout

```
lazyideavim.whichkeylazy/
├── WhichKeyVimExtension.kt      # IdeaVim extension entry point (set which-key)
├── WhichKeyAction.kt            # Manual trigger action (Tools menu)
├── WhichKeyStartup.kt           # ProjectActivity for pre-loading config
├── WhichKeyPluginDisposable.kt  # App-scoped disposable
│
├── config/                      # Configuration & file management
│   ├── WhichKeyConfigService.kt # App service — rootBindings, settings, overrides
│   ├── WhichKeyConfig.kt        # JSON config loader (~/.whichkey-lazy.json)
│   ├── DefaultConfig.kt         # Default config template
│   ├── ConfigFileWatcher.kt     # Watches config file for changes
│   └── ReloadConfigAction.kt    # Reloads config
│
├── dispatch/                    # Key event handling & popup lifecycle
│   ├── WhichKeyActionListener.kt # Core keystroke observer
│   ├── MappingLookup.kt         # Key sequence → nested entries resolution
│   ├── WhichKeyPopupManager.kt  # Stateless popup lifecycle
│   └── ActionExecutor.kt        # Executes IntelliJ actions
│
├── ideavim/                     # IdeaVim integration & parsing
│   ├── IdeaVimApiReader.kt      # Preferred: reads mappings via IdeaVim runtime API
│   ├── IdeaVimRcParser.kt       # Fallback: parses ~/.ideavimrc file
│   ├── IdeaVimTreeBuilder.kt    # Converts flat mappings → KeyNode tree
│   ├── IdeaVimMapping.kt        # Data classes for parsed mappings
│   ├── DefaultGroupNames.kt     # LazyVim group name fallbacks (+git, +code, etc.)
│   └── KeyNodeMerger.kt         # Merges binding trees
│
├── model/                       # Data model
│   ├── KeyNode.kt               # Sealed class: GroupNode | ActionNode (String keys)
│   └── KeyBinding.kt            # Serialization models for JSON config
│
└── ui/                          # User interface & rendering
    ├── WhichKeyPanel.kt         # Custom JPanel — columnar layout rendering
    ├── WhichKeyPopup.kt         # JBPopup wrapper with keyboard navigation
    ├── BreadcrumbBar.kt         # Shows current path in key tree
    ├── IconResolver.kt          # Maps group descriptions → AllIcons
    └── WhichKeyColors.kt        # Theme-aware colors from editor scheme
```

### Key Design Decisions

- **String-based keys**: `KeyNode.key` is `String` (IdeaVim notation like `"f"`, `"<Tab>"`, `"<C-n>"`), not `Char`. This allows lossless representation of all key types including modifier combos.
- **Dual data source**: Prefers IdeaVim runtime API (`IdeaVimApiReader`), falls back to `.ideavimrc` file parsing (`IdeaVimRcParser` + `IdeaVimTreeBuilder`).
- **Runtime query + enrichment**: `MappingLookup` merges two sources — `queryAllMappings` (runtime, mode-aware, determines structure) and `tryLeaderLookup` (pre-built tree, provides richer icons/descriptions).
- **Stateless popup**: Popup is destroyed and recreated on every keystroke, matching idea-which-key's approach. No navigation stack.
- **Theme-aware**: All colors come from `EditorColorsManager.schemeForCurrentUITheme`.
- **Mode filtering**: Pre-built leader tree only used for NORMAL/VISUAL modes. INSERT mode relies solely on runtime query to avoid showing normal-mode mappings.

## IdeaVim Integration Notes

- Uses `injector.parser.toKeyNotation(keyStroke)` for lossless keystroke → string conversion
- Uses `injector.keyGroup.getKeyMapping(mode)` and `getAll(prefix)` for efficient prefix queries
- `<Plug>` and `<Action>` pseudo-keys (internal IdeaVim constructs) are filtered from the LHS of mappings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brandonkramer/which-key-lazy](https://github.com/brandonkramer/which-key-lazy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
