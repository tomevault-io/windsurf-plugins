---
trigger: always_on
description: Standardized patterns for gathering and processing context from multiple sources in Task Master commands, particularly for AI-powered features.
---

# Context Gathering Patterns and Utilities

This document outlines the standardized patterns for gathering and processing context from multiple sources in Task Master commands, particularly for AI-powered features.

## Core Context Gathering Utility

The `ContextGatherer` class (`scripts/modules/utils/contextGatherer.js`) provides a centralized, reusable utility for extracting context from multiple sources:

### **Key Features**
- **Multi-source Context**: Tasks, files, custom text, project file tree
- **Token Counting**: Detailed breakdown using `gpt-tokens` library
- **Format Support**: Different output formats (research, chat, system-prompt)
- **Error Handling**: Graceful handling of missing files, invalid task IDs
- **Performance**: File size limits, depth limits for tree generation

### **Usage Pattern**
```javascript
import { ContextGatherer } from '../utils/contextGatherer.js';

// Initialize with project paths
const gatherer = new ContextGatherer(projectRoot, tasksPath);

// Gather context with detailed token breakdown
const result = await gatherer.gather({
    tasks: ['15', '16.2'],           // Task and subtask IDs
    files: ['src/api.js', 'README.md'], // File paths
    customContext: 'Additional context text',
    includeProjectTree: true,        // Include file tree
    format: 'research',              // Output format
    includeTokenCounts: true         // Get detailed token breakdown
});

// Access results
const contextString = result.context;
const tokenBreakdown = result.tokenBreakdown;
```

### **Token Breakdown Structure**
```javascript
{
    customContext: { tokens: 150, characters: 800 },
    tasks: [
        { id: '15', type: 'task', title: 'Task Title', tokens: 245, characters: 1200 },
        { id: '16.2', type: 'subtask', title: 'Subtask Title', tokens: 180, characters: 900 }
    ],
    files: [
        { path: 'src/api.js', tokens: 890, characters: 4500, size: '4.5 KB' }
    ],
    projectTree: { tokens: 320, characters: 1600 },
    total: { tokens: 1785, characters: 8000 }
}
```

## Fuzzy Search Integration

The `FuzzyTaskSearch` class (`scripts/modules/utils/fuzzyTaskSearch.js`) provides intelligent task discovery:

### **Key Features**
- **Semantic Matching**: Uses Fuse.js for similarity scoring
- **Purpose Categories**: Pattern-based task categorization
- **Relevance Scoring**: High/medium/low relevance thresholds
- **Context-Aware**: Different search configurations for different use cases

### **Usage Pattern**
```javascript
import { FuzzyTaskSearch } from '../utils/fuzzyTaskSearch.js';

// Initialize with tasks data and context
const fuzzySearch = new FuzzyTaskSearch(tasksData.tasks, 'research');

// Find relevant tasks
const searchResults = fuzzySearch.findRelevantTasks(query, {
    maxResults: 8,
    includeRecent: true,
    includeCategoryMatches: true
});

// Get task IDs for context gathering
const taskIds = fuzzySearch.getTaskIds(searchResults);
```

## Implementation Patterns for Commands

### **1. Context-Aware Command Structure**
```javascript
// In command action handler
async function commandAction(prompt, options) {
    // 1. Parameter validation and parsing
    const taskIds = options.id ? parseTaskIds(options.id) : [];
    const filePaths = options.files ? parseFilePaths(options.files) : [];
    
    // 2. Initialize context gatherer
    const projectRoot = findProjectRoot() || '.';
    const tasksPath = path.join(projectRoot, 'tasks', 'tasks.json');
    const gatherer = new ContextGatherer(projectRoot, tasksPath);
    
    // 3. Auto-discover relevant tasks if none specified
    if (taskIds.length === 0) {
        const fuzzySearch = new FuzzyTaskSearch(tasksData.tasks, 'research');
        const discoveredIds = fuzzySearch.getTaskIds(
            fuzzySearch.findRelevantTasks(prompt)
        );
        taskIds.push(...discoveredIds);
    }
    
    // 4. Gather context with token breakdown
    const contextResult = await gatherer.gather({
        tasks: taskIds,
        files: filePaths,
        customContext: options.context,
        includeProjectTree: options.projectTree,
        format: 'research',
        includeTokenCounts: true
    });
    
    // 5. Display token breakdown (for CLI)
    if (outputFormat === 'text') {
        displayDetailedTokenBreakdown(contextResult.tokenBreakdown);
    }
    
    // 6. Use context in AI call
    const aiResult = await generateTextService(role, session, systemPrompt, userPrompt);
    
    // 7. Display results with enhanced formatting
    displayResults(aiResult, contextResult.tokenBreakdown);
}
```

### **2. Token Display Pattern**
```javascript
function displayDetailedTokenBreakdown(tokenBreakdown, systemTokens, userTokens) {
    const sections = [];
    
    // Build context breakdown
    if (tokenBreakdown.tasks?.length > 0) {
        const taskDetails = tokenBreakdown.tasks.map(task => 
            `${task.type === 'subtask' ? '  ' : ''}${task.id}: ${task.tokens.toLocaleString()}`
        ).join('\n');
        sections.push(`Tasks (${tokenBreakdown.tasks.reduce((sum, t) => sum + t.tokens, 0).toLocaleString()}):\n${taskDetails}`);
    }
    
    if (tokenBreakdown.files?.length > 0) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eyaltoledano/claude-task-master](https://github.com/eyaltoledano/claude-task-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
