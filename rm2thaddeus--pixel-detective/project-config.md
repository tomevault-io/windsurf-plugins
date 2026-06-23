---
trigger: always_on
description: Use these rules when unsure how to navigate project rules
---

# Smart Entry Points - Intelligent Rule Routing System

## 🎯 **Context-Aware Rule Activation**

This file defines automatic routing logic for AI assistants to apply the right rules based on work context.

---

## **Automatic Context Detection Rules**

### **File Pattern → Rule Mapping**

#### **Frontend Development Context**
```yaml
file_patterns:
  - "frontend/**/*.tsx"
  - "frontend/**/*.ts" 
  - "frontend/**/*.jsx"
  - "frontend/**/*.js"
  - "frontend/**/*.css"
  - "frontend/**/*.scss"

primary_rule: "@frontend-development-index"
secondary_rules:
  - "@nextjs-hydration-prevention"      # For React components
  - "@react-query-api-integration"      # For API calls
  - "@component-architecture-patterns"  # For component design
  - "@ux-workflow-patterns"            # For UX work
  - "@nextjs-performance-optimization" # For performance

prerequisites:
  - "@sprint-lessons-learned"  # Always check Sprint 10 lessons
  - "@mcp-browser-tools-setup" # Verify MCP setup for frontend work
```

#### **Backend Development Context**
```yaml
file_patterns:
  - "backend/**/*.py"
  - "src/**/*.py"
  - "**/*api*.py"
  - "**/*service*.py"
  - "**/*model*.py"

primary_rule: "@backend-development-index"
secondary_rules:
  - "@fastapi-dependency-injection"     # For API development
  - "@api-design-patterns"             # For API design
  - "@fastapi-microservice-patterns"   # For service architecture
  - "@ml-service-integration"          # For ML services

prerequisites:
  - "@sprint-lessons-learned"  # Always check Sprint 10 lessons
  - "@use-mcp-servers"        # For Supabase MCP usage
```

#### **Database Work Context**
```yaml
file_patterns:
  - "**/*.sql"
  - "**/migrations/**"
  - "**/database/**"
  - "**/schemas/**"

primary_rule: "@use-mcp-servers"
focus: "Supabase MCP operations"
secondary_rules:
  - "@api-design-patterns"  # For data modeling
  
prerequisites:
  - "@sprint-lessons-learned"  # Check for database-related lessons
```

#### **Testing Context**
```yaml
file_patterns:
  - "**/*.test.ts"
  - "**/*.test.tsx"
  - "**/*.test.py"
  - "**/*.spec.*"
  - "**/tests/**"

primary_rule: "@sprint10-testing-patterns"
secondary_rules:
  - "@nextjs-hydration-prevention"    # For frontend tests
  - "@fastapi-dependency-injection"   # For backend tests
  - "@mcp-browser-tools-setup"       # For MCP testing

prerequisites:
  - "@sprint-lessons-learned"  # Apply testing lessons learned
```

#### **Documentation Context**
```yaml
file_patterns:
  - "**/*.md"
  - "docs/**"
  - "README*"
  - "CHANGELOG*"

primary_rule: "@commentsoverwrite"
secondary_rules:
  - "@sprint-planning"        # For sprint documentation
  - "@cross-reference-guide" # For navigation docs

prerequisites:
  - Current sprint context for sprint-specific docs
```

#### **Sprint Planning Context**
```yaml
file_patterns:
  - "docs/sprints/**"
  - "**/PRD.md"
  - "**/sprint-*.md"

primary_rule: "@sprint-planning"
secondary_rules:
  - "@use-mcp-servers"        # For MCP integration in planning
  - "@commentsoverwrite"      # For documentation standards

prerequisites:
  - "@sprint-lessons-learned"  # Review lessons before planning
```

---

## **Emergency Context Detection**

### **Error-Based Auto-Routing**

#### **Hydration Errors** 
```yaml
error_indicators:
  - "hydration"
  - "Text content does not match server-rendered HTML"
  - "Warning: Text content did not match"
  - "Hydration failed"

emergency_rule: "@quick-troubleshooting-index"
specific_rule: "@nextjs-hydration-prevention"
recovery_pattern: "mounted state implementation"
```

#### **Import/Circular Dependency Errors**
```yaml
error_indicators:
  - "circular import"
  - "ImportError"
  - "cannot import name"
  - "partially initialized module"

emergency_rule: "@quick-troubleshooting-index"  
specific_rule: "@fastapi-dependency-injection"
recovery_pattern: "dependency injection refactor"
```

#### **MCP Connection Failures**
```yaml
error_indicators:
  - "Failed to discover browser connector server"
  - "Chrome extension not connected"
  - "MCP server not responding"
  - "Browser Tools server"

emergency_rule: "@mcp-browser-tools-setup"
recovery_pattern: "3-component verification protocol"
```

#### **PowerShell Syntax Errors**
```yaml
error_indicators:
  - "The term '&&' is not recognized"
  - "Unexpected token '&&'"
  - "Cannot run program"
  - "PATH environment variable"

emergency_rule: "@powershell-syntax"
recovery_pattern: "auto-safe command templates"
auto_fix: "split compound commands into separate run_terminal_cmd calls"
```

---

## **Work Type → Entry Point Routing**

### **🚨 Emergency Debugging (Highest Priority)**
```markdown
Entry Point: @quick-troubleshooting-index
├── Hydration errors → @nextjs-hydration-prevention
├── Import errors → @fastapi-dependency-injection  
├── MCP failures → @mcp-browser-tools-setup
└── General issues → @debugging + @sprint-lessons-learned
```

### **🛠️ New Development Work**
```markdown
Entry Point: Context Detection (automatic)
├── Frontend files → @frontend-development-index
├── Backend files → @backend-development-index
├── Database files → @use-mcp-servers (Supabase focus)
└── Test files → @sprint10-testing-patterns
```

### **📋 Feature Implementation**
```markdown
Entry Point: @feature-request
├── Frontend features → @frontend-development-index

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rm2thaddeus/Pixel_Detective](https://github.com/rm2thaddeus/Pixel_Detective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
