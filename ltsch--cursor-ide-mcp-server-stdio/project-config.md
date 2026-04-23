---
trigger: always_on
description: Project Context & Critical Memory - Essential project context and lessons learned for AI development
---


# Project Context & Critical Memory

## 🚨 **CRITICAL: THIS IS A CURSOR IDE MCP SERVER PROJECT**

### **PROJECT CONTEXT**
- **This is a specialized MCP server** for Cursor IDE that manages `.cursor/rules` directories
- **Local-first, secure design** - uses stdio communication, no network connectivity
- **Cross-platform compatibility** - must work on Linux, macOS, and Windows
- **Real-time file watching** - monitors rule files for changes and automatic reloading
- **Portable rule distribution** - copies all MDC files to new projects automatically

### **CURRENT WORKING STATE (December 2024)**
- ✅ **Enhanced File Watching**: Robust file watching with fallback strategies
  - Recursive watching (if supported)
  - Individual file watching (fallback)
  - Directory monitoring for new files
  - Cross-platform compatibility
- ✅ **YAML Frontmatter Validation**: Comprehensive validation of MDC files
  - Checks for required fields (`description`, `globs`)
  - Validates YAML syntax
  - Provides helpful error messages
- ✅ **Portable Rule Distribution**: Self-contained rule management
  - Copies all MDC files from project to new projects
  - Dynamic project name substitution
  - Maintains rule consistency across projects
- ✅ **Enhanced Documentation**: Comprehensive README and rule structure
  - Clear project purpose and architecture
  - Installation and usage instructions
  - Security and design principles

## 🚨 **PAINFUL LESSONS LEARNED**

### **FILE WATCHING PLATFORM COMPATIBILITY (CRITICAL)**

#### **1. Recursive File Watching Not Available on All Platforms**
**Problem**: `fs.watch` with `recursive: true` fails on some Linux systems
**Root Cause**: Platform-specific limitations with recursive file watching
**Solution**: Implemented fallback strategy with individual file watching + directory monitoring
**Impact**: File watching now works reliably across all platforms
**Learning**: ALWAYS implement fallback strategies for platform-specific features

```javascript
// ❌ WRONG - Only try recursive watching
try {
  const watcher = fs.watch(rulesPath, { recursive: true }, callback);
} catch (err) {
  console.error('File watching failed'); // No fallback
}

// ✅ CORRECT - Implement fallback strategies
try {
  const watcher = fs.watch(rulesPath, { recursive: true }, callback);
  console.log('✅ Recursive file watching enabled');
} catch (err) {
  if (err.code === 'ERR_FEATURE_UNAVAILABLE_ON_PLATFORM') {
    // Fallback to individual file watching
    console.warn('⚠️  Recursive file watching not available, trying alternative method...');
    setupIndividualFileWatching();
  }
}
```

#### **2. New File Detection Requires Directory Monitoring**
**Problem**: Individual file watching doesn't detect new files
**Root Cause**: Only watching existing files, not monitoring directory for new files
**Solution**: Added directory watcher to detect new `.mdc` files and add them to watch list
**Impact**: New files are automatically detected and watched
**Learning**: ALWAYS consider what happens when new files are created

```javascript
// ❌ WRONG - Only watch existing files
const ruleFiles = fs.readdirSync(rulesPath).filter(file => file.endsWith('.mdc'));
ruleFiles.forEach(filename => {
  fs.watch(filePath, callback); // Only watches existing files
});

// ✅ CORRECT - Watch directory for new files
const dirWatcher = fs.watch(rulesPath, (eventType, filename) => {
  if (filename && filename.endsWith('.mdc')) {
    if (eventType === 'rename' && fs.existsSync(filePath)) {
      console.log(`🆕 New rule file detected: ${filename}`);
      addFileWatcher(filename); // Add to watch list
    }
  }
});
```

### **YAML FRONTMATTER VALIDATION (CRITICAL)**

#### **3. MCP Files Require Specific YAML Frontmatter**
**Problem**: MDC files need proper YAML frontmatter with required fields
**Root Cause**: Incomplete understanding of MCP file format requirements
**Solution**: Implemented comprehensive validation for `description` and `globs` fields
**Impact**: All MDC files now have proper frontmatter and validation
**Learning**: ALWAYS validate file formats according to specification requirements

```yaml
# ✅ CORRECT - Complete YAML frontmatter
---
title: "Critical Safety Rules"
description: "Essential safety protocols and security rules for AI development"
category: "safety"
priority: "critical"
version: "1.0"
last_updated: "2024-12-19"
tags: ["safety", "security", "destructive-operations", "project-protection"]
globs: ["**/*.py", "**/*.js", "**/*.ts", "**/*.java", "**/*.go", "**/*.rs"]
---

# ❌ WRONG - Missing required fields
---
description: "Some description"
---

# ❌ WRONG - No YAML frontmatter at all
# Content starts immediately
```

#### **4. Project Name Substitution in Rule Files**
**Problem**: Rule files contained hardcoded project names that needed to be dynamic
**Root Cause**: Template files had specific project names instead of placeholders
**Solution**: Implemented dynamic substitution of `${projectName}`, `sse-server`, and `cursor-mdc-server`
**Impact**: Rule files are now truly portable across different projects
**Learning**: ALWAYS use placeholders for project-specific content in templates

```javascript
// ✅ CORRECT - Dynamic project name substitution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ltsch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
