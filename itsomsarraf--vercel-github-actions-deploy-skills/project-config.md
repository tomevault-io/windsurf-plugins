---
trigger: always_on
description: Set up GitHub Actions to deploy any Vercel project using the Git Author Override method, enabling teammates to deploy on the free Hobby plan. Use when the user asks about Vercel deployment via GitHub Actions, CI/CD for Vercel, letting teammates deploy on Vercel free plan, bypassing Vercel's Hobby plan deploy restrictions, or automating Vercel production deploys. Covers workflow setup, GitHub Secrets configuration, and package manager variants (bun, npm, pnpm).
---


# Vercel GitHub Actions Deploy (Git Author Override)

Deploy Vercel projects from GitHub Actions on the **free Hobby plan** — letting any teammate trigger production deploys.

## The Problem

Vercel's free plan ties deployments to the **account owner**. When a teammate pushes to `main`, Vercel checks the git commit author and rejects it. Normally requires Pro plan ($20/mo per member).

## How It Works

```
Teammate pushes to main
        ↓
GitHub Actions triggers
        ↓
Rewrites commit author to account owner (on CI runner only)
        ↓
Vercel CLI builds and deploys to production
```

- Runs on every push to `main` — by **anyone**
- Manual deploy via GitHub Actions tab (`workflow_dispatch`)
- Actual repo history stays **untouched** (rewrite only on disposable runner)
- Works on the **free Vercel plan**

## User Action Required — What You Need Before Starting

This skill generates the workflow file automatically, but **you must provide 5 values** that only you have access to. The AI assistant **cannot** obtain these for you.

### Checklist: Things You Must Do Manually

| # | Action | Where to Do It | What You Get |
|---|--------|----------------|--------------|
| 1 | **Create a Vercel deploy token** | Go to [vercel.com/account/tokens](https://vercel.com/account/tokens) → Create Token → Copy it | `VERCEL_TOKEN` |
| 2 | **Link your project to Vercel** | Run `npx vercel link` in your project root (follow prompts) | Creates `.vercel/project.json` |
| 3 | **Copy Org ID and Project ID** | Open `.vercel/project.json` → copy `orgId` and `projectId` values | `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID` |
| 4 | **Know the Vercel account owner's identity** | The email and name on the Vercel account that owns the project | `DEPLOY_EMAIL`, `DEPLOY_NAME` |
| 5 | **Add all 5 secrets to GitHub** | Go to your repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret** | Secrets stored in GitHub |

**IMPORTANT:** The AI assistant will create the workflow YAML file for you, but it **cannot** create the Vercel token, link your project, or add GitHub secrets — you must do steps 1-5 yourself.

### The 5 Required GitHub Secrets

Add each of these at: `https://github.com/<owner>/<repo>/settings/secrets/actions`

| Secret Name | Where to Get It | Example Value |
|-------------|-----------------|---------------|
| `VERCEL_TOKEN` | [vercel.com/account/tokens](https://vercel.com/account/tokens) → Create Token | `pZt7x...` (long string) |
| `VERCEL_ORG_ID` | `.vercel/project.json` → `"orgId"` field | `team_aBcDeFgHiJkLmN` |
| `VERCEL_PROJECT_ID` | `.vercel/project.json` → `"projectId"` field | `prj_xYzAbCdEfGhIjK` |
| `DEPLOY_EMAIL` | Email of the person who owns the Vercel project | `owner@example.com` |
| `DEPLOY_NAME` | Display name of the Vercel project owner | `Om Sarraf` |

### How to Get the Org ID and Project ID

```bash
# Step 1: Install Vercel CLI (if not installed)
npm install -g vercel

# Step 2: Link your project
npx vercel link
# → Follow prompts: select scope, link to existing project or create new

# Step 3: A file is created at .vercel/project.json
# It looks like this:
# {
#   "orgId": "team_aBcDeFgHiJkLmN",
#   "projectId": "prj_xYzAbCdEfGhIjK"
# }

# Step 4: Make sure .vercel is gitignored
echo ".vercel" >> .gitignore
```

## Quick Start (5 min)

Once you have all 5 secrets ready, setup takes under 5 minutes.

### Step 1: Pick your workflow

Choose based on your package manager:

- **Bun** (has `bun.lock`) → `examples/deploy-bun.yml`
- **npm** (has `package-lock.json`) → `examples/deploy-npm.yml`
- **pnpm** (has `pnpm-lock.yaml`) → `examples/deploy-pnpm.yml`

Copy the chosen file to `.github/workflows/deploy.yml` in your repo.

### Step 2: Add your 5 GitHub Secrets

**(You must do this manually — see "User Action Required" above)**

Go to `https://github.com/<owner>/<repo>/settings/secrets/actions` and add:

1. `VERCEL_TOKEN`
2. `VERCEL_ORG_ID`
3. `VERCEL_PROJECT_ID`
4. `DEPLOY_EMAIL`
5. `DEPLOY_NAME`

### Step 3: Push and deploy

```bash
git add .github/workflows/deploy.yml
git commit -m "ci: add Vercel deploy workflow"
git push origin main
```

Watch it deploy in the **Actions** tab of your GitHub repo.

## Workflow Template (Bun)

```yaml
name: Deploy to Vercel

on:
  push:
    branches: [main]
  workflow_dispatch:

env:
  VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
  VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Override git author to repo owner
        run: |
          git config user.email "${{ secrets.DEPLOY_EMAIL }}"
          git config user.name "${{ secrets.DEPLOY_NAME }}"
          git commit --amend --reset-author --no-edit

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - uses: oven-sh/setup-bun@v2


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itsOmSarraf/vercel-github-actions-deploy-skills](https://github.com/itsOmSarraf/vercel-github-actions-deploy-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
