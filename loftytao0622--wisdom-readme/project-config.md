---
trigger: always_on
description: Generate rigorous, evidence-based README documentation for backend, frontend, full-stack, CLI, and library projects. Use this skill when the user asks to "create a readme", "generate readme", "write documentation", "生成readme", "写文档", "生成项目文档", or requests Chinese/English README files.
---


# README Generator

Generate polished README documentation from the actual repository contents. Be strict about evidence: document what is present, skip what is unknown, and never invent architecture, deployment topology, API behavior, security posture, roadmap, or license terms.

## Core Principles

1. **Evidence first**: every feature, command, API endpoint, dependency, and architectural statement must be traceable to files in the repository.
2. **No secret exposure**: never read, summarize, quote, or list sensitive configuration files or credential-like values.
3. **No overwrite without consent**: if a target README file already exists, ask before overwriting unless the user explicitly requested overwrite.
4. **Language-aware output**: detect the project language first, then choose README sections, commands, and terminology that fit that ecosystem.
5. **Short enough to use**: prefer a focused README over a catalog of every file. Keep generated content concise, attractive, and skimmable.
6. **Human maintainer voice**: security, privacy, and license notes should read like a maintainer's first-person project note, not generic AI advice.

## Step 1: Safe Project Scan

Scan only files that are useful for documentation. Run discovery commands with explicit excludes.

### Must Exclude From Scanning

Do **not** read, grep, summarize, or display content from these files or directories:

- `.git/`, `.svn/`, `.hg/`
- `node_modules/`, `vendor/`, `.venv/`, `venv/`, `env/`, `__pycache__/`
- Frontend build output: `dist/`, `build/`, `.next/`, `.nuxt/`, `out/`, `.svelte-kit/`, `coverage/`
- Backend build output: `target/`, `bin/`, `obj/`, `.gradle/`, `build/`, `classes/`, `generated-sources/`
- JavaScript package caches: `.npm/`, `.pnpm-store/`, `.yarn/`
- Logs and dumps: `*.log`, `logs/`, `dump/`, `dumps/`, `*.dump`, `*.sql`, `*.sqlite`, `*.db`
- Private environment/config files: `.env`, `.env.*`, `*.env`, `application-local.yml`, `application-local.yaml`, `application-prod.yml`, `application-prod.yaml`, `bootstrap-prod.yml`, `bootstrap-prod.yaml`, `settings.local.py`, `local_settings.py`
- Credential/key material: `*.pem`, `*.key`, `*.p12`, `*.pfx`, `id_rsa*`, `id_ed25519*`, `credentials.*`, `secrets.*`, `secret.*`, `service-account*.json`, `firebase-adminsdk*.json`, `kubeconfig`, `*.kubeconfig`

When private files are discovered by name, do not call out their exact paths in the generated README. Use a general note such as "private configuration files are intentionally excluded from documentation" instead of writing "docker/deploy/.env contains sensitive information".

Safe examples may be read if they exist and are clearly examples:

- `.env.example`, `.env.sample`, `.env.template`
- `application-example.yml`, `application-example.yaml`
- `config.example.*`, `settings.example.*`

### Recommended Scan Commands

Use the platform's fastest available search tool. If `rg` is unavailable, use native shell alternatives.

1. Project structure, max depth 4, with excludes for secret files and build output.
2. Manifest files only:
   - Node: `package.json`, `pnpm-lock.yaml`, `yarn.lock`, `package-lock.json`, `vite.config.*`, `next.config.*`, `nuxt.config.*`, `tsconfig.json`
   - Python: `pyproject.toml`, `requirements*.txt`, `setup.py`, `setup.cfg`, `Pipfile`, `poetry.lock`, `uv.lock`
   - Java/Kotlin: `pom.xml`, `build.gradle`, `build.gradle.kts`, `settings.gradle`, `gradle.properties`, `src/main/resources/application*.yml`, `src/main/resources/application*.yaml`, excluding private profiles listed above
   - Go: `go.mod`, `go.sum`, `Makefile`, `Taskfile.yml`
   - Shared: `Dockerfile`, `docker-compose*.yml`, `.github/workflows/*`, `.gitlab-ci.yml`, `LICENSE`, `CHANGELOG.md`, `CONTRIBUTING.md`, `docs/**`
3. Entry points and routes:
   - read only source files likely to define startup, CLI, routing, or public APIs.
   - avoid scanning generated files, minified bundles, compiled output, and vendored dependencies.
4. Existing docs:
   - check `README.md`, `README.zh-CN.md`, `README.en.md`, `docs/`, `CHANGELOG.md`, `LICENSE`.
5. Git metadata:
   - repo URL and recent commit subjects may be used only as weak context; never invent features from commit titles alone.

## Step 2: README Existing-File Branch

Before writing, determine the target file(s):

- `zh`: `README.md` in Chinese, unless the user requests `README.zh-CN.md`.
- `en`: `README.md` in English, unless the user requests `README.en.md`.
- `both`: generate separate `README.zh-CN.md` and `README.en.md`. Optionally generate a short `README.md` index only when no `README.md` exists or the user approves.
- no argument: infer the primary language from existing docs/user request. If the user asks for bilingual docs or the project is meant for both audiences, use `both`.

If any target file already exists:

1. Stop before writing that file.
2. Tell the user which file exists.
3. Ask whether to:
   - overwrite it,
   - create a separate file such as `README.generated.md`, `README.zh-CN.md`, or `README.en.md`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LoftyTao0622/wisdom-readme](https://github.com/LoftyTao0622/wisdom-readme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
