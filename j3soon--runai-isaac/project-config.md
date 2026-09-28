---
trigger: always_on
description: This repository is a documentation-and-assets workspace for running NVIDIA Isaac workloads on Run:ai, with supporting Docker images and shell utilities.
---

# Repository Guidelines

## Project Structure & Module Organization
This repository is a documentation-and-assets workspace for running NVIDIA Isaac workloads on Run:ai, with supporting Docker images and shell utilities.

- `docker/`: Dockerfiles and image-specific assets (for example `docker/pytorch-mnist/Dockerfile`, `docker/isaac-sim/`, `docker/isaac-lab/`).
- Keep each Docker-backed application and its guide together under `docker/<name>/` unless an established product layout requires otherwise.
- For products with multiple major model/image variants (for example Cosmos, GR00T), prefer separate versioned folders (for example `docker/<name>-n1/`, `docker/<name>-n1.6/`) and matching versioned application guides.
- `scripts/`: Operational shell scripts, grouped by purpose (`scripts/docker/run.sh`, `scripts/admin/`, `scripts/vpn/`).
- `docs/`: User/developer documentation and screenshots (`docs/assets/`).
- `thirdparty/omnicli/`: Bundled Omniverse CLI binaries used by `/run.sh`.
- `.github/workflows/`: Per-image CI workflows for images built and published by this repository.
- `skills/<name>/SKILL.md`: Repository task guides for agents (`add-runai-application`, `launch-runai-workload`, `run-isaac-lab-benchmark`, `admin-debug-runai-node`), with optional `references/` and `scripts/`. `.agents/skills` and `.claude/skills` are symlinks to this directory, so each harness finds the same files; add and edit skills under `skills/` only.

## Agent Skills and Durable Notes

- A request to "use the skill" for a task refers to `skills/`. Harnesses that load skills automatically reach them through the `.agents/skills` and `.claude/skills` symlinks; otherwise treat them as plain files: locate the task's `SKILL.md`, read it, and follow it alongside this document.
- **Development machines here are ephemeral and are reset periodically, so never keep a learning only in an assistant's machine-local memory** (Claude Code's `memory/` directory or any equivalent). Anything written only there looks persisted and then disappears with the VM, and teammates and other agents never see it. The repository is the only durable store: record reusable pitfalls and their workarounds in the relevant skill's `references/` notes (for example `skills/add-runai-application/references/validation-notes.md`), cluster/node failures in `troubleshooting.md`, image-specific behavior in that image's `docker/<name>/README.md`, and workflow changes in the relevant `SKILL.md`. Machine-local memory is for pointing back at this rule, nothing else.
- Site- and machine-specific values (cluster identity, project, NFS server and export, node-pool hardware, account names, host GPU and RAM figures) cannot go in tracked files because this repository is public. Put them in `secrets/environment-notes.md`, which `.gitignore` excludes, and keep the environment-agnostic lesson in the tracked files above. Note that `secrets/` does not survive a VM rebuild either, so it is re-provisioned alongside the VPN certs.
- Publishing a Docker image is the user's action, not an agent's. Build and validate against a `local/<name>:<tag>` tag, never tag or push `j3soon/*`, and hand over the exact `docker build` / `docker tag` / `docker push` commands. Being logged in to a registry is not authorization to publish. State plainly which validation steps remain blocked until the image is published rather than implying they ran.
- `docs/developer-notes.md` is maintained by humans. Agents must not add, edit, or reorganize entries there, even when a finding would fit its format; put the finding in the skill's `references/` notes instead and let a maintainer promote it if it belongs in the human-facing document. Reading and linking to it is fine.
- Keep such notes environment-agnostic. Use placeholders (`<YOUR_LAB>`, `<NODE_NAME>`, `<YOUR_USERNAME>`) rather than real hostnames, accounts, or tokens.
- Read the relevant skill's `references/` notes **before** starting work, not while debugging a failure. They exist because each entry already cost someone a wrong conclusion, and the cost repeats when they are consulted late. In particular, do not try to screenshot or screen-record a simulator GUI: `x11grab`, `xwd`, and `import` all read an X surface that Isaac Sim's Vulkan swapchain does not present to, so they yield black or empty images no matter how correct the window looks in `xwininfo`. Use the application's own recorder (`--video`, in-app capture); see `skills/add-runai-application/references/validation-notes.md`.

## Build, Test, and Development Commands
- `docker build -t local/runai-pytorch-mnist -f docker/pytorch-mnist/Dockerfile .`: Build the example training image locally.
- `bash scripts/docker/run.sh "echo hello"`: Smoke-test the helper entrypoint script behavior.
- `bash -n scripts/docker/run.sh` (and other `scripts/**/*.sh`): Shell syntax check before submitting changes.
- `chmod +x scripts/<path>.sh`: Restore executable bit if a script was edited on Windows or copied incorrectly.

Use `README.md` and `install.md` for end-to-end setup and cluster-specific steps.

## Coding Style & Naming Conventions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [j3soon/runai-isaac](https://github.com/j3soon/runai-isaac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
