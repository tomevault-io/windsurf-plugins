---
trigger: always_on
description: This project involves documenting KiwiQ workflows from the active workflows folder.
---

# KiwiQ Workflow Documentation - Claude Instructions

## Project Overview
This project involves documenting KiwiQ workflows from the active workflows folder.

## Project Structure

### Complete Repository Structure
```
document_manager/
├── standalone_client/
│   ├── kiwi_client/
│   │   ├── workflows/
│   │   │   └── active/
│   │   │       ├── content_diagnostics/
│   │   │       │   ├── wf_*.py (8 workflow files)
│   │   │       │   └── llm_inputs/
│   │   │       │       └── *.py (8 prompt/schema files)
│   │   │       ├── content_studio/
│   │   │       │   ├── wf_*.py (10 workflow files)
│   │   │       │   └── llm_inputs/
│   │   │       │       └── *.py (10 prompt/schema files)
│   │   │       ├── playbook/
│   │   │       │   ├── wf_*.py (2 workflow files)
│   │   │       │   └── llm_inputs/
│   │   │       │       └── *.py (2 prompt/schema files)
│   │   │       ├── document_models/
│   │   │       │   └── customer_docs.py
│   │   │       ├── onboarding/
│   │   │       │   └── llm_inputs/
│   │   │       └── testing/
│   │   └── scripts/
│   └── workflow_service_docs/
├── teammate-builder-v1/ (frontend application)
│   ├── app/
│   ├── components/
│   ├── hooks/
│   ├── lib/
│   └── module/
└── product_workflow_documentation/ (our documentation)
    ├── workflow_index.md (main index of all workflows)
    ├── content_diagnostics/ (Content analysis workflows)
    │   ├── prod_*.md (workflow documentation files)
    │   └── llm_inputs/ (copied)
    │       └── *.py (8 files)
    ├── content_studio/ (Content creation workflows)
    │   ├── prod_*.md (workflow documentation files, e.g., prod_blog_user_input_to_brief.md)
    │   └── llm_inputs/ (copied)
    │       └── *.py (10 files)
    ├── playbook/ (Strategic playbook workflows)
    │   ├── prod_*.md (workflow documentation files)
    │   └── llm_inputs/ (copied)
    │       └── *.py (2 files)
    └── frontend_documentation/
        ├── components/
        ├── pages/
        ├── hooks/
        └── services/
```

### Workflow Location
- **Active Workflows**: `/standalone_client/kiwi_client/workflows/active/`
- **Documentation Location**: `/product_workflow_documentation/`

⚠️ **IMPORTANT**: The actual workflow files in `/standalone_client/kiwi_client/workflows/active/` are production files and should **NEVER** be modified under any circumstances. These are source files that the system depends on. All documentation work should only be done in the `/product_workflow_documentation/` folder.

### Workflow Organization
The workflows are organized into three main sections:

#### 1. Content Diagnostics (8 workflows)
Comprehensive analysis and diagnostic workflows that evaluate your content's effectiveness, AI visibility, and competitive positioning. These workflows analyze your existing content across various platforms (blogs, LinkedIn, company websites), assess your executives' digital presence, perform deep research on specific topics, and provide detailed reports on content performance gaps and opportunities for improvement.

**Key Workflows:**
- Blog and LinkedIn content analysis
- Company and executive AI visibility assessment
- Competitor content analysis and benchmarking
- Deep research and content strategy development
- LinkedIn profile scraping and analysis
- Orchestrator workflow for comprehensive diagnostics

#### 2. Content Studio (10 workflows)
End-to-end content creation and optimization workflows that transform user inputs into polished, platform-specific content. These workflows handle the entire content creation pipeline from ideation to publication-ready drafts, including content briefs, calendar planning, and optimization for both blog articles and LinkedIn posts.

**Key Workflows:**
- Blog content creation pipeline (user input → brief → final article)
- LinkedIn content creation and optimization
- Content calendar generation and topic selection
- Content optimization and alternate text suggestions
- Calendar-based topic to brief conversion

#### 3. Playbook (2 workflows)
Strategic content playbook generation workflows that create customized, actionable content strategies. These workflows leverage your content diagnostics reports and predefined strategic plays to build comprehensive playbooks tailored to your brand's voice, goals, and audience, providing clear guidelines for consistent content creation across different platforms.

**Key Workflows:**
- Blog content playbook generation using diagnostics insights
- LinkedIn content playbook creation with platform-specific strategies

### File Naming Conventions
- **Source Workflow Files**: Start with `wf_` prefix (e.g., `wf_blog_user_input_to_brief.py`) in `/standalone_client/kiwi_client/workflows/active/`
- **Documentation Files**: Start with `prod_` prefix (e.g., `prod_blog_user_input_to_brief.md`) in `/product_workflow_documentation/`
- **LLM Input Files**: Start with `prod_llm_inputs_` prefix (e.g., `prod_llm_inputs_blog_user_input_to_brief.py`) in `/product_workflow_documentation/*/llm_inputs/`
- **Schema Files**: Imported from LLM inputs and define the structure of prompts and responses
- **Document Models**: Define document structures in `document_models/customer_docs.py`

## Documentation Template (Product Understanding Focus)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rcortx/kiwiq](https://github.com/rcortx/kiwiq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
