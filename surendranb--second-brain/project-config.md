---
trigger: always_on
description: - Create feature branches: `feature/descriptive-name`
---

# Cursor Rules for brAIn Plugin Development

## Core Development Principles

### 1. **Always Follow GitHub Best Practices**
- Create feature branches: `feature/descriptive-name`
- Make focused, atomic commits with conventional commit messages
- Never push code without local testing first
- Build and test before every commit
- User must validate functionality before merging to main

### 2. **Development Workflow**
```
1. Create feature branch
2. Understand the problem clearly
3. Make incremental changes
4. Build and test after each change
5. Get user feedback early and often
6. Commit with descriptive messages
7. Only merge after user validation
```

### 3. **Code Architecture Rules**

#### **File Structure**
- `main.ts` - Core plugin logic, UI, analysis methods
- `settings.ts` - Settings UI and configuration 
- `prompts.ts` - Prompt templates and constants
- Keep methods focused and single-responsibility

#### **Settings Pattern**
- All configurable options go in `PluginSettings` interface
- Provide sensible defaults in `DEFAULT_SETTINGS`
- Settings UI should have clear labels and descriptions
- Include "Reset to Default" buttons for complex settings
- Use helper methods for default value generation

#### **UI Development**
- Prefer simple, always-visible inputs over complex toggles
- Use clear, descriptive labels and placeholders
- Start with empty/clean states, not confusing defaults
- Get user feedback on UX before finalizing
- Make UI intuitive without documentation
- **Spacing is critical** - Use proper margins, padding, and visual separators
- **Respond to "ugly" feedback immediately** - User comfort trumps feature complexity
- **Avoid cramped layouts** - Elements need breathing room to look professional
- **Use consistent spacing hierarchy** - Follow established standards for margins/padding

#### **Prompt Engineering**
- Use placeholder injection: `{PLACEHOLDER}` format
- Create reusable prompt injection helpers
- Separate base prompts from dynamic content
- Always include JSON format validation in prompts
- Test with edge cases that might break JSON parsing

### 4. **Error Handling & Robustness**

#### **JSON Parsing**
- **Always implement multi-layer fallback strategies** - See Hierarchy Choice Dialog feature for complete pattern
- Use progressive approach: direct parse → enhanced cleaning → smart truncation → aggressive repair
- Optimize model configuration for reliability (lower temperature, reduced max_tokens)
- Add explicit length constraints to all AI prompts
- Log parsing errors with response content for debugging
- Handle malformed AI responses gracefully with fallback JSON structures
- Test with inputs that might break JSON structure (technical terms, quotes, long responses)

#### **Debug Logging**
- Add console logging for critical paths
- Use consistent log prefixes: `[ComponentName]`
- Include emojis for visual scanning: `🔧 🎯 📝 ✅ ❌`
- Log user inputs and processing steps
- Remove debug logs after feature validation

### 5. **Testing Strategy**

#### **Before Every Build**
- Test core functionality still works
- Test new feature with multiple input types
- Verify settings save and load correctly
- Check console for errors or warnings

#### **User Validation Process**
1. Build plugin: `npm run build`
2. User restarts Obsidian
3. User tests specific functionality
4. Check console logs for validation
5. Test edge cases and error scenarios
6. Only proceed after user confirms working

### 6. **Commit Message Standards**

#### **Format**: `type: description`
**Types:**
- `feat:` - New features
- `fix:` - Bug fixes  
- `refactor:` - Code improvements
- `debug:` - Temporary debugging code
- `docs:` - Documentation
- `style:` - UI/UX improvements

#### **Examples:**
```
feat: Add configurable analysis prompts to settings
fix: Resolve JSON parsing error with additional instructions  
refactor: Simplify Additional Instructions UI
debug: Add console logging to validate prompt injection
style: improve model selection UI with better spacing and layout
```

#### **Multi-line Commit Messages for Complex Changes:**
```
style: improve model selection UI with better spacing and layout

- Move model dropdown below radio buttons instead of inline
- Add proper spacing with 12px top margin and 18px bottom margin  
- Add visual separation with subtle border-top
- Increase dropdown width to 300px for better readability
- Use normal text color and improved font sizes
- Create clean section division between input modes and content inputs
```

### 7. **Feature Development Pattern**

#### **Phase 1: Foundation**
- Identify the core problem/bug
- Update interfaces and data structures
- Add default configurations
- Create basic functionality

#### **Phase 2: Implementation** 
- Implement core logic changes
- Add helper methods and utilities
- Update existing methods to use new system
- Ensure backward compatibility

#### **Phase 3: UI/UX**
- Create or update settings UI
- Improve main interface based on user feedback
- Add clear labels, help text, placeholders
- Test usability with user

#### **Phase 4: Robustness**
- Add error handling and fallbacks
- Include debug logging for validation
- Test edge cases and error scenarios
- Clean up temporary debugging code

### 8. **Common Patterns & Solutions**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [surendranb/second-brAIn](https://github.com/surendranb/second-brAIn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
