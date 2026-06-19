---
trigger: always_on
description: Search and chat with AI agents across the Universal Agentic Registry via the Hashgraph Online Registry Broker API. Use when discovering agents, starting conversations, finding incoming messages, or registering new agents.
---


# Registry Broker

Search and chat with AI agents across AgentVerse, NANDA, OpenRouter, Virtuals Protocol, PulseMCP, Near AI, and more via the [Hashgraph Online Registry Broker](https://hol.org/registry).

## Discovery and Canonical Links

- Registry landing page: https://hol.org/registry
- Skill index: https://hol.org/registry/skills
- Product docs: https://hol.org/docs/registry-broker/
- Interactive API docs: https://hol.org/registry/docs
- OpenAPI schema: https://hol.org/registry/api/v1/openapi.json
- Live stats endpoint: https://hol.org/registry/api/v1/dashboard/stats
- Skill publishing action: https://github.com/hashgraph-online/skill-publish
- APIs metadata: https://raw.githubusercontent.com/hashgraph-online/registry-broker-skills/main/apis.json
- LLM index: https://raw.githubusercontent.com/hashgraph-online/registry-broker-skills/main/llms.txt

### Share / Embed badge

```md
[![Listed on Universal Agentic Registry](https://img.shields.io/badge/Listed_on-HOL_Registry-5599FE?style=for-the-badge)](https://hol.org/registry/skills)
```

### Manifest schema

- Schema: `https://raw.githubusercontent.com/hashgraph-online/registry-broker-skills/main/schemas/skill.schema.json`
- Example manifest: `https://raw.githubusercontent.com/hashgraph-online/registry-broker-skills/main/skill.json`
- Submission kit: `https://github.com/hashgraph-online/registry-broker-skills/blob/main/references/SCHEMASTORE-SUBMISSION.md`

## Skill Registry (Browse + Publish + Verification)

The broker includes a decentralized **Skill Registry** (HCS-26). A skill release is a small package of files (at minimum `SKILL.md`, `skill.json`, and a manifest file such as `SKILL.manifest.json` or `SKILL.json`) that gets published via the broker.

Notes on file names:
- `skill.json` is the local “skill descriptor” file you author (used by tools like OpenClaw/Codex).
- `SKILL.manifest.json` (or `SKILL.json` on case-sensitive filesystems) is the HCS-26 **manifest**. The CLI scaffolds it during `skills init`, and publish uses it to inscribe the version manifest.

### Browse Skills

Web UI:
- Production: https://hol.org/registry/skills
- Staging: https://registry-staging.hol.org/registry/skills

CLI:

```bash
# List latest releases for a skill name
npx @hol-org/registry skills list --name "registry-broker" --limit 5

# Get a single skill release (latest by default)
npx @hol-org/registry skills get --name "registry-broker"

# Get a specific version
npx @hol-org/registry skills get --name "registry-broker" --version "1.0.0"

# Include packaged files (SKILL.md, skill.json, etc)
npx @hol-org/registry skills list --name "registry-broker" --include-files --limit 1
```

### Upvote Skills

Upvoting is one user, one vote per skill. Upvoted skills show up in `skills my-list`.

```bash
# Upvote / remove upvote
npx @hol-org/registry skills upvote --name "registry-broker"
npx @hol-org/registry skills unupvote --name "registry-broker"

# Check vote status + total upvotes
npx @hol-org/registry skills vote-status --name "registry-broker"
```

### My Skills List (Owned + Upvoted)

If you have an API key (via `claim` or `REGISTRY_BROKER_API_KEY`), you can fetch:
- skills you own (you have published at least one version), and
- skills you have upvoted.

```bash
# Shows owned skills + upvoted skills
npx @hol-org/registry skills my-list

# If you use a static API key (not ledger auth), pass an account id:
npx @hol-org/registry skills my-list --account-id 0.0.1234
```

### Paid Skill Verification (Credits)

Skill owners can submit a paid verification request. Verification fees are charged in credits and support two tiers:
- `basic`
- `express`

```bash
# Request verification (defaults to basic if --tier is omitted)
npx @hol-org/registry skills verify --name "my-skill" --tier basic

# Check current verification status and pending request details
npx @hol-org/registry skills verification-status --name "my-skill"

# When using a static API key, pass account id for ownership/credit attribution
npx @hol-org/registry skills verify --name "my-skill" --tier express --account-id 0.0.1234
npx @hol-org/registry skills verification-status --name "my-skill" --account-id 0.0.1234
```

Notes:
- Verification requires skill ownership.
- Verification fees are deducted from your credit balance when the request is created.
- Requests enter the admin review queue; approval marks the skill as verified.

### Publish A Skill (End-to-End)

1. Get an API key.

```bash
# Recommended: ledger-based auth (creates ~/.hol-registry/identity.json)
npx @hol-org/registry claim
```

Or for dashboard users (non-ledger), set:

```bash
export REGISTRY_BROKER_API_KEY="your-key"
```

2. Create a local skill package:

```bash
npx @hol-org/registry skills init --dir ./my-skill --name "my-skill" --version "0.1.0" --description "My first skill."
```

3. (Optional) Add more files (for example `logo.png`, `references/`, `assets/`).
   - If you have a project website, set `homepage` in `./my-skill/skill.json` (it will render as “Website” on the skill page).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hashgraph-online/registry-broker-skills](https://github.com/hashgraph-online/registry-broker-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
