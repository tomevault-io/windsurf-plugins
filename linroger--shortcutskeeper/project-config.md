---
trigger: always_on
description: Coordinates all .claude folder components to work together as an active intelligence enhancement system for software development and research projects.
---

# Claude Code Master Instructions

## System Integration Hub

### Purpose
Coordinates all .claude folder components to work together as an active intelligence enhancement system for software development and research projects.

### Active Components
- **ai_history/**: @.claude/ai_history/CLAUDE.md
- **claude_code/**: @.claude/claude_code/CLAUDE.md  
- **commands/**: @.claude/commands/CLAUDE.md
- **sub-agent-tasks/**: @.claude/sub-agent-tasks/CLAUDE.md

### Integration Protocol
Every session must actively use ALL components:

#### Knowledge → Tools → Commands Flow
1. **Check existing knowledge** before starting work
2. **Apply tool optimization** during execution
3. **Create automation commands** for repeated patterns
4. **Update knowledge base** with discoveries

#### Cross-Component Behaviors
- **Save conversations**: ai_history tracks all significant exchanges
- **Optimize workflows**: claude_code patterns guide tool usage
- **Delegate strategically**: sub-agent-tasks stores parallel execution outputs
- **Automate repetition**: commands folder stores custom slash commands
- **Evolve intelligently**: Each component learns from the others

#### System Questions
ALWAYS consider during work:
- "Should this be saved as knowledge?"
- "Can this be automated with a command?"
- "Is there relevant knowledge to apply?"
- "Can I delegate part of this for parallel execution?"
- "Should I check sub-agent outputs for context before proceeding?"

---

## Core Development Workflows

### 1. Native macOS App Development

#### 1.1 Initial Project Setup & Analysis

**ALWAYS Follow This Sequence:**
1. **Read existing project structure** - Use LS tool to understand current codebase organization
2. **Analyze CLAUDE.md files** - Read any project-specific instructions first
3. **Review existing Swift files** - Understand current architecture and patterns
4. **Identify the main app target** - Locate the primary .swift files and entry points

#### 1.2 SwiftUI App Architecture Approach

**UI Design Philosophy:**
- **Start with the data model** - Define your core data structures first
- **MVVM Architecture** - Always separate ViewModels from Views
- **SwiftData integration** - Use SwiftData for persistence, not Core Data
- **Native macOS design** - Use NavigationSplitView, sidebars, and native controls
- **Responsive layouts** - Design for different window sizes from the start

**UI Connection Pattern:**
```
Data Model → ViewModel (@Observable) → SwiftUI View → User Interface
```

**Key UI Principles:**
- **Use @Observable** for ViewModels (Swift 6 pattern)
- **@State for local view state** only
- **@Environment for shared data** across views
- **SF Symbols** for all icons
- **System colors** for native appearance

#### 1.3 Function-to-UI Connection Strategy

**Standard Connection Pattern:**
1. **ViewModel Methods** - All business logic goes in ViewModel
2. **View Actions** - Buttons and controls call ViewModel methods
3. **State Updates** - ViewModel updates trigger UI refresh automatically
4. **Error Handling** - Show alerts/sheets for errors from ViewModel

**Example Pattern:**
```swift
// In ViewModel
@Observable class MyViewModel {
    var items: [Item] = []
    var isLoading = false
    
    func performAction() {
        isLoading = true
        // Business logic here
        isLoading = false
    }
}

// In View
Button("Action") {
    viewModel.performAction()
}
.disabled(viewModel.isLoading)
```

#### 1.4 MANDATORY Build-Test-Fix Cycle

**ALWAYS Implement This Cycle:**

1. **After Every Code Change:**
   - Run `xcodebuild -scheme [ProjectName] build` to test compilation
   - If build fails, analyze errors and fix immediately
   - Never proceed to next feature until current code builds successfully

2. **Build Testing Commands:**
   ```bash
   # Clean build folder
   xcodebuild clean -scheme [ProjectName]
   
   # Build for macOS
   xcodebuild -scheme [ProjectName] -destination 'platform=macOS' build
   
   # Build and run tests
   xcodebuild test -scheme [ProjectName] -destination 'platform=macOS'
   ```

3. **Error Resolution Process:**
   - Read error messages carefully
   - Fix syntax/import errors first
   - Address deprecation warnings
   - Resolve missing dependencies
   - Fix architectural issues last

4. **Success Criteria:**
   - Zero build errors
   - Zero warnings (unless explicitly legacy)
   - App launches without crashes
   - Basic functionality works as expected

#### 1.5 Common macOS App Patterns You Should Always Use

**File Organization:**
```
App/
├── Models/          # Data models and business logic
├── ViewModels/      # @Observable view models
├── Views/           # SwiftUI views
├── Services/        # Network, persistence, external APIs
├── Extensions/      # Swift extensions
└── Resources/       # Assets, localizations
```

**Navigation Pattern:**
```swift
NavigationSplitView {
    // Sidebar content
} detail: {
    // Main content area
}
```

**Data Persistence Pattern:**
```swift
@Model
class MyDataModel {
    // SwiftData model definition
}

// In App file:
.modelContainer(for: MyDataModel.self)
```

### 2. Python Development Workflow

#### 2.1 Project Structure & Setup


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linroger/ShortcutsKeeper](https://github.com/linroger/ShortcutsKeeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
