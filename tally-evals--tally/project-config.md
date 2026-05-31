---
trigger: always_on
description: This document provides comprehensive guidelines for using the available tools in Cursor editor effectively and efficiently.
---


# Cursor Tools Usage Guidelines

This document provides comprehensive guidelines for using the available tools in Cursor editor effectively and efficiently.

## Table of Contents

1. [File Management Tools](#file-management-tools)
2. [Code Search and Analysis Tools](#code-search-and-analysis-tools)
3. [Terminal and Execution Tools](#terminal-and-execution-tools)
4. [Browser Automation Tools](#browser-automation-tools)
5. [Database Tools](#database-tools)
6. [Task Management Tools](#task-management-tools)
7. [External Integration Tools](#external-integration-tools)
8. [General Best Practices](#general-best-practices)

## File Management Tools

### Reading Files

#### When to Use `read_file`
- Reading configuration files (package.json, tsconfig.json, etc.)
- Examining source code to understand implementation
- Reviewing documentation files
- Checking test files

#### Best Practices
- Use `offset` and `limit` parameters for large files to avoid unnecessary token usage
- Always check file size before reading very large files
- Use absolute paths when possible to avoid ambiguity

```typescript
// Good: Reading specific lines of a large file
read_file({
  target_file: "/home/hammad/projects/spec-lab/apps/web/src/components/FeatureForm.tsx",
  offset: 1,
  limit: 50
})

// Good: Reading entire small file
read_file({
  target_file: "/home/hammad/projects/spec-lab/package.json"
})
```

### Listing Directories

#### When to Use `list_dir`
- Exploring project structure
- Finding available files in a directory
- Understanding folder organization

#### Best Practices
- Use `ignore_globs` to exclude unnecessary files (node_modules, .git, etc.)
- Start with root directory to understand overall structure
- Use absolute paths for consistency

```typescript
// Good: Listing directory with ignored files
list_dir({
  target_directory: "/home/hammad/projects/spec-lab",
  ignore_globs: ["node_modules", ".git", "dist"]
})
```

### Editing Files

#### When to Use `edit_file`
- Making targeted code changes
- Adding new functionality to existing files
- Updating configuration files
- Creating new files

#### Best Practices
- Provide clear, specific instructions
- Use `// ... existing code ...` to represent unchanged code
- Make minimal, focused changes per edit
- Always include sufficient context around changes

```typescript
// Good: Clear instruction with sufficient context
edit_file({
  target_file: "/home/hammad/projects/spec-lab/apps/web/src/components/FeatureForm.tsx",
  instructions: "Add validation for the feature name field",
  code_edit: `
const validateFeatureName = (name: string): boolean => {
  // ... existing code ...
  return name.length > 0 && name.length <= 100;
};

// ... existing code ...

const handleSubmit = () => {
  if (!validateFeatureName(featureName)) {
    setError("Feature name must be between 1 and 100 characters");
    return;
  }
  // ... existing code ...
};
`
})
```

### Editing Jupyter Notebooks

#### When to Use `edit_notebook`
- Modifying Jupyter notebook cells
- Adding new cells to notebooks
- Updating notebook content

#### Best Practices
- Always specify the correct cell index (0-based)
- Set `is_new_cell` correctly based on whether you're creating or editing
- Include sufficient context in `old_string` to uniquely identify the cell
- Use the correct `cell_language` parameter

```typescript
// Good: Editing an existing cell
edit_notebook({
  target_notebook: "/home/hammad/projects/spec-lab/notebooks/analysis.ipynb",
  cell_idx: 0,
  is_new_cell: false,
  cell_language: "python",
  old_string: "import pandas as pd\n# Load data\ndf = pd.read_csv('data.csv')",
  new_string: "import pandas as pd\n# Load data\ndf = pd.read_csv('data.csv')\n# Clean data\ndf = df.dropna()"
})
```

### Deleting Files

#### When to Use `delete_file`
- Removing temporary files
- Cleaning up generated files
- Removing deprecated code files

#### Best Practices
- Always provide a clear explanation
- Verify the file exists before attempting deletion
- Check if the file is referenced elsewhere before deletion

```typescript
// Good: Clear explanation
delete_file({
  target_file: "/home/hammad/projects/spec-lab/temp-backup.json",
  explanation: "Removing temporary backup file created during migration"
})
```

### Checking Linting Errors

#### When to Use `read_lints`
- Identifying linting issues in files
- Checking code quality before committing
- Finding potential problems in specific files or directories

#### Best Practices
- Focus on specific files or directories rather than the entire workspace
- Use after making significant changes to verify code quality
- Address linting errors before finalizing changes

```typescript
// Good: Checking specific files
read_lints({
  paths: ["/home/hammad/projects/spec-lab/apps/web/src/components/FeatureForm.tsx"]
})
```

## Code Search and Analysis Tools

### Using `grep`

#### When to Use `grep`
- Finding exact text matches in code
- Searching for specific function names, variables, or patterns
- Finding all occurrences of a specific import or usage
- Counting occurrences of patterns

#### Best Practices
- Use specific patterns to avoid too many results
- Use `head_limit` for large codebases

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tally-evals/tally](https://github.com/tally-evals/tally) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
