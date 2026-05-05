---
trigger: always_on
description: This document provides foundational instructions, context, and conventions intended for AI assistants (e.g., Gemini CLI) operating within this repository.
---

# GenAI Factory

## Audience

This document provides foundational instructions, context, and conventions intended for AI assistants (e.g., Gemini CLI) operating within this repository.

Genai-factory is a collection of **end-to-end blueprints to deploy generative AI infrastructures** in GCP, following security best-practices.

- Embraces IaC best practices. Infrastructure is implemented in [Terraform](https://developer.hashicorp.com/terraform), leveraging [Terraform resources](https://registry.terraform.io/providers/hashicorp/google/latest/docs) and [Cloud Foundations Fabric modules](https://github.com/GoogleCloudPlatform/cloud-foundation-fabric).
- Follows the least-privilege principle: no default service accounts, primitive roles, minimal permissions.
- Compatible with [Cloud Foundation Fabric FAST](https://github.com/GoogleCloudPlatform/cloud-foundation-fabric/tree/master/fast) [project-factory](https://github.com/GoogleCloudPlatform/cloud-foundation-fabric/tree/master/modules/project-factory) and application templates.

## Factories Structure

The repository is composed of multiple factories.
Each factory is a folder. The only folders that are not factories are `.github`, `tests`, `tools`.

### Creating a New Factory

When scaffolding a new factory, adhere to these rules:

- **Naming:** The factory folder name should describe the use case (e.g., `cloud-run-nl2sql-bq`), the main product used underneath (e.g., `cloud-run-*`), or a mix of both.
- **Sub-folders:** Each factory must contain exactly two stage folders: `0-projects` and `1-apps`.
- **0-projects Structure:** The `0-projects` folder structure is identical across all factories, with the exception of the templates inside `data/projects`, and occasionally `templates/terraform.auto.tfvars.tpl` or `outputs.tf` depending on the values passed to `1-apps`.
- **1-apps Structure:** The `1-apps` stage must always include an architecture diagram named `diagram.png`. This diagram must be linked in the `README.md` of `1-apps`.
- **Applications:** The `1-apps` stage may contain an `apps` subfolder for application code (e.g., Python) deployed on top of the infrastructure. Commands to deploy these apps must be returned to the user via Terraform outputs in `1-apps`.
- **Documentation:** The factory root folder, the `0-projects` stage, and the `1-apps` stage must all have a `README.md` file. These READMEs follow a standard logical structure consistent across the repository; review existing factories (e.g., `cloud-run-single`) to derive and replicate this structure.

Each factory (folder) is split into two stages (which correspond to two Terraform modules and two sub-folders):

- **0-projects**: meant to be executed by infrastructure teams to prepare the project where the application team will deploy the resources and delegate permissions to the application team).
- **1-apps**: used by the application team to deploy the resources in the project created at the step before, with the identity created at the step before, that has been granted the roles needed at the step before.

### 0-projects

- It creates the project(s).
- It creates the service accounts used:
  - by the application team to run terraform in the next stage.
  - by the application itself, if needed (i.e., cloud run service account, cloud build service account).
- It creates the GCS buckets to store the Terraform state while applying the next stage.
- It enables APIs needed for the specific use case.
- It grants IAM roles to the service accounts created and to the service agents.

The stage contains one or more `.yaml` files within the `./data/projects` folder.
Each file contains the definition of a project to be created (and some other resources/roles that relate to it).

The stage asks users for a minimum set of values for some mandatory variables, declared in a `variables.tf` file.
For example, this includes (but is not limited to):
- The parent (either a folder or the organization id) under which the project(s) should be created.
- The region where regional resources will be deployed.

There are multiple ways users can prepare their projects:

- By running this stage, if users don't have their own project factory. The stage uses Fabric FAST application templates underneath and calls the [Cloud Foundation Fabric - Project Factory module](https://github.com/GoogleCloudPlatform/cloud-foundation-fabric/tree/master/modules/project-factory).
- If users run their own [FAST stage project factory](https://github.com/GoogleCloudPlatform/cloud-foundation-fabric/tree/master/fast/stages/2-project-factory), they can do so by copying the yaml files from the `./data/projects` folder.
- If users have a custom project creation process, they can leverage the information contained within the `.yaml` files in the `./data/projects` folder.

If users run this stage, it automatically creates two files in the `1-apps` stage:

- A **providers.tf** file that tells the stage (Terraform module):
  - Which service account to impersonate to run Terraform.
  - Which GCS bucket to use to store the Terraform state and which service account to impersonate while writing the Terraform state file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoogleCloudPlatform/genai-factory](https://github.com/GoogleCloudPlatform/genai-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
