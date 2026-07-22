---
trigger: always_on
description: ArkUI-X is a cross-platform UI framework that enables development of applications running on HarmonyOS, Android, and iOS from a single codebase. It provides a declarative UI framework using ArkTS (TypeScript-based) with native performance through C++ rendering engines.
---

## Project Overview

ArkUI-X is a cross-platform UI framework that enables development of applications running on HarmonyOS, Android, and iOS from a single codebase. It provides a declarative UI framework using ArkTS (TypeScript-based) with native performance through C++ rendering engines.

## Knowledge Base

This project maintains a comprehensive knowledge base system for in-depth component analysis and development guidance.

**Location**: `knowledge/`

### Quick Reference


| Topic               | Document                                        |
| ------------------- | ----------------------------------------------- |
| System Architecture | `knowledge/architecture/system_architecture.md` |
| Plugin Adaptation   | `knowledge/plugin_adaptation/adapter_guide.md`  |
| Development Guide   | `knowledge/best_practices/development_guide.md` |

### Usage Guidelines

When answering questions or providing guidance:

1. **Check for relevant knowledge base documents** before diving into code analysis
2. **Search the knowledge base** using Grep tools to find component-specific information
3. **Reference knowledge base content** to provide comprehensive, context-aware answers
4. **Cross-reference with actual code** using the file paths and line numbers cited in knowledge base documents

> **Complete structure**: See [Knowledge Base README](knowledge/knowledge_base_README.md) for full documentation structure.

## Core Working Principles

### 1. Code Verification: Actual Code Only

When answering questions about ArkUI-X Crossplatform code:

- **Always provide actual code from the repository**
  - Use Read/Grep tools to locate and read actual source code
  - Reference complete file paths when mentioning code (e.g., `frameworks/core/xxx/yyy.cpp:123`)
  - Never guess or fabricate code implementations
- **Missing information triggers user feedback**
  - If required code is not found in current project, explicitly state: "此代码在当前工程中未找到"
  - Do not make assumptions or write hypothetical code
  - Ask user to provide the missing implementation

**Example**:

```cpp
// ✅ Correct: Read actual source
// Source: frameworks/core/components_ng/pattern/menu/menu_pattern.cpp:123
Size GetSubWindowSize(int32_t parentContainerId, uint32_t displayId)
{
    auto finalDisplayId = displayId;
    auto defaultDisplay = Rosen::DisplayManager::GetInstance().GetDisplayById(displayId);
    // ... actual implementation
}

// ❌ Wrong: Fabricated code
// Do not write hypothetical implementations without verification
```

### 2. Speculation Management

When dealing with uncertain or incomplete information:

- **Explicitly label speculation**
  - Clearly mark any unverified content as: "推测" (speculation)
  - Provide reasoning for the speculation when possible
  - Request user confirmation for speculative statements
- **Verify before implementation**
  - If speculating about behavior, first use Grep/Read to verify
  - If implementation is not found in ace_engine, ask user to provide it
  - Never implement based on speculation alone

**Example**:

```markdown
✅ Correct:
基于 menu_pattern.cpp:123 的分析，推测 OnModifyDone 在以下情况下会被调用...（推测）

❌ Wrong:
OnModifyDone 会在以下情况下调用...（未标注推测）
```

### 3. Code Logic Verification: Code Over Suggestions

When receiving suggestions or corrections:

- **Verify suggestions against actual code**
  - User suggestions about code logic may be incorrect
  - Always verify with Read/Grep tools before accepting suggestions
  - Base conclusions on actual code behavior, not assumptions
- **Evidence-based reasoning**
  - When user questions code behavior, analyze actual implementation
  - Use "代码为准原则" (Code-first principle)
  - Provide evidence from source code to support or refute suggestions

**Example**:

```
User: "这个函数应该在初始化时调用，对吗？"
Claude: 让我先查看源码验证...
[Read source file]
根据 frameworks/xxx/yyy.cpp:456，该函数实际上是在 OnDirtyLayoutWrapperSwap 时调用，而非初始化时。
```

### 4. Error Learning: Knowledge Base Updates

When errors are corrected by users:

- **Learn from corrections**
  - Document the error and correction in knowledge base
  - Identify root cause of the misunderstanding
  - Add preventive measures to avoid similar errors
- **Update documentation**
  - Create or update knowledge base entries with correct information
  - Reference actual code locations (file:line)
  - Share lessons learned across sessions

**Example**:

```markdown
## Learned Lessons

### Error: Incorrect GetSubWindowSize Branch Coverage
**Date**: 2025-01-27
**Issue**: Assumed defaultDisplay is always available
**Root Cause**: Did not verify Rosen::DisplayManager dependency
**Correction**: User clarified that mock infrastructure is required
**Prevention**: Always verify external dependencies with user before assuming availability
**Reference**: adapter/ohos/entrance/subwindow/subwindow_ohos.cpp:199-243
```

### 5. Knowledge Base Maintenance

When discovering discrepancies between documentation and actual code:

- **Verify actual code first**
  - Use Read/Grep to confirm current implementation
  - Compare documented behavior with actual code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arkui-x/docs](https://github.com/arkui-x/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
