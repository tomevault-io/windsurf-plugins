---
trigger: always_on
description: This documentation is for use by LLM agents tasked with drafting and editing documentation.
---

# Contributing to the Docs

This documentation is for use by LLM agents tasked with drafting and editing documentation.

Humans can refer to the [online contributing guide](https://docs.cscs.ch/contributing)

Agents should:

1. read the contributing guide in its raw form in the `docs/contributing/index.md` file — the spell checker section in particular is not duplicated here.
2. read the rest of this page.

To validate changes, run `./serve build` (requires [uv](https://docs.astral.sh/uv/getting-started/installation/) to be installed).
This will catch broken links and build errors before the CI pipeline does.

## Guidelines for agents

The documentation uses the Material for MkDocs framework.

The project configuration is in the `mkdocs.yml` file.

The docs are in the `docs` directory.

### What are we documenting?

These is the public facing documentation for the Swiss National Supercomputing Center (CSCS).
They are mostly technical documentation for all users that aim to guide users through their first steps getting an account and logging in, through to advanced usage of the different systems and services.

### Documentation structure

CSCS has a large HPE Cray EX system called Alps.
Alps is not deployed as a monolithic cluster, instead it is partitioned into clusters, and these clusters are assigned to use-case specific Platforms.
Most users come to CSCS through one of the platforms, each of which has its own clusters, storage configuration, and user software.
The docs are structured to provide an on-ramp through a platform page, which links out to more general purpose documentation about services/storage/software etc that apply to all platforms.

NOTE:

* the layout of pages in the table of contents does not match directory structure in `docs`.
    * This is partly due to history: a page might move to a new area in the ToC but not move inside the repo
    * The url is determined by the location in the `docs` path: if we don't move files urls point to our docs are less likely to break.
* we use autorefs. This allows us to move pages and files without having to update internal links to the pages.
* each section of the docs has an `index.md` file that introduces the section, with a table of cards on the index page that link to the child pages, and maybe a quickstart guide if appropriate.
    * see the docs in `docs/software/uenv` for a good example

Here is a quick overview of the top level ToC entries:

* `alps`:
    * general documentation about the system, with information about the hardware, network and storage
    * also has an overview page for the platforms, and overview pages for each platform that summarise the specific clusters used by that platform, details about storage and software provided to users of the platform.
* `connecting to alps`:
    * guides for all methods provided for users to connect to Alps.
    * MFA and SSH key management
    * using SSH, using VS Code, FirecREST, JupyterLab, etc
* `running jobs`:
    * guide to running batch and interactive jobs with the Slurm job scheduler
    * HyperQueue for high-throughput scheduling of many small tasks
    * tools for profiling job performance: job reports and GPU reports
* `environments`:
    * how to set up the shell environment after logging in to a cluster
    * uenv: the CSCS tool for delivering scientific software stacks on Alps
    * container engine: recommended for machine learning workflows and Python environments
* `building and installing software`:
    * programming environments (prgenv-gnu, prgenv-nvfortran, linalg, julia, CPE) and Alps Extended Images
    * guides for building software using uenv or Python
    * packaging and deployment: creating containers with podman, or building uenv with the build service
* `applications and frameworks`:
    * scientific applications: CP2K, GROMACS, LAMMPS, NAMD, Quantum ESPRESSO, VASP
    * machine learning: PyTorch, and tutorials for LLM inference, fine-tuning and pre-training
    * climate and weather: ICON, netcdf-tools
    * communication libraries: libfabric, Cray MPICH, MPICH, OpenMPI, NCCL, NVSHMEM
    * user applications: ESMF/CESM, ORCA, WRF
    * scientific visualisation: ParaView
    * commercial software: Matlab
* `debugging and performance analysis`:
    * parallel debugging tools: Linaro Forge DDT
    * performance analysis tools: NVIDIA Nsight, Linaro Forge MAP, Score-P/Scalasca
    * job report and GPU report tools
* `data management and storage`:
    * file systems available on Alps
    * data transfer: moving data into and out of CSCS and between CSCS systems
    * long term storage (LTS): preserving scientific data with persistent identifiers
    * object storage: Ceph-based public cloud object storage
* `services`:
    * CI/CD: integrating GitHub, GitLab and Bitbucket projects with Alps
    * developer portal: creating and managing API subscriptions
    * Kubernetes: platform for deploying and managing containerised applications
* `accounts and projects`:
    * how to get a CSCS account (requires an invitation from the PI of an active project)
    * the project and resources management tool at portal.cscs.ch
    * linking external institutional accounts to a CSCS account
* `guides`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eth-cscs/cscs-docs](https://github.com/eth-cscs/cscs-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
