---
trigger: always_on
description: **ALWAYS follow these instructions first and fallback to additional search and
---

# AI-Infra Repository Copilot Instructions

**ALWAYS follow these instructions first and fallback to additional search and
context gathering only if the information here is incomplete or found to be in
error.**

## Repository Overview

AI-Infra is a documentation-focused repository providing curated landscape and
structured learning paths for engineers building and operating modern AI
infrastructure, especially in the Kubernetes and cloud-native ecosystem. This
is a pure documentation repository with no source code - only markdown files
organized into learning topics.

## Working Effectively

### Repository Bootstrap & Validation

- **Bootstrap dependencies:**

  ```bash
  npm install -g markdownlint-cli markdown-link-check
  ```

  - Takes 1-2 minutes. NEVER CANCEL. Set timeout to 5+ minutes.

- **Validate all markdown files:**

  ```bash
  markdownlint **/*.md
  ```

  - Takes 10-30 seconds. Identifies formatting issues, line length
    violations, and style problems.

- **Check all external links:**

  ```bash
  markdown-link-check README.md
  ```

  - Takes 30-60 seconds per file. Some external links may be temporarily
    unreachable (Status: 0 errors are common for conference websites).
  - Check key documentation files: `README.md`, `docs/inference/README.md`,
    `docs/kubernetes/learning-plan.md`

### Content Validation Requirements

- **ALWAYS run markdown linting before committing:**

  ```bash
  markdownlint **/*.md
  ```

  - Fix line length violations (80 character limit)
  - Fix trailing spaces and multiple blank lines
  - Address inline HTML warnings where appropriate

- **ALWAYS validate external links for new content:**

  ```bash
  find . -name "*.md" -exec markdown-link-check {} \;
  ```

  - NEVER CANCEL: Full link check takes 3-5 minutes across all files.
    Set timeout to 10+ minutes.
  - Ignore Status: 0 errors for conference websites and GitHub assets -
    these are often temporary

### Repository Structure

```text
/home/runner/work/AI-Infra/AI-Infra/
├── README.md                          # Main learning path and landscape overview
├── diagrams/                          # All images centralized
│   ├── ai-infra-landscape.png        # Visual landscape diagram
│   └── pod-lifecycle.png             # Pod lifecycle diagram
├── docs/
│   ├── kubernetes/
│   │   ├── README.md                 # Kubernetes overview (canonical)
│   │   ├── learning-plan.md          # Kubernetes learning plan (3-phase approach)
│   │   ├── pod-lifecycle.md          # Pod lifecycle documentation
│   │   ├── pod-startup-speed.md      # Pod startup optimization guide
│   │   ├── scheduling-optimization.md # Scheduling strategies and optimization
│   │   ├── isolation.md              # Workload isolation techniques
│   │   ├── dra.md                    # Dynamic Resource Allocation reference
│   │   └── nri.md                    # Node Resource Interface reference
│   ├── inference/
│   │   ├── README.md                 # LLM inference engine comparisons
│   │   ├── aibrix.md                 # AIBrix platform
│   │   ├── pd-disaggregation.md      # Prefill-Decode disaggregation
│   │   ├── caching.md                # Caching strategies
│   │   ├── memory-context-db.md      # Memory/Context DB
│   │   ├── large-scale-experts.md    # MoE models
│   │   ├── model-lifecycle.md        # Model lifecycle
│   │   └── ome.md                    # OME platform
│   └── training/
│       ├── README.md                 # Training on Kubernetes overview
│       ├── kubeflow.md               # Kubeflow training operators
│       └── argocd.md                 # GitOps with ArgoCD
└── .github/
    └── copilot-instructions.md
```

### Common Development Tasks

#### Adding New Learning Materials

1. **Create new markdown files in appropriate directories:**
   - Scheduling/Workloads content → `docs/kubernetes/`
   - Inference optimization content → `docs/inference/`
   - Training and ML pipelines → `docs/training/`

2. **Always include metadata headers and proper structure:**

   ```markdown
   ---
   status: Active
   maintainer: pacoxu
   last_updated: YYYY-MM-DD
   tags: tag1, tag2, tag3
   canonical_path: docs/category/filename.md
   ---
   
   # Title
   
   Brief introduction paragraph.
   
   ## Section Headers
   
   Content with proper formatting.
   ```

3. **Reference external projects consistently:**

   ```markdown
   - <a href="https://github.com/org/project">`ProjectName`</a>: Brief description.
     CNCF Status if applicable.
   ```

4. **Images should be placed in diagrams/ and referenced with relative paths:**

   ```markdown
   ![Image Description](../../diagrams/image-name.png)
   ```

5. **Add learning topics and roadmap sections for educational content:**

   ```markdown
   ## Learning Topics
   - Topic 1
   - Topic 2
   
   ## RoadMap (Ongoing Proposals)
   - Link to KEPs, issues, or proposals
   ```

#### Updating the Main README

- **Always update the main learning path when adding new topics**
- **Maintain the three-tier structure: Scheduling & Workloads, Inference &
  Runtime, AI Gateway & Agentic Workflow**
- **Include CNCF project status (Graduated, Incubating, Sandbox) where
  applicable**
- **Link to detailed documentation in subdirectories using relative paths:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pacoxu/AI-Infra](https://github.com/pacoxu/AI-Infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
