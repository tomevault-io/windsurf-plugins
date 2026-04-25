---
trigger: always_on
description: <!-- CHECKPOINT: SF-CL-7F1B -->
---

# CLAUDE.md - Super-Flashcards AI Instructions
<!-- CHECKPOINT: SF-CL-7F1B -->

---

## ⚠️ Handoff Bridge — MANDATORY

ALL responses to Claude.ai/Corey MUST use the handoff bridge.
Create file → Run handoff_send.py → Provide URL.
NO EXCEPTIONS. See project-methodology/CLAUDE.md for details.

---

## Handoff Lifecycle Protocol

### Receiving Handoffs
1. Note the ID (HO-XXXX) from the handoff header
2. Archive handoff: `handoffs/archive/HO-XXXX_request.md`
3. Delete from inbox (garbage collect)

### Completion Response Format

Every completion MUST include this exact format:

| Field | Value |
|-------|-------|
| ID | HO-XXXX (from original request) |
| Project | [Icon] [Name] |
| Task | [Brief description] |
| Status | COMPLETE / PARTIAL / BLOCKED |
| Commit | [hash] |
| Handoff | [Full GCS URL] |

### Summary Section
After the table, include:
- Brief description of what was done
- Files changed (list)
- Inbox cleanup confirmation

### Git Commit Format
Include ID in commit message:
```
feat: [description] (HO-XXXX)
fix: [description] (HO-XXXX)
```

### Garbage Collection Checklist
After processing handoff:
- [ ] Deleted from inbox: `handoffs/inbox/*.md`
- [ ] Archived to: `handoffs/archive/`
- [ ] Reminded Corey about Downloads folder cleanup

---

> ⚠️ **READ THIS ENTIRE FILE** before writing any code or running any commands.
> **DO NOT** invent or guess infrastructure values. Use EXACT values below.

---

## Project Identity

| Field | Value |
|-------|-------|
| Project Name | Super-Flashcards |
| Description | French vocabulary learning with spaced repetition and pronunciation |
| Repository | https://github.com/coreyprator/Super-Flashcards |
| Local Path | G:\My Drive\Code\Python\Super-Flashcards |
| Methodology | [coreyprator/project-methodology](https://github.com/coreyprator/project-methodology) v3.17 |

---

## GCP Infrastructure (EXACT VALUES - DO NOT GUESS)

| Resource | Value |
|----------|-------|
| GCP Project ID | `super-flashcards-475210` |
| Region | `us-central1` |
| Cloud Run Service | `super-flashcards` |
| Cloud Run URL | `https://super-flashcards-wmrla7fhwa-uc.a.run.app` |
| Cloud SQL Instance | `flashcards-db` |
| Cloud SQL IP | `35.224.242.223` |
| Database Name | `LanguageLearning` |

### Verification Commands
```powershell
# Always verify correct project before ANY gcloud command
gcloud config get-value project
# Must output: super-flashcards-475210

# If wrong:
gcloud config set project super-flashcards-475210
```

---

## Deployment

### Deploy Command
```powershell
cd "G:\My Drive\Code\Python\Super-Flashcards"
gcloud run deploy super-flashcards --source . --region us-central1 --allow-unauthenticated
```

### View Logs
```powershell
gcloud run logs read super-flashcards --region=us-central1 --limit=50
```

---

## Secret Manager

| Secret Name | Purpose |
|-------------|---------|
| `db-password` | Database password (sqlserver user) |
| `google-oauth-client-secret` | Google OAuth for authentication |
| `ELEVENLABS_API_KEY` | ElevenLabs API for pronunciation |
| `GEMINI_API_KEY` | Gemini API |
| `openai-api-key` | OpenAI API |

### Access Secret
```powershell
gcloud secrets versions access latest --secret="db-password"
```

---

## SQL Connectivity
```powershell
# Connect via sqlcmd
sqlcmd -S 35.224.242.223,1433 -U sqlserver -P "$(gcloud secrets versions access latest --secret='db-password')" -d LanguageLearning

# Quick test
sqlcmd -S 35.224.242.223,1433 -U sqlserver -P "$(gcloud secrets versions access latest --secret='db-password')" -d LanguageLearning -Q "SELECT TOP 5 * FROM INFORMATION_SCHEMA.TABLES"
```

---

## Compliance Directives

### Before ANY Work (LL-045)
1. ✅ Read this entire CLAUDE.md file
2. ✅ State what you learned: "Service is super-flashcards, database is LanguageLearning"
3. ❌ Never invent infrastructure values

### Definition of Done (MANDATORY for ALL Tasks)

Before sending a completion handoff, ALL items must be checked:

**Code**:
- [ ] Code changes complete
- [ ] Tests pass (if applicable)

**Git (MANDATORY)**:
- [ ] All changes staged: `git add [files]`
- [ ] Committed: `git commit -m "type: description (vX.X.X)"`
- [ ] Pushed: `git push origin main`

**Deployment (MANDATORY)**:
- [ ] Deployed: `gcloud run deploy super-flashcards --source . --region us-central1`
- [ ] Health check passes: `curl https://flashcards.rentyourcio.com/health`
- [ ] Version matches: Response shows new version

**UAT (MANDATORY for features)**:
- [ ] Claude.ai creates UAT checklist
- [ ] Corey executes UAT
- [ ] UAT results submitted to Claude.ai
- [ ] UAT PASSED (all critical tests green)
- [ ] UAT results stored in MetaPM

**Handoff (MANDATORY)**:
- [ ] Handoff created with deployment verification
- [ ] Uploaded to GCS
- [ ] URL provided

⚠️ "Next steps: Deploy" is NOT acceptable. Deploy first, then send handoff.
⚠️ A feature is NOT complete until UAT passes.

### Before ANY Handoff (LL-030, LL-049)
1. ✅ Git commit and push (MANDATORY)
2. ✅ Deploy code (you own deployment)
3. ✅ Run tests: `pytest tests/ -v`
4. ✅ Verify deployment with PINEAPPLE test
5. ✅ Include test output in handoff
6. ❌ Never say "complete" without proof

### Locked Vocabulary (LL-049)
These words require proof (deployed revision + test output):
- "Complete" / "Done" / "Finished" / "Ready"
- "Implemented" / "Fixed" / "Working"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coreyprator) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
