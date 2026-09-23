---
trigger: always_on
description: Use the entire `HarmonyOS_Skills/harmonyos-agent-skills` repository as the online skill catalog for this project, including nested skills and future additions. Select skills for the actual task; a small fixed list of local skill names must not limit discovery.
---

# Repository Guidelines

## Online HarmonyOS Skill Routing

Use the entire `HarmonyOS_Skills/harmonyos-agent-skills` repository as the online skill catalog for this project, including nested skills and future additions. Select skills for the actual task; a small fixed list of local skill names must not limit discovery.

Sources:

- Repository: https://gitcode.com/HarmonyOS_Skills/harmonyos-agent-skills
- Current index: https://raw.gitcode.com/HarmonyOS_Skills/harmonyos-agent-skills/raw/main/README.md
- Root directory API: https://api.gitcode.com/api/v5/repos/HarmonyOS_Skills/harmonyos-agent-skills/git/trees/main?recursive=0&per_page=100&page=1
- Raw document base: https://raw.gitcode.com/HarmonyOS_Skills/harmonyos-agent-skills/raw/main/

For each new HarmonyOS development, investigation, review, or verification task:

1. Fetch the current index and root directory metadata in memory. Match the request against all relevant areas: design, ArkTS/ArkUI, SDK capabilities, architecture, stability, performance, testing, device tools, and release workflows. Follow new categories found upstream as well. Reuse fetched content within the same task and refresh discovery when its scope changes.
2. Browse relevant directories through the tree API. For each entry with `type: tree`, request `/git/trees/{entry.sha}?recursive=0&per_page=100&page=N`; preserve the parent path because child `path` values are relative to that tree. Read pages until a page contains fewer than 100 entries. The API's default page contains only 20 entries, and a `path=` query does not select a subtree. Do not mistake either result for the whole catalog.
3. Discover actual `SKILL.md` files, including skills nested under another skill's references or SDK collection. The README is a navigation aid; use the live directory tree when a skill is missing from it or a listed link returns 404. A request for a full catalog requires traversing every directory and page; ordinary work only requires traversing the branches relevant to that task.
4. Fetch the selected `SKILL.md` from the raw document base plus its repository-relative path. Read its description and instructions, then fetch the relevant referenced documents, indexes, examples, or dependent skills. Resolve relative URLs against the document that contains them. Read enough dependencies to use the skill correctly, without downloading the entire repository.
5. Apply the project constraints below: HarmonyOS 6.0+ / minimum API 20, target API 26, ArkUI V2, phone target, existing architecture, and current product scope. Check API-version annotations against the installed SDK; APIs introduced after 20 require a version guard and a working fallback. An upstream sample or general workflow does not authorize adding unrelated features, changing project conventions, or repeating approvals already given by the user.
6. Match upstream tool names to available operations, including `check_ets_files` / `arkts_check`, `build_project`, `init_project_path`, `start_app`, and UI/log tools. Inspect the actual tool schema; a different MCP server prefix does not require another server installation. Diagnose and report any genuinely missing executable or dependency.
7. Briefly identify the selected online skills and cite the URLs actually read when explaining technical decisions. Distinguish a guide being available from its workflow having been executed or validated.

Use an available web reader, `Invoke-WebRequest`, or `Invoke-RestMethod` with a finite timeout. Parse directory responses as JSON and retrieve reference text in memory. Do not clone, reinstall, cache, or back up the online knowledge catalog into project or user skill directories. If a task genuinely needs an upstream executable/template, inspect it and materialize only the necessary files in a task-specific temporary location; remove temporary downloads after use while retaining intended project outputs. Local SDKs, compilers, and project-owned test tools remain execution dependencies.

If online discovery is unavailable, report the failed source and the resulting coverage limit. Continue using supplied evidence and installed SDK diagnostics where sufficient; do not claim the online catalog was refreshed. Public HarmonyOS entrypoints may help reach online sources but are not required for this project's catalog routing.

## Build & Run Flow

Agents should use the provided tools, not raw `hvigorw`:

1. **`arkts_check`** on changed `.ets` files — catches ArkTS strict-mode violations faster than full build
2. **`build_project`** — incremental build by default; only pass `clean=true` if cache corruption is suspected
3. **`start_app`** — launch on device/emulator; requires prior successful build


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [end-web/HarmonyOS-book](https://github.com/end-web/HarmonyOS-book) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
