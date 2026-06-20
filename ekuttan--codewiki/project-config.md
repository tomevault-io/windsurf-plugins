---
trigger: always_on
description: >
---


# CodeWiki — Product Knowledge Base Generator

Scan code, fetch product context, discover user flows, and generate a structured `codewiki/` knowledge base.

Output templates are in this skill's `templates/` directory. Read each template only when generating that specific file — do NOT load all templates at once.

## HARD RULES

1. **Phase 1 is not optional.** Product context is the difference between a useful wiki and a mechanical code listing. Complete it before scanning. If user can't answer, use the code-first fallback.
2. **Multi-repo workspace must be verified before scanning.** If repos aren't in a shared parent folder, stop and help the user set it up.
3. **Never scan all repos simultaneously.** Use Scan → Write to `_scratch/` → Carry 30-line summary → Next. This is how you survive large multi-repo projects.
4. **Never skip a service in microservices.** User flows that touch 5 services are useless if one is undocumented. Every service gets at least a lightweight scan.
5. **Scan in priority order.** Infrastructure/contracts → gateway → frontends → core business services → supporting services.
6. **Report progress between repos.** Tell the user what you found and which repo is next. Do not ask for permission between repos — just continue. Only pause on errors or contradictions.
7. **On startup, check if `codewiki/_scratch/` exists.** If it does, offer to resume from previous scan data.
8. **Always use `codewiki/` (no dot prefix).** Ensures visibility in file explorers and default git tracking.
9. **Adaptive detection.** If hardcoded grep patterns don't match anything for a component, read the entry point file and trace outward. Don't report "no endpoints found" without trying alternative approaches.
10. **WebFetch failures.** Report the specific error (401, 404, timeout) and continue. Never silently skip a URL.

---

## Phase 1: Product Context Gathering (MANDATORY — DO NOT SKIP)

> **STOP. You MUST complete Phase 1 before scanning any code.**
> Do not run `ls`, `grep`, or read any files until Phase 1 is done.
> If the user says "just scan the code" or tries to skip, explain that
> the wiki quality depends heavily on product context and ask them to
> answer at least questions 1–3 before proceeding.

### Step 1a: Ask the User

Ask all of these in a single message. Mark which are required vs. optional:

1. **Product name** *(required)* — What is this product called?
2. **One-liner** *(required)* — Describe it in one sentence. What does it do and for whom?
3. **Target users** *(required)* — Who uses this? (e.g., developers, marketers, internal ops team)
4. **Product URLs** *(optional but strongly recommended)* — Provide any of:
   - Marketing website / landing page
   - Documentation site
   - API docs URL
   - App Store / Play Store listing
   - README or repo URL (if different from the working directory)
5. **Repo list** *(required for multi-repo)* — If this is a multi-repo product:
   - List all repos (backend, frontend, mobile app, comms, infra, etc.)
   - Are they all cloned into a single parent folder? If not, I'll ask you to set that up.
   - Provide git clone URLs for any not yet cloned.
6. **Anything the code won't tell me?** *(optional)* — Business context, upcoming changes, non-obvious terminology, areas of tech debt, inter-service communication patterns, etc.

**Code-first fallback:** If user says they can't answer 1–3, say: "No problem — I'll scan the code first and come back with my best guess for you to correct." Infer product name from package.json `name` field or README title, one-liner from package.json `description` or README first paragraph, users from UI/API patterns (admin dashboard = internal ops, public signup = consumers, API-only = developers). Present guesses for confirmation before continuing to Phase 2.

### Step 1b: Multi-Repo Workspace Setup

> **For multi-repo products, all repos MUST be cloned into a single parent folder.**
> This is how CodeWiki discovers cross-repo relationships.

Expected structure:
```
<product-name>/              ← master folder
├── backend/                 ← git clone <backend-url>
├── frontend/                ← git clone <frontend-url>
├── mobile-app/              ← git clone <mobile-url>
├── comms-service/           ← git clone <comms-url>
├── shared-libs/             ← git clone <shared-url>
└── infra/                   ← git clone <infra-url>
```

**If repos are not yet organized this way**, give the user exact commands:
```bash
mkdir -p ~/projects/<product-name> && cd ~/projects/<product-name>
git clone <url-1> <folder-name-1>
git clone <url-2> <folder-name-2>
# ... etc
```

Then ask the user to `cd` into the master folder and run CodeWiki from there.

**Verify the setup:**
```bash
ls -la
for dir in */; do echo "$dir: $(git -C "$dir" remote get-url origin 2>/dev/null || echo 'not a git repo')"; done
```

If any repos are missing, tell the user which ones and provide the clone command. Do NOT proceed to Phase 2 until the workspace is verified.

### Step 1c: Fetch External Context

For each URL provided, use WebFetch to extract product context.

**Error handling:** If a URL fails (auth wall, 404, timeout), tell the user which URL failed and why, then continue with the rest. Do not silently skip.

For each successful fetch, extract:
- Landing page copy, feature descriptions, value propositions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekuttan/codewiki](https://github.com/ekuttan/codewiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
