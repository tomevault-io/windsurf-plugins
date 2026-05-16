---
trigger: always_on
description: **Rule Priority:** Advanced Intelligence
---

# Advanced Context Awareness and Dynamic Rule Activation

**Rule Priority:** Advanced Intelligence  
**Activation:** All development activities with dynamic context switching  
**Scope:** Global context management and intelligent rule selection

## Overview

Cursor's advanced context system in v1.2+ enables sophisticated awareness of development environment, file types, git status, time of day, project phase, and user patterns. This rule creates intelligent context switching that dynamically activates relevant rules and optimizes AI assistance based on current conditions.

## Context Detection Systems

### Environment Context Detection

```typescript
// Advanced context detection engine
interface ContextState {
  // File and Project Context
  currentFile: string;
  fileType: string;
  fileSize: number;
  projectType: string;
  
  // Git Context
  gitBranch: string;
  gitStatus: 'clean' | 'modified' | 'staged' | 'conflict';
  uncommittedChanges: number;
  lastCommitTime: Date;
  
  // Development Context
  activeRules: string[];
  recentFiles: string[];
  openTabs: string[];
  cursorPosition: { line: number; column: number };
  
  // Temporal Context
  timeOfDay: 'morning' | 'afternoon' | 'evening' | 'night';
  dayOfWeek: string;
  timezone: string;
  
  // User Context
  workingPattern: 'focused' | 'exploratory' | 'debugging' | 'reviewing';
  errorCount: number;
  productivityScore: number;
  
  // System Context
  cpuUsage: number;
  memoryUsage: number;
  networkStatus: 'online' | 'offline' | 'slow';
}

class ContextEngine {
  private state: ContextState;
  private listeners: ContextListener[] = [];
  
  async detectContext(): Promise<ContextState> {
    return {
      // File context
      currentFile: await this.getCurrentFile(),
      fileType: await this.detectFileType(),
      fileSize: await this.getFileSize(),
      projectType: await this.detectProjectType(),
      
      // Git context
      gitBranch: await this.getGitBranch(),
      gitStatus: await this.getGitStatus(),
      uncommittedChanges: await this.countUncommittedChanges(),
      lastCommitTime: await this.getLastCommitTime(),
      
      // Development context
      activeRules: await this.getActiveRules(),
      recentFiles: await this.getRecentFiles(),
      openTabs: await this.getOpenTabs(),
      cursorPosition: await this.getCursorPosition(),
      
      // Temporal context
      timeOfDay: this.getTimeOfDay(),
      dayOfWeek: new Date().toLocaleDateString('en-US', { weekday: 'long' }),
      timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
      
      // User context
      workingPattern: await this.detectWorkingPattern(),
      errorCount: await this.getErrorCount(),
      productivityScore: await this.calculateProductivityScore(),
      
      // System context
      cpuUsage: await this.getCPUUsage(),
      memoryUsage: await this.getMemoryUsage(),
      networkStatus: await this.getNetworkStatus()
    };
  }
  
  onContextChange(listener: ContextListener): void {
    this.listeners.push(listener);
  }
  
  private async detectWorkingPattern(): Promise<string> {
    // Analyze recent activity patterns
    const recentActions = await this.getRecentActions();
    
    if (recentActions.includes('debugging')) return 'debugging';
    if (recentActions.includes('exploring')) return 'exploratory';
    if (recentActions.includes('reviewing')) return 'reviewing';
    return 'focused';
  }
}
```

### SYMindX-Specific Context

```typescript
// SYMindX project-specific context detection
class SYMindXContextDetector {
  async detectSYMindXContext(): Promise<SYMindXContext> {
    const currentFile = await this.getCurrentFile();
    
    return {
      // Component detection
      component: this.detectComponent(currentFile),
      
      // Agent context
      agentType: this.detectAgentType(currentFile),
      
      // Module context
      moduleType: this.detectModuleType(currentFile),
      
      // Portal context
      portalProvider: this.detectPortalProvider(currentFile),
      
      // Architecture layer
      layer: this.detectArchitectureLayer(currentFile)
    };
  }
  
  private detectComponent(filePath: string): string {
    if (filePath.includes('mind-agents/src/core/')) return 'core-runtime';
    if (filePath.includes('mind-agents/src/portals/')) return 'ai-portal';
    if (filePath.includes('mind-agents/src/memory/')) return 'memory-system';
    if (filePath.includes('mind-agents/src/emotion/')) return 'emotion-system';
    if (filePath.includes('mind-agents/src/cognition/')) return 'cognition-module';
    if (filePath.includes('mind-agents/src/extensions/')) return 'platform-extension';
    if (filePath.includes('mind-agents/src/characters/')) return 'character-system';
    if (filePath.includes('website/')) return 'web-interface';
    if (filePath.includes('docs-site/')) return 'documentation';
    return 'general';
  }
  
  private detectAgentType(filePath: string): string | null {
    // Extract agent type from file path or content
    if (filePath.includes('characters/')) {
      const filename = filePath.split('/').pop();
      return filename?.replace('.json', '') || null;
    }
    return null;
  }
  
  private detectModuleType(filePath: string): string | null {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
