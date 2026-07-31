---
trigger: always_on
description: Required instructions for Terraform Variable Consistency including canonical definitions, requirements, and detailed instructions - Brought to you by microsoft/edge-ai
---


# Terraform Variable Consistency Manager Instructions

## Description Standards

You MUST follow these project standards:

- Single-line: You WILL use short, imperative style, capitalized first letter, no trailing period
- Multi-line: You WILL use heredoc with consistent delimiter and indentation preserved

## Canonical Variables (Single-line)

The table below is the authoritative source of canonical single-line descriptions. You MUST parse this table to build the canonical mapping.

<!-- <canonical-variables-table> -->

| Variable                                  | Description                                                                                                                                                                          |
|-------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| acr                                       | Azure Container Registry                                                                                                                                                             |
| acr_sku                                   | SKU for the Azure Container Registry. Valid values: Basic, Standard, Premium. Default is "Premium" (Premium is required for private endpoints).                                      |
| aio_dataflow_profile                      | The AIO dataflow profile                                                                                                                                                             |
| aio_identity                              | Azure IoT Operations managed identity for workspace access                                                                                                                           |
| aio_instance                              | The Azure IoT Operations instance                                                                                                                                                    |
| arc_onboarding_identity                   | The Principal ID for the identity that will be used for onboarding the cluster to Arc                                                                                                |
| arc_onboarding_principal_ids              | The Principal IDs for the identity or service principal that will be used for onboarding the cluster to Arc                                                                          |
| asset_endpoint_profiles                   | List of asset endpoint profiles to create. Otherwise, an empty list.                                                                                                                 |
| capacity_id                               | The capacity ID for the workspace                                                                                                                                                    |
| cluster_a_name                            | The name identifier for Cluster A                                                                                                                                                    |
| cluster_admin_oid                         | The Object ID that will be given cluster-admin permissions with the new cluster. (Otherwise, current logged in user Object ID if 'should_add_current_user_cluster_admin=true')       |
| cluster_admin_upn                         | The User Principal Name that will be given cluster-admin permissions with the new cluster. (Otherwise, current logged in user UPN if 'should_add_current_user_cluster_admin=true')   |
| cluster_b_name                            | The name identifier for Cluster B                                                                                                                                                    |
| cluster_server_host_machine_username      | Username used for the host machines that will be given kube-config settings on setup.                                                                                                |
| cluster_server_ip                         | The IP Address for the cluster server that the cluster nodes will use to connect.                                                                                                    |
| cluster_server_token                      | The token that will be given to the server for the cluster or used by the agent nodes to connect them to the cluster. (ex. [K3s token documentation](https://docs.k3s.io/cli/token)) |
| custom_location_id                        | The resource ID of the Custom Location                                                                                                                                               |
| custom_locations_oid                      | The object id of the Custom Locations Entra ID application for your tenant.                                                                                                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/edge-ai](https://github.com/microsoft/edge-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
