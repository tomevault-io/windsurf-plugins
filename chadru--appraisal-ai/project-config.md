---
trigger: always_on
description: When a session begins:
---

# Claude Code Instructions for This Repo

## On Every Session Start

When a session begins:

### 1. Load context snapshot
Read `.claude/CPM_SNAPSHOT.md` if it exists. This contains project context, architecture decisions, known issues, and lessons learned from previous sessions. **This is your institutional memory — read it before doing anything else.**

### 2. Check virtual environment

```bash
ls ~/appraisal_ai/venv/bin/python
```

- **If it exists:** The user has already set up. Use `~/appraisal_ai/venv/bin/python` for all script commands. Tell the user:
  > Welcome back. Your virtual environment is set up and ready. What are we working on?

- **If it doesn't exist:** The user is either new or hasn't set up yet. Check if `requirements.txt` dependencies are needed and walk them through setup (see Step 3 below).

### 3. CPM snapshot updates (automatic — do not wait for user to ask)

Update `.claude/CPM_SNAPSHOT.md` at these trigger points:

- **After delivering a draft/grid** (end of Phase 5) — record pipeline results, issues found, project status
- **After user provides corrections** (Phase 4.5 Step 4) — record what was corrected and whether it reveals a pattern worth adding to Lessons Learned
- **When user says "run CPM" / "do a CPM pass"** — full refresh of the entire snapshot
- **End of any session with significant work** — capture new rules, workflow changes, lessons

What to include: project status, lessons learned summary, architecture changes, known issues.
What NOT to include: client-specific data (addresses, dollar amounts, names) — the snapshot is in git.

## New User Onboarding

When a user first opens this repo and asks for help (e.g., "help me get started", "how do I use this", "I just cloned this repo"), walk them through setup step by step:

### Step 1: Ask for their project folder

Say something like:

> I'll help you get set up. First, I need you to create a **new empty folder** for your project. Name it with your file number and address, like `2026-001 123 Main St`.
>
> **Important:** The folder needs to be empty so I can set it up correctly. If you already have documents for this project, that's fine — I'll tell you where to put them in a moment.
>
> Once you've created the folder, I need the folder path. The easiest way is to **drag the folder from File Explorer (Windows) or Finder (Mac) directly into this terminal window** — it will paste the full path automatically.
>
> Or you can copy the path manually:
> - **Windows:** Open File Explorer, navigate to the folder, click the **address bar** at the top — it will highlight the full path. Right-click and **Copy**, then paste it here.
> - **Mac:** Right-click the folder in Finder, hold **Option**, and click **"Copy as Pathname"**. Then paste it here.

### Step 2: Create the project folder structure

Once they give you the path, create these subfolders inside it. **Do not use dots, spaces, or numbered prefixes in folder names** — just plain names:

```
<project-folder>/
├── Subject/         — engagement letter, assessor card, deed, LOI, CoStar, etc.
├── Narrative/       — output drafts will go here
├── Comparables/     — comp sheets, CoStar/MLS printouts, sale data
├── Exhibits/        — maps, photos, figures
└── Template/        — put your completed appraisal template (.docx) and grid (.xlsx) here
```

Create all five folders using mkdir. Then tell the user:

> I've set up your project folder with five subfolders. Now you need to add your files:
>
> 1. **Subject/** — Drop in your engagement letter, assessor card, deed, LOI, CoStar reports, and any other subject property documents.
> 2. **Comparables/** — Drop in your comp sheets, CoStar/MLS printouts, and sale data.
> 3. **Exhibits/** — Drop in any maps, photos, or figures.
> 4. **Template/** — Drop in a **completed appraisal** (.docx) that you want to use as your master template. If you have a grid/schedule (.xlsx), drop that in too.
>
> Go ahead and add your files, then come back and tell me when you're ready.

### Step 3: Set up virtual environment and install dependencies

**Always use a virtual environment.** Never install packages directly on the host system.

Create the venv and install packages:

```bash
python3 -m venv ~/appraisal_ai/venv
~/appraisal_ai/venv/bin/pip install -r ~/appraisal_ai/requirements.txt
```

If the user is on **Windows (not WSL)** and using PowerShell:
```powershell
python -m venv ~\appraisal_ai\venv
~\appraisal_ai\venv\Scripts\pip install -r ~\appraisal_ai\requirements.txt
```

**Running scripts:** Always use the venv's Python interpreter directly. Shell state doesn't persist between commands in Claude Code, so `source activate` won't stick. Use the full path:

```bash
~/appraisal_ai/venv/bin/python -c "import sys, os; sys.path.insert(0, os.path.expanduser('~/appraisal_ai/scripts')); ..."
```

Tell the user:

> I've created a virtual environment so all the Python packages stay isolated from your system. I'll use it automatically when running scripts — you don't need to do anything extra.
>
> If you ever want to run scripts manually outside of Claude Code, activate the venv first:
>
> **WSL/Linux/Mac:** `source ~/appraisal_ai/venv/bin/activate`
> **Windows PowerShell:** `~\appraisal_ai\venv\Scripts\Activate.ps1`
>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chadru/appraisal_ai](https://github.com/chadru/appraisal_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
