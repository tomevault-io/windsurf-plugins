---
trigger: always_on
description: These conventions apply to the documentation sources: the Markdown and reST files in `doc/` and the files they
---

# Documentation conventions

These conventions apply to the documentation sources: the Markdown and reST files in `doc/` and the files they
include with `.. include::` (for example `README.md`, `tools/Docker/README.md` and `recon_surf/README.md`). This file
itself is not part of the built documentation (`exclude_patterns` in `doc/conf.py`).

## Examples and explanations

Code blocks are either examples or explanations, and the two must look different.

- **Examples** are commands a reader can copy and run with at most small changes. They use a `bash` fence and the
  [example values](#example-values). Introduce them with "For example:". Sphinx adds a copy button. A value that no
  example can supply, such as the reader's GitHub user name (`<username>`), stays a placeholder.
- **Explanations** show the general form of a command. They use a `text` fence and [placeholders](#placeholders).
  Introduce them with "Usage:". Sphinx adds no copy button (`copybutton_selector` in `doc/conf.py`), and a `text`
  fence is not highlighted, so the two kinds also differ on GitHub.
- **Output** of commands, logs, file trees and file contents use a `text` fence as well, and commands in them have no
  prompt (`$`).

Explanations name scripts without a path, for example
`run_fastsurfer.sh --sd <subjects_dir> --sid <subject_id> --t1 <t1_path>`. Shell commands in examples use a
`bash` fence, not `sh`, `shell` or no language. Code in other languages (`python`, `toml`, `json`, `yaml`, ...) keeps
its language.

## Placeholders

Placeholders are `<snake_case>`: lowercase words joined by underscores, in angle brackets, for example `<subject_id>`,
`<subjects_dir>`, `<t1_path>`, `<template_id>` or `<tpid_1>`. Use them in explanations and in prose, and use the
same name for the same thing. A placeholder for a file or a folder stands for its full path (absolute or relative to
the current directory), and its suffix says which of the two it is:

- `_path` for a file: `<t1_path>`, `<freesurfer_license_path>`, `<sif_path>`,
- `_dir` for a folder: `<subjects_dir>`, `<subject_dir>`, `<data_dir>`,
- `_name` for a single file or folder name that is not a path: `<some_dir_name>`.

`<fastsurfer_home>` and `<freesurfer_home>` are named after the environment variables they stand for and keep that
name.

| Placeholder                                               | Meaning                                                                                              |
|-----------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| `<data_dir>`                                              | folder with the input images                                                                         |
| `<subjects_dir>`                                          | output folder, passed to `--sd`                                                                      |
| `<subject_id>`                                            | subject name, passed to `--sid`                                                                      |
| `<subject_dir>`                                           | output folder of one subject, `<subjects_dir>/<subject_id>`                                          |
| `<t1_path>`, `<t2_path>`                                  | input images, passed to `--t1` and `--t2`                                                            |
| `<freesurfer_license_path>`                               | FreeSurfer license file, passed to `--fs_license`                                                    |
| `<lesion_mask_path>`                                      | lesion mask, passed to `--lesion_mask`                                                               |
| `<subjects_list_path>`                                    | subjects list file, passed to `--subjects_list`                                                      |
| `<fastsurfer_home>`, `<freesurfer_home>`                  | FastSurfer checkout and FreeSurfer installation, where an explanation sets `FASTSURFER_HOME` or `FREESURFER_HOME` |
| `<fastsurfer_flags>`, `<docker_flags>`, `<singularity_flags>` | further flags of FastSurfer, `docker run` or `singularity exec`                                  |
| `<torch_device>`                                          | run-time device, passed to `--device`: `auto`, `cpu`, `cuda`, `cuda:1`, `mps`                        |
| `<version>`                                               | FastSurfer version without `v`, image tags are `deepmi/fastsurfer:<device>-v<version>`              |
| `<device>`                                                | device part of an image tag: `cu<cuda_version>`, `rocm`, `cpu`, ...                                  |
| `<sif_path>`                                              | Singularity image file                                                                               |
| `<image_tag>`                                             | full image reference, for example `deepmi/fastsurfer:<device>-v<version>`                            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Deep-MI/FastSurfer](https://github.com/Deep-MI/FastSurfer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
