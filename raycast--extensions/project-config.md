---
trigger: always_on
description: You are tasked with ensuring that pull requests (PRs) submitted to the Raycast Extensions repository comply with the following guidelines and standards. Use these instructions to provide consistent, helpful feedback to contributors.
---

# GitHub Copilot Review Instructions for Raycast Extensions

You are tasked with ensuring that pull requests (PRs) submitted to the Raycast Extensions repository comply with the following guidelines and standards. Use these instructions to provide consistent, helpful feedback to contributors.

## General Guidelines for Suggestions

When providing feedback:
- **Always provide specific code suggestions** instead of general advice
- **Include exact code snippets** that can be copy-pasted
- **Use proper JSON/TypeScript formatting** without unnecessary character escaping
- **Be actionable** - show exactly what needs to be changed
- **Reference documentation** when applicable

For example, instead of saying "consider adding a subtitle", provide the exact JSON property to add: `"subtitle": "ServiceName"`

## 1. CHANGELOG.md Entries

### Required Format
- **Title Format**: New entries in `CHANGELOG.md` files MUST end with the template string `{PR_MERGE_DATE}`. 
  - ✅ Correct: `## [Bug Fix] - {PR_MERGE_DATE}`
  - ❌ Incorrect: `## [Bug Fix] - 2024-07-12`
- **Do NOT suggest** editing or replacing the `{PR_MERGE_DATE}` string if it is already present
- **If missing**: Provide this exact format to add: `## [Description] - {PR_MERGE_DATE}`

When the format is missing, suggest adding this exact line at the top of the CHANGELOG.md file:
```markdown
## [Your Change Description] - {PR_MERGE_DATE}
```

### Positioning and Content
- Ensure new entries are added **at the top** of the `CHANGELOG.md` file (newest entries first)
- Only suggest changing the title in `[XX]` if there is a spelling error
- The format must be: `## [Description] - {PR_MERGE_DATE}`

### Examples
```markdown
# Extension Changelog

## [New Feature] - {PR_MERGE_DATE}
- Added support for dark mode
- Fixed memory leak in background processes

## [Bug Fix] - 2024-03-15
- Fixed crash when loading preferences
```

## 2. package.json Updates

### Command Name Changes
When reviewing updates to existing extensions, check if command names have changed in `package.json`. If so, provide this specific comment:

```markdown
Can we keep the old `name`? It is the unique ID of the command where ranking, aliases, and hotkeys are saved. You can, of course, change the title :slightly_smiling_face:.
```

### Author Field Changes
If the `author` field in `package.json` changes, provide this specific comment:

```markdown
I notice the author field has changed from "previous-author" to "new-author". Can you confirm this change is intentional? This affects the extension's attribution in the Raycast Store.
```

### New Extension Requirements
For PRs labeled `new extension`:
- Verify the extension includes a `metadata` folder with screenshots if there are any `view` commands in `package.json`
- If missing, provide this exact feedback:

```markdown
This extension needs a `metadata` folder with screenshots since it includes view commands. Please see the [Raycast Documentation - Prepare Extension for Store](https://developers.raycast.com/basics/prepare-an-extension-for-store#how-to-use-it) for details on adding the required metadata.
```

- **Only mention this** if the PR has the `new extension` label

## 3. Error Handling

### launchCommand Usage
Ensure `launchCommand` is always wrapped in a try-catch block. When you find unwrapped `launchCommand` calls, provide this specific suggestion:

**Replace this:**
```typescript
await launchCommand({ name: "command-name", type: LaunchType.UserInitiated });
```

**With this:**
```typescript
try {
  await launchCommand({ name: "command-name", type: LaunchType.UserInitiated });
} catch (error) {
  console.error("Failed to launch command:", error);
}
```

### Lists and Grids
Lists and Grids should use `isLoading` to avoid empty state flicker. When you find Lists or Grids without proper loading states, provide this specific suggestion:

**Replace this:**
```typescript
<List>
  {isLoading ? null : items.map(item => <List.Item key={item.id} title={item.title} />)}
</List>
```

**With this:**
```typescript
<List isLoading={isLoading}>
  {items.map(item => <List.Item key={item.id} title={item.title} />)}
</List>
```

Reference: [Raycast Documentation - Empty States](https://developers.raycast.com/basics/prepare-an-extension-for-store#empty-states)

### getSelectedText() Usage
Ensure `getSelectedText()` has graceful error handling for cases where no text is selected or the operation fails. When you find unhandled `getSelectedText()` calls, suggest wrapping them like this:

```typescript
try {
  const selectedText = await getSelectedText();
  if (!selectedText) {
    await showToast({
      style: Toast.Style.Failure,
      title: "No text selected",
      message: "Please select some text first"
    });
    return;
  }
  // Use selectedText...
} catch (error) {
  await showToast({
    style: Toast.Style.Failure,
    title: "Failed to get selected text",
    message: error instanceof Error ? error.message : "Unknown error"
  });
}
```

### Toast Error Handling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raycast/extensions](https://github.com/raycast/extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
