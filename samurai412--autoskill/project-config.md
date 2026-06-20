---
trigger: always_on
description: >-
---


# Adapt Skill

**One system that does everything:**
1. **Auto-detects project type** and loads matching skills
2. Instruments skills (adds learning structure if missing)
3. Scans project (finds opportunities + exemplars)
4. Proposes improvements (skill teaches project)
5. Learns patterns (project teaches skill)
6. Enables continuous learning (ongoing improvement)

---

## Three Modes

### Mode 0: Auto-Detection (On Session Start)

```
Agent starts working on project

→ Scan project for file types (*.cs, *.py, *.tsx, etc.)
→ Match file types to skill registry
→ Auto-load matching skills
→ Enable continuous learning for loaded skills

No user invocation. Happens at session start.
```

### Mode 1: Explicit Adaptation (User Invokes)

```
User: "adapt unity-game-dev.md"

→ Full scan of project
→ Propose all improvements
→ Bulk learn from project
→ One-time comprehensive pass
```

### Mode 2: Continuous Learning (Automatic)

```
Agent working on task, reads PlayerController.cs

→ Notice: "GetComponent cached in Awake" (good pattern)
→ Queue learning
→ On task complete: flush learnings to skill

No user invocation. Happens automatically.
```

**Mode 0 sets up the system. Mode 2 keeps it running. Mode 1 is for deep passes.**

---

## Mode 0: Auto-Detection Process

### When to Run

Run auto-detection at the START of a session, before any user task:

```
SESSION START
    │
    ▼
┌─────────────────────────────────────────┐
│           AUTO-DETECTION                │
│                                         │
│  1. Scan project root for file types    │
│  2. Match against skill registry        │
│  3. Load matching skills                │
│  4. Instrument if needed                │
│  5. Report what was loaded              │
│                                         │
└─────────────────────────────────────────┘
    │
    ▼
READY FOR USER TASK (skills loaded, continuous learning active)
```

### Step 1: Scan for File Types

```
Use Glob to find files in project:

file_types = {}
FOR pattern in ["*.cs", "*.py", "*.tsx", "*.ts", "*.jsx", "*.js", "*.go", "*.rs", "*.rb"]:
  count = Glob(pattern, project_root).count()
  IF count > 0:
    file_types[pattern] = count

OUTPUT: {"*.cs": 47, "*.json": 12, ...}
```

### Step 2: Skills Registry

Map file types to skills. The registry can be:

**Embedded (default):**
```yaml
skill_registry:
  "*.cs":
    - skill: unity-game-dev
      confidence: high
      requires: ["*.unity", "Assets/"]  # Optional extra checks
    - skill: dotnet-backend
      confidence: medium

  "*.py":
    - skill: python-backend
      confidence: high
    - skill: django-web
      requires: ["manage.py", "settings.py"]

  "*.tsx":
    - skill: react-frontend
      confidence: high
    - skill: nextjs-app
      requires: ["next.config.*"]

  "*.go":
    - skill: golang-backend
      confidence: high

  "*.rs":
    - skill: rust-systems
      confidence: high
```

**Or external (skill-registry.yaml in skills folder).**

### Step 3: Match and Score

```
FOR file_type, skills in skill_registry:
  IF file_type IN detected_file_types:
    FOR skill in skills:
      score = 0

      # Base score from file count
      score += min(detected_file_types[file_type] / 10, 5)

      # Confidence boost
      IF skill.confidence == "high": score += 3
      IF skill.confidence == "medium": score += 1

      # Requirements check
      IF skill.requires:
        matches = check_requirements(skill.requires)
        IF matches: score += 5
        ELSE: score -= 10  # Don't load if requirements fail

      add_candidate(skill.name, score)

# Take top skills (avoid loading too many)
selected = candidates.sort_by_score().take(3)
```

### Step 4: Load Skills

```
FOR skill_name in selected:
  skill_path = find_skill(skill_name)  # Check local .skills/, then ~/.agents/skills/

  IF skill_path:
    skill = READ(skill_path)

    # Instrument if needed (Step 0 from existing process)
    IF not_instrumented(skill):
      instrument(skill)

    # Add to loaded skills
    loaded_skills.append(skill)

    LOG: "Loaded {skill_name} for {file_types}"
```

### Step 5: Report

```
At session start, briefly report:

"Auto-loaded skills for this project:
  - unity-game-dev (47 .cs files, Assets/ folder detected)
  - netcode-multiplayer (NetworkBehaviour found)

Continuous learning is active."
```

---

## Default Skill Registry

Built-in mappings (can be overridden):

```yaml
# Game Development
"*.cs" + "*.unity":     unity-game-dev
"*.cs" + "*.uproject":  unreal-cpp  # Unreal uses .cs for build scripts
"*.gd":                 godot-gdscript

# Web Frontend
"*.tsx" + "*.jsx":      react-frontend
"*.vue":                vue-frontend
"*.svelte":             svelte-frontend
"next.config.*":        nextjs-app
"nuxt.config.*":        nuxt-app

# Web Backend
"*.py" + "manage.py":   django-web
"*.py" + "app.py":      flask-api
"*.py" + "main.py":     fastapi-backend
"*.rb" + "Gemfile":     rails-backend
"*.go" + "go.mod":      golang-backend
"*.rs" + "Cargo.toml":  rust-backend

# Mobile
"*.swift":              ios-swift
"*.kt" + "*.gradle":    android-kotlin

# Infrastructure
"*.tf":                 terraform-infra
"*.yaml" + "k8s/":      kubernetes-ops
"docker-compose.*":     docker-compose

# Data/ML
"*.ipynb":              jupyter-datascience

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Samurai412/autoskill](https://github.com/Samurai412/autoskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
