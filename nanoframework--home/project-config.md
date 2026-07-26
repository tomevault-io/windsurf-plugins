---
trigger: always_on
description: When you are assigned to an issue titled **"Update {LibraryName} declaration"** in this repository, follow the steps below end-to-end. The issue body contains all the specific data you need: old and new values for native version and checksum, a link to the stubs build artifact, and a reference to the originating PR in the class library repository.
---

# GitHub Copilot Instructions

## Updating Native Assembly Declarations in nf-interpreter
When you are assigned to an issue titled **"Update {LibraryName} declaration"** in this repository, follow the steps below end-to-end. The issue body contains all the specific data you need: old and new values for native version and checksum, a link to the stubs build artifact, and a reference to the originating PR in the class library repository.

---
### 1. Understand the task
You are updating the native assembly declaration for a class library in [nanoframework/nf-interpreter](https://github.com/nanoframework/nf-interpreter). The CI pipeline in the class library repository detected that either the native checksum or the native version changed. It opened this issue to request that the corresponding stub files in nf-interpreter be updated to reflect the new values.
Before doing anything else, read the issue body carefully.

Extract all of the following before touching any file:

- **Library name** — from the issue title (`Update {LibraryName} declaration`)
- **Old checksum** — from the table (`Previously published` column)
- **New checksum** — from the table (`New` column)
- **Old native version** — from the table (`Previously published` column)
- **New native version** — from the table (`New` column)
- **Stubs artifact link** — direct zip URL or Azure DevOps build link in the issue body
- **Originating PR** — the PR in the class library repo that triggered this issue (e.g. `nanoframework/System.Device.Spi#171`)
- **This issue's number** — from the URL of the nf-interpreter issue (e.g. `#NNNN`)

---
### 2. Obtain the stub files
The issue body provides two ways to download the `stubs` artifact. Use whichever works in your environment:
**Option 1 — direct zip URL (no authentication, works for public projects)**
The issue body contains a direct URL ending in `&$format=zip`. Download it with:
```bash
curl -L -o /tmp/stubs.zip "<url from issue body>"
unzip /tmp/stubs.zip -d /tmp/stubs
```
**Option 2 — Azure CLI**
The issue body also contains a ready-to-run `az pipelines runs artifact download` command. Copy it from the issue and run it directly. It requires the `azure-devops` CLI extension and that you are already signed in (`az login`):
```bash
az extension add --name azure-devops  # if not already installed
az pipelines runs artifact download --run-id <buildId> --artifact-name stubs --path /tmp/stubs --org https://dev.azure.com/nanoframework --project "<project>"
```
**Option 3 — Azure DevOps REST API (for agents without filesystem zip support)**
If `curl`+`unzip` and the Azure CLI are unavailable, query individual files via the REST API. First, retrieve the artifact manifest to get file IDs:
```
GET https://dev.azure.com/nanoframework/{project}/_apis/build/builds/{buildId}/artifacts?artifactName=stubs&api-version=7.1-preview.5
```
The response JSON contains a `resource.data` field with the root file ID and a `resource.downloadUrl` for the zip. To fetch individual files, use:
```
GET https://dev.azure.com/nanoframework/{project}/_apis/build/builds/{buildId}/artifacts?artifactName=stubs&fileId={blobId}&fileName={filename}&api-version=7.1-preview.5
```
where `{blobId}` is the full blob hash for the file (obtained from the manifest) and `{filename}` is the file name (e.g. `nf_rt_events_native.cpp`).
---
The artifact contains a `Stubs/<LibraryName>/` subfolder with the `.cpp`, `.h`, and `.cmake` files.
Do NOT generate, infer, or modify the stub files. Use exactly what is in the artifact — these files were produced by the CI pipeline and are the authoritative source of truth for the new declarations.
After extracting, inspect the artifact files to understand their structure before comparing with nf-interpreter:
- **`.cpp` file** contains: `#include` directives; the `static const CLR_RT_MethodHandler method_lookup[]` array (one entry per managed method, in declaration order); and the `const CLR_RT_NativeAssemblyData g_CLR_AssemblyNative_...` struct (assembly name, checksum, `method_lookup` reference, native version tuple).
- **`.h` file** contains: enum/typedef declarations; per-class `struct Library_...` definitions with `FIELD__*` integer constants and `NANOCLR_NATIVE_DECLARE(...)` macros; and the `extern` data declaration.
- **`Find{LibraryName}.cmake`**: CMake module — **ignore this file**, DO NOT make any changes to it.

Your goal in step 5 is to make the nf-interpreter files match these artifact files, while preserving any existing local additions (see step 5 critical rules).
---
### 3. Prepare the nf-interpreter branch
Clone [nanoframework/nf-interpreter](https://github.com/nanoframework/nf-interpreter) if you haven't already. Choose the base branch as follows:
- If the **originating PR** (in the class library repo) targets **`main`** — branch off `main` in nf-interpreter. This corresponds to a stable/release build.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nanoframework/Home](https://github.com/nanoframework/Home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
