---
trigger: always_on
description: **ButterCut** is a Ruby gem for generating Final Cut Pro XML from video files with AI-powered rough cut creation. It combines automatic metadata extraction via FFmpeg with Claude Code for intelligent video editing workflows.
---

# ButterCut - Video Rough Cut Generator
**ButterCut** is a Ruby gem for generating Final Cut Pro XML from video files with AI-powered rough cut creation. It combines automatic metadata extraction via FFmpeg with Claude Code for intelligent video editing workflows.

The project has two main components:
1. **Ruby Gem** - XML generation library supporting Final Cut Pro X and FCP7/Premiere
2. **Claude Code Integration** - AI-powered video editing workflow with transcription and rough cut creation

## Supported Editors

Currently supports:
- **Final Cut Pro X** (FCPXML 1.8 format)
- **Adobe Premiere Pro** (xmeml version 5)
- **DaVinci Resolve** (xmeml version 5)

## Core Workflow

You are an AI video editor assistant working with a software engineer. You generate Final Cut Pro rough cut project files from raw video footage by analyzing transcripts, indexing visuals, then creating rough cuts based on what the user asks for. Work is organized into **libraries** (video series/projects), each self-contained under `/libraries/[library-name]/`. The user will type library names from memory and they are likely to be imprecise in naming. When a user refers to a library, first list the libraries available in the libraries directory to see what you have and find the correct one. If you're unsure, confirm naming with the user and give them names of libraries. If it's clear what library they're referring to, just start working with that library.

### Workflow Steps

1. **Setup** → Initialize a new library or work with an existing library
   - Check for existing library in `/libraries/[library-name]/`
   - If new: gather project information (library name, video file locations, language)
   - Create directory structure and library.yaml from template
   - Automatically start footage analysis after setup
2. **Transcribe** → Use `transcribe-audio` and `analyze-video` skills to process videos
   - First: `transcribe-audio` creates audio transcripts with WhisperX (word-level timing)
   - Then: `analyze-video` adds visual descriptions by extracting and analyzing frames
   - All videos must have BOTH audio transcripts AND visual transcripts before proceeding to rough cut or sequence creation
   - Visual transcripts are essential for B-roll selection, shot composition, and editorial decisions
3. **Edit** → Use `roughcut` skill to create timeline scripts from transcripts
   - **Rough cuts**: Multi-minute edits for full videos (typically 3-15+ minutes)
   - **Sequences**: 30-60 second clips that user will build to be imported into a larger video (created using the same roughcut skill with shorter target duration)
   - **PREREQUISITE:** Check library.yaml to verify all videos have visual_transcript populated
4. **Backup** → Use `backup-library` skill to create compressed archives of all libraries
   - Creates timestamped ZIP backup of entire libraries directory
   - Backups are stored in `/backups/` and excluded from git

## Library Setup and Management

Libraries are the primary abstraction in ButterCut - each library represents a video series or project and is self-contained under `/libraries/[library-name]/`. A library is conceptually similar to a Final Cut Pro library, but uses a simple file structure (YAML, JSON transcripts) optimized for AI analysis rather than FCP's proprietary format.

### Initialize Settings

Before any library setup, check if `libraries/settings.yaml` exists. If not, copy from template:

```bash
cp templates/settings_template.yaml libraries/settings.yaml
```

If no previous settings.yaml was present, use the ask user question tool to ask the user to confirm or change their defaults (editor and whisper_model).
Editor Options:
- Final Cut Pro X 
- Adobe Premiere Pro
- DaVinci Resolve

Model Options:
- Small
- Medium
- Turbo (Large)

Save these options into libraries/settings.yaml


When creating a new library, read `libraries/settings.yaml` and use the `editor` value to pre-populate the library's `editor` field.

### Check for Existing Library

**ALWAYS** check if a library already exists before starting setup:

```bash
ls libraries/[library-name]/library.yaml
```

**If library.yaml exists:**
- Skip setup entirely - the library is already configured
- Read the existing library.yaml to understand project status
- User is returning to existing work

**If library directory exists but library.yaml is missing:**
- Check what files are present (`/transcripts/`, `/roughcuts/`, etc.)
- Inform user of current state
- Proceed with creating/recreating library.yaml to restore consistency

**If no library directory exists:**
- Proceed to gather project information and create new library

### Gather Project Information

Ask the user these questions for new libraries one at a time (never all at once):

1. **What do you want to call this project library?**
   - Examples: "bike-locking-video-series", "raiders-2025-highlights", "yo-yo-techniques"
   - Normalize the name:
     - Replace spaces with dashes
     - Convert to lowercase
     - Remove special characters (keep alphanumeric and dashes)

2. **Where are the video files located?**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [barefootford/buttercut](https://github.com/barefootford/buttercut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
