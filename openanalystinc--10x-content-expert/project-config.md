---
trigger: always_on
description: This is a comprehensive content creation and file management system powered by Claude. It helps create, edit, and manage content across multiple formats.
---

# 10X Content Expert - Claude Instructions

## Project Overview
This is a comprehensive content creation and file management system powered by Claude. It helps create, edit, and manage content across multiple formats.

## CRITICAL: Output Path Rules

**ALL generated files MUST go to the `output/` folder structure:**

| File Type | Output Location |
|-----------|-----------------|
| PDF files | `output/pdf/` |
| PPTX files | `output/pptx/` |
| DOCX files | `output/docx/` |
| XLSX files | `output/xlsx/` |
| LinkedIn content | `output/content/social/linkedin/` |
| Twitter content | `output/content/social/twitter/` |
| Email content | `output/content/emails/` |
| Blog content | `output/content/blogs/` |
| Presentations content | `output/content/presentations/` |
| Email sequences | `output/content/sequences/` |
| Hooks/CTAs | `output/content/hooks/` |
| Campaigns | `output/content/campaigns/` |
| Analysis results | `output/analysis/` |
| Content plans | `output/plans/` |
| Working/temp files | `output/working/` |
| MEGA downloads | `output/mega-downloads/` |
| Transcripts | `output/transcripts/` |
| Canvas exports | `output/canvas/` |
| Logs | `output/logs/` |

## Output Path Constants

When generating ANY file output, use these paths:

```
PROJECT_ROOT = C:\Users\Anit\Downloads\10x-Content-Expert

OUTPUT_PATHS = {
    "pdf": "output/pdf/",
    "pptx": "output/pptx/",
    "docx": "output/docx/",
    "xlsx": "output/xlsx/",
    "linkedin": "output/content/social/linkedin/",
    "twitter": "output/content/social/twitter/",
    "instagram": "output/content/social/instagram/",
    "emails": "output/content/emails/",
    "blogs": "output/content/blogs/",
    "presentations": "output/content/presentations/",
    "sequences": "output/content/sequences/",
    "hooks": "output/content/hooks/",
    "campaigns": "output/content/campaigns/",
    "analysis": "output/analysis/",
    "plans": "output/plans/",
    "working": "output/working/",
    "mega": "output/mega-downloads/",
    "transcripts": "output/transcripts/",
    "canvas": "output/canvas/",
    "logs": "output/logs/"
}
```

## File Naming Convention

Use this pattern for generated files:
```
[YYYY-MM-DD]_[topic/name]_[type].[extension]
```

Examples:
- `2026-01-28_arjun_maheshwari_linkedin_profile.pdf`
- `2026-01-28_product_launch_email_sequence.md`
- `2026-01-28_brand_analysis.json`

## Folder Structure

```
10x-Content-Expert/
├── input/                    # User files to edit (READ-ONLY)
├── references/               # Learning materials (READ-ONLY)
│   ├── transcripts/
│   ├── examples/
│   ├── brand-voice/
│   └── templates/
├── samples/                  # Visual references (READ-ONLY)
├── output/                   # ALL OUTPUTS GO HERE
│   ├── pdf/                  # Generated/edited PDFs
│   ├── pptx/                 # Generated/edited PowerPoints
│   ├── docx/                 # Generated/edited Word docs
│   ├── xlsx/                 # Generated/edited Excel files
│   ├── content/              # Written content
│   │   ├── social/
│   │   │   ├── linkedin/
│   │   │   ├── twitter/
│   │   │   └── instagram/
│   │   ├── emails/
│   │   ├── blogs/
│   │   ├── presentations/
│   │   ├── sequences/
│   │   ├── hooks/
│   │   └── campaigns/
│   ├── analysis/             # Analysis results
│   ├── plans/                # Content plans
│   ├── working/              # Temp/working files
│   ├── mega-downloads/       # MEGA cloud downloads
│   ├── transcripts/          # Generated transcripts
│   ├── canvas/               # TLDraw exports
│   └── logs/                 # Operation logs
├── scripts/                  # Python utilities
└── .claude/                  # Skills and agents
```

## Safety Rules

### NEVER Modify
- `input/` - Original files
- `references/` - Learning materials
- `samples/` - Design references

### ALWAYS Output To
- `output/` - All generated content goes here

### The Copy Pattern
When editing user files:
1. Copy original from `input/` to `output/working/`
2. Make ALL edits on the copy
3. Save final result to appropriate `output/[type]/` folder
4. Original in `input/` stays untouched

## Python Script Output

When creating Python scripts that generate files:

```python
import os

# Define project root and output paths
PROJECT_ROOT = r"C:\Users\Anit\Downloads\10x-Content-Expert"

OUTPUT_PATHS = {
    "pdf": os.path.join(PROJECT_ROOT, "output", "pdf"),
    "pptx": os.path.join(PROJECT_ROOT, "output", "pptx"),
    "docx": os.path.join(PROJECT_ROOT, "output", "docx"),
    "xlsx": os.path.join(PROJECT_ROOT, "output", "xlsx"),
    "content": os.path.join(PROJECT_ROOT, "output", "content"),
    "analysis": os.path.join(PROJECT_ROOT, "output", "analysis"),
    "plans": os.path.join(PROJECT_ROOT, "output", "plans"),
    "working": os.path.join(PROJECT_ROOT, "output", "working"),
}

# Always use these paths for output
def get_output_path(file_type, filename):
    """Get the correct output path for a file type."""
    base_path = OUTPUT_PATHS.get(file_type, OUTPUT_PATHS["working"])
    os.makedirs(base_path, exist_ok=True)
    return os.path.join(base_path, filename)
```

## MEGA Integration

- MEGA downloads go to: `output/mega-downloads/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenAnalystInc/10x-Content-Expert](https://github.com/OpenAnalystInc/10x-Content-Expert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
