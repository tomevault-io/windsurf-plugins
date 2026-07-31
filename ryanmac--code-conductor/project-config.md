---
trigger: always_on
description: This directory is reserved for future template functionality. Currently, task and configuration templates are provided via the `examples/` directory at the repository root. Claude Code should use the existing examples to understand patterns for generating new tasks, configurations, and project-specific customizations.
---

# CLAUDE.md: Instructions for Claude Code - Templates & Examples

## Purpose
This directory is reserved for future template functionality. Currently, task and configuration templates are provided via the `examples/` directory at the repository root. Claude Code should use the existing examples to understand patterns for generating new tasks, configurations, and project-specific customizations.

## Current Template Sources

### Task Templates
Located in `examples/[stack-name]/example-tasks.json`:
- React webapp tasks
- Python microservices tasks
- Mobile app tasks
- Desktop app tasks
- Go services tasks
- And more...

### Configuration Templates
Located in `examples/[stack-name]/config.yaml`:
- Stack-specific configurations
- Role recommendations
- Build/test commands

## Guidelines for Template Usage
- **Do**: Study existing examples before generating new content
- **Do**: Maintain the JSON structure when creating tasks
- **Do**: Include all required fields from templates
- **Do**: Adapt templates to specific project needs
- **Don't**: Create templates that bypass quality checks
- **Don't**: Generate tasks without proper file locking
- **Don't**: Omit required fields like success_criteria

## Task Template Structure

When generating new tasks, follow this structure:

```json
{
  "id": "unique_id",
  "title": "Clear, actionable title",
  "description": "Detailed description of what needs to be done",
  "specs": "path/to/specification.md",
  "best_practices": [
    "List of recommended approaches",
    "Technology-specific guidelines"
  ],
  "success_criteria": {
    "tests": "Testing requirements",
    "performance": "Performance metrics",
    "security": "Security requirements",
    "accessibility": "Accessibility standards"
  },
  "required_skills": [],  // Empty for generalist, or ["frontend", "devops"]
  "estimated_effort": "small|medium|large",
  "files_locked": [
    "paths/to/files/that/will/be/modified"
  ],
  "dependencies": ["task_ids_that_must_complete_first"]
}
```

## Autonomous Task Generation Algorithm

```python
def generate_task(description, context):
    # 1. Detect task type from description
    task_type = classify_task(description)  # ui, api, devops, etc.
    
    # 2. Load appropriate template
    template = load_template_for_type(task_type)
    
    # 3. Extract key requirements
    requirements = extract_requirements(description)
    
    # 4. Auto-generate task fields
    task = {
        "id": generate_unique_id(task_type),
        "title": extract_action_title(description),
        "description": expand_description(description, context),
        "specs": f"docs/{task_type}-{task['id']}-spec.md",
        "best_practices": template["best_practices"] + requirements["practices"],
        "success_criteria": merge_criteria(template["criteria"], requirements),
        "required_skills": detect_required_skills(task_type, requirements),
        "estimated_effort": estimate_effort(requirements),
        "files_locked": predict_affected_files(task_type, requirements),
        "dependencies": detect_dependencies(context["existing_tasks"])
    }
    
    # 5. Validate before returning
    validate_task_completeness(task)
    validate_no_conflicts(task, context["active_tasks"])
    
    return task
```

## Examples

### Example 1: Generate Frontend Task
**Input**: "Implement user profile component with avatar upload"

**Automated generation**:
```json
{
  "id": "ui_profile_001",
  "title": "Implement user profile component with avatar upload",
  "description": "Build a responsive user profile component featuring avatar upload functionality, user details display, and edit capabilities",
  "specs": "docs/ui-profile-001-spec.md",
  "best_practices": [
    "Use semantic HTML for accessibility",
    "Implement keyboard navigation",
    "Add proper ARIA labels",
    "Ensure responsive design works on all screen sizes",
    "Validate image uploads (size, format)",
    "Add loading states for upload process"
  ],
  "success_criteria": {
    "tests": "Component tests with 90% coverage including upload scenarios",
    "accessibility": "WCAG AA compliant",
    "responsive": "Works on mobile, tablet, desktop",
    "performance": "Image upload < 3s on 3G connection",
    "validation": "Handles invalid file types gracefully"
  },
  "required_skills": [],
  "estimated_effort": "large",
  "files_locked": [
    "src/components/UserProfile/",
    "src/components/UserProfile/UserProfile.tsx",
    "src/components/UserProfile/AvatarUpload.tsx",
    "src/services/upload.ts",
    "src/types/user.ts"
  ],
  "dependencies": []
}
```

### Example 2: Generate API Integration Task
**Prompt**: "Create a task for integrating with a payment API"

**Expected approach**:
1. Study `examples/nodejs-api/example-tasks.json`
2. Include security best practices
3. Add proper error handling requirements
4. Lock API service files and tests
5. Consider dependencies on auth tasks

### Example 3: Generate DevOps Task
**Prompt**: "Create a CI/CD pipeline setup task"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryanmac/code-conductor](https://github.com/ryanmac/code-conductor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
