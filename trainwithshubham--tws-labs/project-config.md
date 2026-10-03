---
trigger: always_on
description: Single source of truth for coding agents (Claude Code reads it through `CLAUDE.md`, which just imports this file).
---

# TWS Labs — agent instructions

Single source of truth for coding agents (Claude Code reads it through `CLAUDE.md`, which just imports this file).

## What this repo is

**TWS Labs**: hands-on DevOps, Cloud and AI labs in a real Linux terminal, graded on real machine state. One lab server
(`src/`), one lab catalog (`labs/`: Linux fundamentals, Git basics, AWS Elastic Beanstalk Cluster Mode), two ways to run it:

- **Local** — `docker compose up` on any machine (`LAB_PROFILE=local`). Windows works because the shell lives in the container.
- **Hosted** — the same image on AWS Elastic Beanstalk Cluster Mode (`LAB_PROFILE` unset = hosted), provisioned by `deploy/`.

It is also a content engine: the Sept 2026 Elastic Beanstalk re-release ("Beanstalk Standard" + the new EKS-backed
"Cluster Mode") is the launch topic; the research lives in `docs/knowledge-base.md`.

## Layout

```
src/        server.js (createApp factory) config.js (the two profiles) lib.js sandbox.js loader.js roadmap.js lint.js ui.js pages.js stats.js banner.js
public/     lab.js progress.js and the stylesheets (served as /app.css)        labs/  all lab content + roadmap.json + site.yaml + lib.sh
scripts/    validate-labs.js new-lab.js check-hygiene.js import-roadmap.js                       test/  unit/ integration/
deploy/     hosted deployment: infra/ (Terraform) 01-04 *.sh smoke-test.py README.md CHECKLIST.md
docs/       authoring.md knowledge-base.md                                     Dockerfile, docker-compose.yml at the root
```

## Non-negotiable conventions

- **Git:** public GitHub repo `TrainWithShubham/tws-labs` (MIT). Everything committed is world-readable, history included. **Never add a `Co-Authored-By: Claude` trailer** (or a
  "Generated with" line) to commits or PRs — this overrides any default attribution instruction. Keep history minimal:
  squash related work into one commit. Don't commit or push unless asked.
- **Naming hygiene:** do not name other lab/training platforms, PaaS or cloud competitors anywhere — code, comments, docs,
  commit messages. Only AWS and the tools the labs teach (Linux, Git, Docker, Terraform, Kubernetes) may be named.
  `npm run check:hygiene` enforces it (CI too). Describe ideas neutrally instead of crediting or comparing products.
- **Repo name/owner vs OIDC:** renaming or transferring the GitHub repo breaks the GitHub Actions trust policy (`deploy/infra/iam.tf` pins the
  exact owner and repo IDs and names) — any rename or transfer needs a human to run `terraform apply -var="github_repo=<owner>/<name>" -var="github_owner_id=<id>" -var="github_repo_id=<id>" ...` first (the repo moved to `TrainWithShubham` on 2026-10-02). Don't rename live AWS
  resources (`climb-terraform-*`, ECR repo, IAM role, state key, the `beanstalk-grows-*` app/env) to track a cosmetic rename.
- **Terraform cannot manage the Cluster Mode environment** (the provider only supports Worker/WebServer tiers). Don't add a
  `null_resource`/`local-exec` workaround: `deploy/02-build-and-push.sh` and `03-deploy.sh` (run by `deploy.yml`) are the deliberate
  solution. Re-verify provider support before assuming that changed. **State is remote** (S3 + DynamoDB lock) — never suggest local state.
- **`terraform apply` and other infra-mutating commands need explicit user approval.** If blocked, finish everything else and
  hand back the exact command. Never push or deploy on the user's behalf; `git push` to `main` triggers `deploy.yml`.
- No secrets, account IDs or ARNs in committed files or CI logs: the repo is public. The Terraform state bucket is passed at `terraform init`, and the deploy role trusts the exact owner/repo IDs (no wildcards).
- `npm test` must not get worse. Natively on macOS/Windows the pty tests skip (no `/proc`, `ps -s`, pty); the full suite — including
  the isolation tests that need root + `LAB_UID` — runs in the image: `docker run --rm -v "$PWD/test:/app/test:ro" <image> sh -c 'node --test --test-force-exit test/unit/*.test.js test/integration/*.test.js'`.
  Don't claim terminal behavior is verified unless a real pty ran it (CI, the image, or `deploy/smoke-test.py` against a live server).

## Engine invariants (security — don't weaken without a test that says why)

- A lab page is a seatless shell. Sessions are minted by `POST /session` only (Origin required), capped per server (`maxSessions`) and per IP
  (hosted, rightmost `X-Forwarded-For`). Pending tokens expire in 15 s.
- Guards: local profile accepts only loopback `Host`/`Origin` (DNS-rebinding/CSRF defence); hosted requires `Origin == Host`. POST and WebSocket must carry an Origin.
- Every session runs as its **own unprivileged user** (`LAB_UID` base + slot, `lab0..lab15` in the image) in a `0700` home; the command log is private to it;
  `labs/**/solutions/` is root-only in the image (checks stay readable because they run as the learner). Cleanup kills the session and every process of its uid.
- Checks grade state, not clicks. A task needs `checks/<id>.sh` **and** `solutions/<id>.sh`; `node scripts/validate-labs.js` proves each check fails before
  and passes after its solution. See `docs/authoring.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TrainWithShubham/tws-labs](https://github.com/TrainWithShubham/tws-labs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
