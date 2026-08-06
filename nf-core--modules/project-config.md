---
trigger: always_on
description: This is the AI agent context file for the nf-core modules repository. All AI agents and coding assistants **MUST** read and strictly follow the rules contained in this document.
---

# nf-core/modules: agents

This is the AI agent context file for the nf-core modules repository. All AI agents and coding assistants **MUST** read and strictly follow the rules contained in this document.

## Natural language

All comments and documentation **MUST** be written in English with British spelling.

## Key terms

- **Module**: a single process that achieves a single, well defined task (e.g. aligning reads to a genome)
- **Subworkflow**: a sequence of chained modules that achieve a specific objective (e.g. FASTQ cleanup and quality check)
- **Component**: a module or subworkflow

## Repository structure

You are working in the nf-core modules repository. Repository structure:

```
.
├── modules
│   ├── environment-schema.json // schema for module's environment.yml
│   ├── meta-schema.json        // schema for module's meta.yml
│   └── nf-core                 // directory containing nf-core modules, described below
├── nf-test.config // configuration for nf-test
├── subworkflows
│   ├── nf-core          // directory containing nf-core subworkflows, described below
│   └── yaml-schema.json // schema for subworkflow meta.yaml
└── tests // assets for running tests
    └── config // Nextflow configurations, do not edit unless directly prompted
```

Certain rarely edited files were omitted in the treemap. You **MUST NOT** edit them unless the user explicitly asks for it.

### modules/nf-core structure

Module directories are under modules/nf-core, either directly or with one nesting level. The following rules apply:

- If a tool has only one command, it is included directly (e.g. modules/nf-core/iqtree)
- If a tool has subcommands, they are contained as subdirectories of the tool directory (e.g. modules/nf-core/bedops/convert2bed); if the tool without subcommands has a separate functionality, it is included as a subcommand (e.g. modules/nf-core/kraken2/kraken2)
- If a tool has multiple levels of subcommands, the subcommands are concatenated (e.g. `aws s3 ls` would be modules/nf-core/aws/s3ls)

### Module directory structure

Required structure:

```
modules/nf-core/{module}/
├── environment.yml       // list of Conda packages with pinned versions
├── main.nf               // Nextflow code of the module
├── meta.yml              // contains information about inputs, outputs and tools in the module
└── tests
    ├── main.nf.test      // nf-test unit tests for the module
    ├── main.nf.test.snap // snapshots for the tests (see note below)
    └── nextflow.config   // Nextflow configuration used for testing ONLY
```

You **MAY** manually change version strings in test snapshots after a version bump. Any other change **MUST ONLY** be introduced through `nf-test` or `nf-core` commands.

### Subworkflow directory structure

Each subworkflow has a single top-level directory, without nesting. Required structure:

```
subworkflows/nf-core/{subworkflow}/
├── main.nf               // Nextflow code of the subworkflow
├── meta.yml              // contains information about inputs, outputs and tools in the subworkflow
└── tests
    ├── main.nf.test      // nf-test unit tests for the subworkflow
    ├── main.nf.test.snap // snapshots for the tests (see note above)
    └── nextflow.config   // Nextflow configuration used for testing ONLY; multiple config files may exist in some cases
```

## Structure of a module

Each module contains several files. Each file has a well-defined structure that you **MUST** follow.

### main.nf

- main.nf **MUST** define **ONLY** one Nextflow `process`
- You **MUST NOT** add any directives other than `tag`, `label`, `conda`, and `container`
- `input` and `output` sections **MUST** follow the specification at https://nf-co.re/docs/specifications/components/modules/input-output-options
- You **MUST NOT** edit the `when` section if present
- You **SHOULD NOT** remove `args` and `prefix` definition unless clearly unnecessary
- You **MAY** edit the `prefix` definition to set a reasonable default prefix, respecting `task.ext.prefix`
- The `stub` section **MUST** emulate the output of the module as closely as possible; see https://nf-co.re/docs/specifications/components/modules/general#stubs
- Module code **MUST** pass all checks triggered by `nf-core modules lint` and `nextflow lint .`

### meta.yml

`meta.yml` **MUST** follow the specification at https://nf-co.re/docs/specifications/components/modules/documentation

### environment.yml

This file lists Conda channels and packages necessary to run the module with Conda and to build Seqera containers.

- You **SHOULD NOT** add channels unless strictly necessary.
- You **MUST NOT** add "defaults" to channels.
- Each dependency **MUST** specify the channel and version, but **NOT** the build number.
- If pip dependencies are used, you **MUST** also pin the version of pip.

### tests/main.nf.test

The test file **MUST** follow the specificaton at https://nf-co.re/docs/specifications/components/modules/testing.

- Test names **MUST** follow the rules in the "Test names" section of the specification. Stub test names **MUST** end with `- stub`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nf-core/modules](https://github.com/nf-core/modules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
