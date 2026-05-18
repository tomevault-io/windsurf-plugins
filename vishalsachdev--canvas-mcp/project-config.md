---
trigger: always_on
description: This guide helps AI agents (Claude, Cursor, Zed, Windsurf, and other MCP clients) effectively use the Canvas MCP server.
---

# Canvas MCP - AI Agent Guide

This guide helps AI agents (Claude, Cursor, Zed, Windsurf, and other MCP clients) effectively use the Canvas MCP server.

## Quick Start

Canvas MCP is a Model Context Protocol server that bridges AI assistants with Canvas Learning Management System. It provides tools for students to track their academic work and for educators to manage courses, grade assignments, and communicate with students.

**Key capability:** The server supports both traditional MCP tool calls AND a code execution API for bulk operations with 99.7% token savings.

## Authentication

All tools require a valid Canvas API token. Two connection modes:

### Remote (Hosted Server — No Installation)
Connect to the hosted server with your credentials as HTTP headers. Best for students and quick evaluation. Educators with FERPA obligations should use local mode.
```json
{
  "mcpServers": {
    "canvas": {
      "url": "https://mcp.illinihunt.org/mcp",
      "headers": {
        "X-Canvas-Token": "your_canvas_api_token",
        "X-Canvas-URL": "https://your-school.instructure.com/api/v1"
      }
    }
  }
}
```

### Local (Self-Hosted)
Configure credentials in the MCP server's `.env` file:
```
CANVAS_API_TOKEN=your_token_here
CANVAS_API_URL=https://your-institution.instructure.com/api/v1
```

Students and educators use the same server but have access to different tools based on Canvas API permissions.

### Tool Profile (Optional)
Reduce tool overhead by setting a role-based profile. Only tools relevant to the selected role are registered:

```
# In .env:
CANVAS_ROLE=student    # ~32 tools (student + shared)
CANVAS_ROLE=educator   # ~87 tools (educator + shared)
CANVAS_ROLE=all        # All 88 tools (default)
```

Or via CLI flag: `canvas-mcp-server --role student` (CLI flag takes precedence over env var).

## Tool Categories

### Student Tools
Personal academic tracking using Canvas "self" endpoints. Students only see their own data.

| Tool | Purpose |
|------|---------|
| `get_my_upcoming_assignments` | Assignments due in next N days |
| `get_my_todo_items` | Canvas TODO list |
| `get_my_submission_status` | What's submitted vs missing |
| `get_my_course_grades` | Current grades across courses |
| `get_my_peer_reviews_todo` | Pending peer reviews to complete |

### Educator Tools
Course management, grading, and analytics. Requires instructor/TA role.

| Tool | Purpose |
|------|---------|
| `list_assignments` | All assignments in a course |
| `get_assignment_details` | Full assignment info including description |
| `list_submissions` | Student submissions for grading |
| `get_assignment_analytics` | Performance statistics |
| `create_assignment` | Create new assignment with due date, submission types, peer reviews |
| `update_assignment` | Update existing assignment (name, due date, points, published, etc.) |
| `get_student_analytics` | Individual student performance |
| `list_rubrics` | List rubrics in a course |
| `get_rubric` | View rubric details (by rubric_id or assignment_id) |
| `get_rubric_assessment` | View rubric assessment for a student submission |
| `create_rubric` | Create rubric with criteria, ratings, and optional assignment association |
| `associate_rubric` | Associate existing rubric with an assignment |
| `grade_with_rubric` | Grade single submission with rubric |
| `bulk_grade_submissions` | Grade multiple submissions efficiently |
| `send_conversation` | Message students |
| `send_peer_review_reminders` | Automated reminder workflow |
| `create_announcement` | Post course announcements |

### Shared Tools (Students & Educators)
Content access tools available to all authenticated users.

| Tool | Purpose |
|------|---------|
| `list_courses` | Enrolled courses |
| `get_course_details` | Course info and syllabus |
| `list_pages` | Course pages |
| `get_page_content` | Read page content |
| `update_page_settings` | Publish/unpublish, set front page, editing roles |
| `bulk_update_pages` | Update multiple pages at once |
| `list_modules` | List course modules |
| `create_module` | Create a new module |
| `update_module` | Update module settings |
| `delete_module` | Delete a module |
| `add_module_item` | Add content to a module |
| `update_module_item` | Update module item settings |
| `delete_module_item` | Remove item from module |
| `list_announcements` | Course announcements |
| `list_discussion_topics` | Discussion forums |
| `list_discussion_entries` | Posts in a discussion |
| `post_discussion_entry` | Add a discussion post |
| `reply_to_discussion_entry` | Reply to a post |

### Learning Designer Tools
Course design, quality assurance, and accessibility compliance.

| Tool | Purpose |
|------|---------|
| `get_course_structure` | Full module→items tree as JSON (one call) |
| `scan_course_content_accessibility` | Scan for WCAG violations |
| `fetch_ufixit_report` | Retrieve UFIXIT accessibility report |
| `parse_ufixit_violations` | Extract structured violations from report |
| `format_accessibility_summary` | Format violations into readable report |

### Developer Tools
Advanced tools for bulk operations and custom logic.

| Tool | Purpose |
|------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vishalsachdev/canvas-mcp](https://github.com/vishalsachdev/canvas-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
