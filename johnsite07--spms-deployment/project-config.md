---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status

This repository is being built from scratch. As of now there is no application or infrastructure code — only docs and the source milestone documents under [docs/milestones/](docs/milestones/). Those milestones are the authoritative blueprint; the distilled, working specs live under [docs/](docs/). Everything below is the **target** state being built toward, not yet-existing code; verify a path/command exists before relying on it.

The two specs that matter most when implementing:
- **Application spec** → [docs/requirements/](docs/requirements/) (use cases, business rules, NFRs) and [docs/architecture/domain-model.md](docs/architecture/domain-model.md) (the 14-class object model). Source: Milestone 3.
- **Deployment spec** → [docs/architecture/overview.md](docs/architecture/overview.md) and this file. Source: Milestone 4 (`docs/milestones/SecureVault_Milestone4_Deployment.docx`).

## What this is

SecureVault (SPMS — Secure Password Management System) is a web-based, zero-knowledge password manager: a single user stores, generates, and manages credentials and sensitive documents in an encrypted vault (AES-256 at rest, TLS in transit, 2FA). It is a containerised Node.js/Express app over MySQL, deployed to Google Cloud Platform. This repo (`SPMS_Deployment`) is the **deployment & DevOps deliverable** and also the home for building the app: it holds the application source, the Terraform that provisions every cloud resource, and the GitHub Actions pipelines that ship it.

This is a Seneca PRG800 academic project (the deployment work is Milestone 4; the app is specified by Milestones 1–3). Two hard constraints shape every decision:
- **Stay inside the $300 GCP free-trial credit** over a ~2-month window. Favour scale-to-zero and shared-core tiers; a single `terraform destroy` must return all spend to zero after grading.
- **Zero-knowledge / least-privilege posture.** Encryption is applied at the application layer and at rest — infrastructure never handles plaintext vault contents.

## Target repository layout

```
SPMS_Deployment/
├── .github/workflows/
│   ├── ci.yml          # lint, test, terraform plan — runs on PRs
│   └── cd.yml          # build, push, apply, deploy — runs on push to main
├── app/                # Node.js / Express source + Dockerfile (also serves the built SPA)
├── client/             # React + Vite single-page app (frontend)
├── terraform/
│   ├── main.tf         # wires modules together
│   ├── variables.tf
│   ├── outputs.tf
│   ├── backend.tf      # GCS remote state backend
│   └── modules/
│       ├── network/    # VPC + Direct VPC egress
│       ├── iam/        # service accounts + Workload Identity Federation
│       ├── data/       # Cloud SQL + Cloud Storage
│       ├── app/        # Cloud Run + Artifact Registry
│       └── secrets/    # Secret Manager
├── docs/               # structured documentation (see Documentation below)
│   ├── architecture/   # overview + domain model
│   ├── requirements/   # functional + non-functional (app spec)
│   ├── decisions/      # ADRs
│   ├── deployment/ · runbooks/ · guides/
│   └── milestones/     # source PRG800 deliverables M1–M4 (authoritative)
└── README.md
```

## Architecture (big picture)

Three zones: **GitHub** (code + CI/CD), the **GCP project** (all runtime resources), and **external actors/services** (users, 2FA, SMTP).

- **Compute:** Cloud Run (`google_cloud_run_v2_service`), 1 vCPU / 512 MiB, `min=0` / `max=2`. Terminates HTTPS via a Google-managed cert; scales to zero when idle. The single Express container serves **both** the JSON API (`/api/*`) and the built React SPA (static assets + an `index.html` fallback for client-side routes) — Option A, no separate frontend host or CDN (see ADR 0009).
- **Database:** Cloud SQL for MySQL **8.0** (`ENTERPRISE` edition — MySQL 8.4 requires Enterprise Plus, whose smallest tiers break the budget; see PRD 0002 outcome), `db-f1-micro`, 10 GB SSD, **private IP only** — never publicly exposed, reached from Cloud Run over the VPC via **Direct VPC egress** (deliberately not a Serverless VPC connector, to avoid an always-on cost). This is the largest cost line; it can be stopped between sessions.
- **Storage:** two Cloud Storage buckets — one versioned bucket for Terraform remote state, one for encrypted document blobs (lifecycle rules expire old objects).
- **Secrets:** Secret Manager holds DB creds, JWT key, AES key, SMTP creds (~6 secrets), injected into Cloud Run at start-up under its own service account. Secrets never live in source, the Docker image, or committed env files. Rotate by adding a new secret version — no code redeploy needed.
- **Images:** Artifact Registry (Docker format, single regional repo). **Images are tagged by git commit SHA** so every revision traces to an exact commit.
- **Identity:** separate least-privilege service accounts for pipeline vs. runtime — compromise of one does not grant the other's access. The pipeline stores **no service-account key**; it uses Workload Identity Federation (see below).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JohnSite07/SPMS_Deployment](https://github.com/JohnSite07/SPMS_Deployment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
