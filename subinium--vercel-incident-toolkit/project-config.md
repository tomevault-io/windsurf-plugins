---
trigger: always_on
description: Vercel account hardening and incident response. Use when the user mentions a Vercel breach/incident, asks to audit or rotate Vercel environment variables, mark env vars sensitive, or respond to leaked Vercel tokens. Scope is Vercel only (not Netlify/Cloudflare). Covers four flows — audit, harden, incident response, per-vendor rotation — plus prevention guidance.
---


# Vercel Security Skill

This skill handles Vercel account hardening and incident response. Four user flows exist — **identify which flow the user is in before running any script**.

## Decision tree

If the user's intent is not clear from their prompt, ask one short question:

> "Which best matches: audit only / harden (mark everything sensitive) / active incident / single-vendor rotation?"

| User signal | Flow | Destructive? |
|---|---|---|
| "check what secrets I have", "just list my env vars" | **A. Audit** | no |
| "mark everything sensitive", "lock this down", "we're fine but want to harden" | **B. Harden** | yes (re-uploads values) |
| "breach", "got hacked", "leaked token", "suspicious deploy", "ex-teammate still had access" | **C. Incident response** | yes (rotates 7 internal secrets) |
| "rotate my Supabase key", "rotate DB password", "quarterly rotation" | **D. Per-vendor rotation** | yes (one key at a time) |

## Threat model (this skill is built around it)

Vercel stores env vars in three modes with different attack-surface properties:

| Type | Where the decryption key lives | Post-breach assumption |
|---|---|---|
| `plain` | Nowhere — stored as plaintext | **Assume leaked.** Treat as if the value is in the attacker's hands. |
| `encrypted` | Vercel internal KMS. Decrypted server-side at runtime **and** on-demand for the dashboard. | **Assume leaked** in any breach that reaches internal services (e.g. Vercel April 2026). A decryption key accessible to Vercel is accessible to an attacker in that environment. |
| `sensitive` | Decrypted only inside the build/runtime sandbox. Not readable by the dashboard or API after creation. | **Probably safe** unless the breach reached the build infrastructure itself. Lower probability — but not zero. |

Rules that follow from this model:
1. On a confirmed breach, treat every `plain`/`encrypted` value as already stolen and **rotate**, don't just harden. Hardening preserves the current (already-leaked) value — it's useful for future resilience, not for current mitigation.
2. `sensitive` alone is not a silver bullet. If the attacker reached the build infrastructure, sensitive values used at build-time leak too. Prefer *runtime-only* use of sensitive secrets where possible.
3. When in doubt about which tier a value is in, run `scripts/audit.py` — it labels every var.

## Lingering threats (what comes after the initial leak)

Credential theft is the visible part of a breach. The invisible part is what the attacker left behind:

- **Backdoored builds / deployments.** Malicious code injected into a past production build is still serving traffic. Mitigation: list all production deploys since the earliest suspected compromise window (`vercel ls --prod`), diff HEAD vs deployed git SHA, force a clean redeploy from a known-good commit.
- **Rootkit in project config.** `vercel.json` rewrites, redirects, headers, or function regions altered to exfil data or inject scripts. Mitigation: `git diff` `vercel.json` against a known-good revision, inspect dashboard Project → Settings for unexpected values.
- **Unauthorized team members or tokens.** Attacker added a service token or invited a member with low scrutiny. Mitigation: Teams → Settings → Members and Tokens → audit every row, revoke anything unrecognized.
- **Deploy hook URLs exfil.** Deploy hooks (`/v1/integrations/deploy/...`) are unauthenticated — anyone with the URL can trigger a build. Mitigation: Project → Settings → Git → Deploy Hooks → rotate every hook.
- **Supply-chain injection via internal npm tokens.** If any deploy published an npm package, treat those versions as tainted until verified. Mitigation: check npm audit logs, rotate `NPM_TOKEN` anywhere it was used, consider unpublishing questionable versions.
- **Serverless function warm instances** still holding old env. Rotation doesn't kill warm Lambdas — force scale-down via redeploy.
- **Preview deployments pinned to old env.** Old previews retain old env values and often have weaker auth. Mitigation: `vercel remove --safe <deployment>` for stale previews.
- **Data in logs/analytics.** Attacker may have dumped request logs containing tokens. Mitigation: review Log Drains, Analytics, and any third-party logging sink for data retention that needs purging.

Always surface 2–3 of these when running Flow C — don't let the user believe rotating env vars is the whole job.

## Post-incident monitoring (stays on for weeks, not hours)

See `runbooks/03-post-incident-monitoring.md`. Short version:

- Re-run `scripts/audit.py` weekly and diff for 4+ weeks. New env vars, new projects, new team members in the diff = investigate.
- Enable Audit Log email alerts for high-risk actions (member added, token created, deploy protection disabled).
- Deploy a **canary env var** — an unused secret whose only purpose is to trigger an alert if it's ever used. See `runbooks/03-post-incident-monitoring.md` for patterns.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [subinium/vercel-incident-toolkit](https://github.com/subinium/vercel-incident-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
