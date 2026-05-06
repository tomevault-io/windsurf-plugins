---
trigger: always_on
description: This document contains important patterns and conventions for developing the `hatchet` TUI application.
---

# Hatchet Development Guide

This document contains important patterns and conventions for developing the `hatchet` TUI application.

## Project Structure

- `src/main.ts` - Main application with all views
- `src/helpers/git.ts` - Git worktree operations
- `src/helpers/fizzy.ts` - Fizzy (task management) integration
- `src/helpers/project.ts` - Project detection and post-worktree hooks
- `src/helpers/html.ts` - HTML to TUI rendering
- `src/helpers/image.ts` - Image extraction and placeholders
- `src/helpers/terminal.ts` - Terminal launcher utilities
- `src/helpers/card-tile.ts` - Fizzy card tile component
- `src/helpers/pr-tile.ts` - GitHub PR tile component
- `src/helpers/github.ts` - GitHub PR integration (uses gh CLI)
- `src/helpers/config.ts` - Configuration file loading
- `src/helpers/cli.ts` - CLI argument parsing (yargs)
- `src/helpers/protocol-handler.ts` - Protocol handler installation logic
- `src/theme.ts` - Color theming system
- `src/types.ts` - TypeScript type definitions
- `scripts/install-protocol-handler.sh` - Protocol handler installer (bash, legacy)
- `chrome-extension/` - Chrome extension for Fizzy integration

## OpenTUI Key Event Handling

### The Enter Key Bleed-Through Problem

When navigating between views in OpenTUI, there's a critical timing issue with Enter key handling.

**Problem**: When pressing Enter on a `SelectRenderable`:
1. The `ITEM_SELECTED` event fires
2. Handler navigates to new view synchronously  
3. New view's `SelectRenderable` is focused in the same event loop tick
4. The same Enter keypress propagates to the new view and triggers its selection

**Solution**: Always use `process.nextTick()` to defer navigation:

```typescript
// WRONG - causes enter bleed-through
select.on(SelectRenderableEvents.ITEM_SELECTED, (_idx, opt) => {
  showNextView(renderer);
});

// CORRECT - defers to next tick
select.on(SelectRenderableEvents.ITEM_SELECTED, (_idx, opt) => {
  // Defer navigation to next tick to prevent enter key from bleeding through
  process.nextTick(() => showNextView(renderer));
});
```

This pattern must be applied to ALL `SelectRenderableEvents.ITEM_SELECTED` handlers that navigate to another view.

### Focus Deferral for New Views

When creating a new view with a `SelectRenderable`, also defer the focus call:

```typescript
// Defer focus to next tick to prevent Enter key from immediately triggering selection
setTimeout(() => {
  select.focus();
}, 0);
```

## View Navigation Pattern

Each view function follows this pattern:

1. Set `currentView` to track current state
2. Clear the root with `clearChildren(root)`
3. Build the new UI
4. Register key handlers (with cleanup functions)
5. Add content to root
6. Focus the primary interactive element (deferred with setTimeout)

## Key Handler Cleanup

Views that register custom key handlers must clean them up before navigating:

```typescript
let keyHandler: ((key: { name?: string }) => void) | null = null;

const cleanup = () => {
  if (keyHandler) {
    renderer.keyInput.off("keypress", keyHandler);
    keyHandler = null;
  }
};

// Use cleanup before any navigation
keyHandler = (key) => {
  if (key.name === "escape") {
    cleanup();
    showMainView(renderer);
  }
};
renderer.keyInput.on("keypress", keyHandler);
```

## Nerd Font Icons

The app uses Nerd Font icons via Unicode escapes:

```typescript
const ICONS = {
  branch: "\ue725",      // nf-dev-git_branch
  ahead: "\uf062",       // nf-fa-arrow_up
  behind: "\uf063",      // nf-fa-arrow_down
  clean: "\uf00c",       // nf-fa-check
  staged: "\uf067",      // nf-fa-plus
  modified: "\uf040",    // nf-fa-pencil
  untracked: "\uf128",   // nf-fa-question
  commit: "\uf417",      // nf-oct-git_commit
};
```

## Running and Testing

```bash
# Run the app (TUI mode)
bun hatchet

# Type check (ignore known issues in unused files)
npx tsc --noEmit 2>&1 | grep -v "src/app.ts\|src/views/"

# Toggle console for debugging (in app)
Press ` (backtick)
```

## Command-Line Interface

Hatchet supports both interactive TUI mode and non-interactive CLI mode.

### CLI Options

```bash
hatchet [options]

Options:
  -c, --card <number>    Fizzy card number to create/switch worktree for
      --pr <number>      GitHub PR number to create/switch worktree for
  -p, --path <dir>       Path to git repository (required for protocol handler)
  -o, --launch-opencode  Launch OpenCode in the worktree after creation
      --with-context     Include card/PR context in OpenCode prompt (requires -o)
  -l, --list             List worktrees and exit
      --install-handler  Install hatchet:// protocol handler (Linux)
  -h, --help             Show help
  -v, --version          Show version
```

### Examples

```bash
# Launch interactive TUI
hatchet

# Create/switch to worktree for card #123
hatchet --card 123

# Create/switch to worktree for PR #456
hatchet --pr 456

# Create worktree and launch OpenCode
hatchet --card 123 --launch-opencode
hatchet -c 123 -o
hatchet --pr 456 -o

# Create worktree and launch OpenCode with card/PR context in prompt
hatchet --card 123 --launch-opencode --with-context
hatchet -c 123 -o --with-context
hatchet --pr 456 -o --with-context

# Work with a specific repo (useful for protocol handler)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryanrhughes/hatchet](https://github.com/ryanrhughes/hatchet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
