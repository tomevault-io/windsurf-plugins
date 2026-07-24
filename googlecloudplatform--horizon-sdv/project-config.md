---
trigger: always_on
description: <!-- Copyright (c) 2026 Accenture, All Rights Reserved.
---

<!-- Copyright (c) 2026 Accenture, All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License. -->

# Gemini CLI integration

Scripts and environment for running the [Gemini CLI](https://geminicli.com/docs/cli/headless/) in headless mode (e.g. from Jenkins or Argo). Used by the AAOS Builder AI review step and the Gemini AI Assistant utility.

## Argo Workflows and Jenkins

**Entrypoint:** **`workloads/common/agentic-ai/gemini/run_ai_review.sh`** — same contract for **Argo** (**aaos-builder** inline **`gemini-review`**), **Jenkins** (AAOS, ABFS, OpenBSW, **`gemini_ai_assistant`**), and **CVD/CTS** (**`cvdPipeline.groovy`**). Callers set **`PIPELINE_REPO_ROOT`** (or **`WORKSPACE`** / **`REPO_ROOT`**), **`GEMINI_*`**, **`CLOUD_*`**, optional **`GEMINI_HOOK_DIR`** / **`GEMINI_HOOK_PROFILE`**, optional **`GEMINI_ARTIFACTS_COMMAND`** / **`GEMINI_POST_REVIEW_COMMAND`**, and **`GEMINI_SKIP_MOVE_ARTIFACTS`** when outputs already live under **`WORKSPACE`**.

**Argo Workflows:** **aaos-builder**, **cvd-launcher**, and **cts-execution** run **`run_ai_review.sh`** from an inline **`gemini-review`** template in each workflow chart (native **`volumeMounts`** for workflow PVCs — **`templateRef`** does not merge per-step mounts on this controller). CVD/CTS first run **`gemini_argo_prepare_staging.sh`** on PVC **`gemini-test-results`**, then **`gemini-review`** with **`GEMINI_SKIP_MOVE_ARTIFACTS=1`** and profile **`cvd`** / **`cts`** post-hooks. See **[tests README](../../../../workloads/android/pipelines/tests/README.md)** and chart READMEs under **`workloads/android/pipelines/tests/*/helm/`**.

### Argo integration (inline templates)

| Step | Where |
|------|--------|
| Prepare staging (CVD/CTS) | **`workloads/android/pipelines/tests/gemini_argo_prepare_staging.sh`** — **`prepare-gemini-cvd`** / **`prepare-gemini-cts`** in each chart’s **`_gemini.tpl`** |
| **`gemini-review`** pod | Inline in **`aaos_builder/helm/templates/workflow/_gemini-review.tpl`**, **`cvd_launcher/helm/.../_gemini.tpl`**, **`cts_execution/helm/.../_gemini.tpl`** — all call **`run_ai_review.sh`** |
| Cluster **`ai-review`** + **`templateRef`** | Not used (AAOS, CVD, and CTS inline only on this controller) — see **[AI review reuse options](../../../guides/ai_review_reuse_options.md)** |

## Scripts

| Script | Purpose |
|--------|--------|
| `gemini_environment.sh` | Sets and exports variables (prompt file, command line, artifact path, preview/location). Source this before other scripts. |
| `gemini_initialise.sh` | Cleans artifacts, installs/upgrades gemini-cli, writes `.gemini/settings.json`. If skills.yaml is found (`GEMINI_SKILLS_YAML` or `skills.yaml` next to prompt path), runs `gemini_skills_from_yaml.py` to populate `.gemini/skills/`. Skills file is always named `skills.yaml`. In Jenkins / `CI=true` / Argo workflow runs, copies every `*.toml` under `gemini/policies/` into `.gemini/policies/` (see [Workspace policy files](#workspace-policy-files-toml)). |
| `gemini_analysis.sh` | One prompt → single run. Two or three prompts → **sequenced**: **one Gemini CLI invocation per step**, each with its own JSON output file (`headless_output_stepN_<timestamp>_<random>.json`). Prior-step context is file-based (`stepN_output.md`). Step 3 includes **both** `step1_output.md` and `step2_output.md` when present (optional byte caps). Set `GEMINI_PROMPT_FILE` (step 1), optionally `GEMINI_PROMPT_FILE_2` and `GEMINI_PROMPT_FILE_3`. Default when unset: AAOS three-step. Requires `jq` for extraction. See [Sequenced analysis](#sequenced-analysis). |
| `run_ai_review.sh` | **Shared runner:** reads **`GEMINI_*`**, **`CLOUD_*`**, repo root (**`PIPELINE_REPO_ROOT`** / **`WORKSPACE`** / **`REPO_ROOT`**), optional hooks, **`gemini_initialise.sh`** + **`gemini_analysis.sh`**, optional **`move_gemini_artifacts`**, client-error zip handling. |
| `workloads/android/pipelines/tests/gemini_argo_prepare_staging.sh` | **Argo only (CVD/CTS):** merge ephemeral VM artifacts into **`/workspace/test-results`** on the **`gemini-test-results`** PVC before **`gemini-review`**. Set **`GEMINI_PREPARE_MODE`** to **`cvd`** or **`cts`**. Invoked from **`prepare-gemini-cvd`** / **`prepare-gemini-cts`** in chart **`_gemini.tpl`**. |
| `review-pre-<profile>.sh`, `review-post-<profile>.sh` | Optional hooks under **`${REPO_ROOT}/${GEMINI_HOOK_DIR}`** when **`GEMINI_HOOK_DIR`** is set. **`run_ai_review.sh`** dispatches by **`GEMINI_HOOK_PROFILE`**. **AAOS** uses profile **`aaos`**; **CVD/CTS Argo** use **`cvd`** / **`cts`** (**`review-post-*.sh`** under **`cvd_launcher/hooks`** and **`cts_execution/hooks`** runs **`gemini_storage.sh`** after analysis). |

## Key environment variables


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoogleCloudPlatform/horizon-sdv](https://github.com/GoogleCloudPlatform/horizon-sdv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
