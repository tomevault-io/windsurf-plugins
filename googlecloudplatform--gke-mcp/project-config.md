---
trigger: always_on
description: This document provides instructions for an AI agent on how to use the available tools to manage Google Kubernetes Engine (GKE) resources.
---

# GKE MCP Extension for Gemini CLI

This document provides instructions for an AI agent on how to use the available tools to manage Google Kubernetes Engine (GKE) resources.

## Guiding Principles

- **Prefer Native Tools:** Always prefer to use the tools provided by this extension (e.g., `list_clusters`, `get_cluster`) instead of shelling out to `gcloud` or `kubectl` for the same functionality. This ensures better-structured data and more reliable execution.
- **Clarify Ambiguity:** Do not guess or assume values for required parameters like cluster names or locations. If the user's request is ambiguous, ask clarifying questions to confirm the exact resource they intend to interact with.
- **Use Defaults:** If a `project_id` is not specified by the user, you can use the default value configured in the environment.
- **Verify Commands:** Before providing any command to the user， verify it is correct and appropriate for the user's request. You can search online or refer to [gcloud documentation](https://cloud.google.com/sdk/gcloud).
- **Verbosity:** In the end of response add related links which were used to form a response.
- **Table Investigation:** If in document search Table appears - read it in JSON format to correctly interpret provided data.

## Authentication

Some MCP tools required [Application Default Credentials](https://cloud.google.com/docs/authentication/application-default-credentials). If they return an "Unauthenticated" error, tell the user to run `gcloud auth application-default login` and try again. This is an interactive command and must be run manually outside the AI.

## Manifest Generation Agent

The `generate_manifest` tool is a specialized agent for generating, analyzing, and optimizing Kubernetes YAML manifests for GKE. It uses advanced reasoning to translate natural language requests into secure, best-practice-adhering manifests.

- **Intent:** It handles requests to create or modify Kubernetes resources (Deployments, Services, ConfigMaps, etc.).
- **Best Practices:** It automatically applies health probes, high availability settings, and security contexts.
- **Inference Workloads:** For AI/LLM inference workloads, it is configured to prioritize using the `giq_generate_manifest` function tool to generate optimized serving manifests.
- **Usage:** Provide a detailed prompt describing the desired workload, and the agent will return only the raw YAML manifest.

## GKE Logs

- When searching for GKE logs, always use the `query_logs` tool to fetch them. It's also **strongly** recommended to call the `get_log_schema` tool before building or running a query to obtain information about the log schema, as well as sample queries. This information is useful when building Cloud Logging LQL queries.

- When using time ranges, make sure you check the current time and date if the range is relative to the current time or date.

- When searching log entries for a single cluster, **always** include an LQL filter clause for the project ID, cluster name, and cluster location. Note that filtering by project ID is needed even if the project ID is specified in the `query_logs` request, as depending on the log ingention configuration, multiple logs with same name and location can be ingested into the same project.

- If you need help understanding LQL syntax, consider fetching [Logging query language](https://cloud.google.com/logging/docs/view/logging-query-language) to learn more about it.

## GKE Monitoring

When users ask a question about the Monitoring or monitored resource types, the following instructions could be applied:

- Please use the tool `list_monitored_resource_descriptors` to get all monitored resource descriptors
- After getting all the monitored resource, if the user ask for GKE specific ones, please filter the output and only include the GKE related ones
  \*\* Full GKE related monitored resources are the one contains `gke` or `k8s` or `container.googleapis.com`

## GKE Cost

GKE costs are available from **[GCP Billing Detailed BigQuery Export](https://cloud.google.com/billing/docs/how-to/export-data-bigquery#setup):**. The user will have to provide the full path to their BigQuery table, which inludes their BigQuery dataset name and the table name which contains their Billing Account ID.

These costs can be queried in two ways:

- **BigQuery CLI:** Using the `bq` command-line tool is the preferred way to view the costs, since that can be run locally. If the `bq` CLI is available prefer to use that and offer to run queries for the user.
- **BigQuery Studio:** If the `bq` CLI is not available, user's can run the query themselves in [BigQuery Studio](https://console.cloud.google.com/bigquery).

Some parameters that may be required based on the query:

- Time frame: Assume the last 30 days unless indicated otherwise
- GCP project ID
- GKE cluster location
- GKE cluster name
- Kubernetes namespace (requires [GKE Cost Allocation enabled on the cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/cost-allocations))
- Kubernetes workload type (requires [GKE Cost Allocation enabled on the cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/cost-allocations))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoogleCloudPlatform/gke-mcp](https://github.com/GoogleCloudPlatform/gke-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
