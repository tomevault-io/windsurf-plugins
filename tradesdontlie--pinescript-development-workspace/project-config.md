---
trigger: always_on
description: - **Starting** a new feature/modification
---

# Automatic Versioning Rules - Intelligent & Balanced

## Rule: Smart Auto-Versioning (Not Every Change!)

### 🎯 Version ONLY at Key Points:

#### DO Version When:
- **Starting** a new feature/modification
- **Completing** a feature successfully  
- **Before** major refactoring
- **After** significant progress (multiple functions added)
- **Failed** attempt that needs rollback point
- **Ending** a work session
- User **explicitly asks** for version

#### DON'T Version When:
- Fixing a typo
- Adjusting a single parameter
- Adding a comment
- Minor formatting changes
- In rapid iteration mode
- Testing different values

### 📊 Intelligent Version Detection

```mermaid
flowchart TD
    A[User Message Received] --> B{Parse Intent}
    
    B -->|"Add [feature]"| C[🆕 START: Version + Track]
    B -->|"That didn't work"| D[⏪ ROLLBACK: Version before revert]
    B -->|"Perfect, that works!"| E[✅ SUCCESS: Capture stable state]
    B -->|"I'm done for now"| F[🔚 END: Final session version]
    B -->|"Save this" / "Version this"| G[💾 EXPLICIT: User request]
    B -->|Minor edits/tweaks| H[⚡ CONTINUE: No versioning needed]
    
    C --> I[📦 Create Minor Version]
    D --> J[📦 Create Rollback Point]
    E --> K[📦 Create Patch Version]
    F --> L[📦 Create Session Version]
    G --> M[📦 Create Requested Version]
    H --> N[🔨 Continue Coding]
    
    style C fill:#e8f5e8
    style D fill:#fff3e0
    style E fill:#e8f5e8
    style F fill:#e8f5e8
    style G fill:#e8f5e8
    style H fill:#f3e5f5
```

**Trigger Phrase Examples:**
- 🆕 **Start**: "Add weekly filter to FVG" → Version ONCE at start
- ⏪ **Rollback**: "That didn't work" → Version before rollback  
- ✅ **Success**: "Perfect, that works" → Version to capture success
- 🔚 **End**: "I'm done for now" → Final version
- 💾 **Explicit**: "Save this" / "Version this" → User request

### 🔄 Workflow Integration

```mermaid
flowchart LR
    subgraph "✅ SMART PATTERN - Efficient Versioning"
        A1[Start Feature] --> B1[📦 Version]
        B1 --> C1[🔨 Code Code Code]
        C1 --> D1{Error?}
        D1 -->|Yes| E1[🔧 Fix]
        D1 -->|No| F1[🔨 Code More]
        E1 --> F1
        F1 --> G1[✅ Complete]
        G1 --> H1[📦 Final Version]
    end
    
    subgraph "❌ BAD PATTERN - Version Spam"
        A2[Start] --> B2[📦 Version]
        B2 --> C2[🔨 Code]
        C2 --> D2[📦 Version]
        D2 --> E2[🔧 Tweak]
        E2 --> F2[📦 Version]
        F2 --> G2[🔧 Fix]
        G2 --> H2[📦 Version]
    end
    
    style A1 fill:#e8f5e8
    style H1 fill:#e8f5e8
    style A2 fill:#ffebee
    style B2 fill:#ffebee
    style D2 fill:#ffebee
    style F2 fill:#ffebee
    style H2 fill:#ffebee
```

**Result Comparison:**
- ✅ **Smart Pattern**: 2 versions for entire feature (clean history)
- ❌ **Bad Pattern**: 4+ versions for simple feature (cluttered history)

### 📁 Working Directory Management

Keep working directory clean:
- Version at milestones, not every change
- Use descriptive version increments:
  - **patch**: Bug fixes, error corrections
  - **minor**: New features, enhancements
  - **major**: Breaking changes, rewrites

### 🎮 Practical Examples

#### Good Versioning Flow:
```
You: "Add weekly filter to FVG"
Me: [VERSION] fvg_v0.1.0 → fvg_v0.2.0 (minor - new feature)
    [CODE] Implement filter
    [ERROR] Fix HTF issue
    [CODE] Complete implementation
You: "Great, that works"
Me: [VERSION] fvg_v0.2.0 → fvg_v0.2.1 (patch - stabilized)
Total: 2 versions for complete feature
```

#### Bad Versioning Flow:
```
You: "Add weekly filter"
Me: [VERSION] v0.1.0 → v0.2.0
    [CODE] Add basic structure
Me: [VERSION] v0.2.0 → v0.2.1
    [CODE] Add timeframe check
Me: [VERSION] v0.2.1 → v0.2.2
    [ERROR] Fix issue
Me: [VERSION] v0.2.2 → v0.2.3
Total: Too many versions for one feature!
```

### 🤖 My Decision Process

```mermaid
flowchart TD
    A[User Request Received] --> B{New Feature/Task?}
    B -->|YES| C[🧠 AI CHOOSES APPROACH<br/>• Version if valuable<br/>• Track if helpful<br/>• Start immediately if simple]
    B -->|NO| D[Continue Current Work]
    
    C --> E[🔨 AI Develops Intelligently<br/>Full tool freedom]
    D --> F{Making Changes?}
    
    F -->|YES| G{Changes SIGNIFICANT?}
    F -->|NO| H[💭 AI Codes Freely<br/>Any tools needed]
    
    G -->|YES| I{User Mentioned<br/>Completion/Save?}
    G -->|NO| H
    
    I -->|YES| J[📦 AI DECIDES VERSION METHOD<br/>• Script if complex tracking needed<br/>• Direct file ops if simpler<br/>• Whatever works best]
    I -->|NO| K[⏳ Keep Working]
    
    H --> L[✅ Continue Development<br/>AI uses optimal tools]
    J --> L
    K --> L
    
    style A fill:#e3f2fd
    style C fill:#e8f5e8
    style J fill:#e8f5e8
    style H fill:#fff3e0
    style L fill:#f3e5f5
```

**Quick Decision Reference:**
- 🆕 NEW task/feature → Always version first
- 📝 Minor edits → Just code, no version
- ✅ "That works!" → Version success
- 🔚 "I'm done" → Final version

### 📝 Version Commit Messages

When creating versions, use clear indicators:
```bash
# Feature start
./scripts/pine_master.sh develop file minor  # "Starting: [feature name]"

# Feature complete  
./scripts/pine_master.sh develop file patch  # "Completed: [feature name]"

# Failed attempt
./scripts/pine_tracker.sh rollback file version "Reason for rollback"
```

### 🎯 Key Principles


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tradesdontlie/pinescript-development-workspace](https://github.com/tradesdontlie/pinescript-development-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
