---
trigger: always_on
description: Guide for AI agents working in this repository.
---

# AGENTS.md

Guide for AI agents working in this repository.

## Project Overview

**kubernetes-mcp** is a production-grade Model Context Protocol (MCP) server
that exposes a Kubernetes cluster (or several) to an LLM through a stable,
authorization-aware tool surface. It speaks both **stdio** (for desktop
clients like Claude Desktop) and **HTTP Streamable** (for remote agents),
implements **OAuth 2.1** discovery (RFC 8414 / RFC 9728), and ships its own
fine-grained RBAC layer evaluated with CEL on top of the cluster's native RBAC.

- **Language**: Go 1.25+
- **Module**: `kubernetes-mcp`
- **Primary dependency**: [mcp-go](https://github.com/mark3labs/mcp-go)
- **Tools**: 25 (read / modify / scale / rollout / logs / exec / events /
  cluster info / context / RBAC / metrics / diff)

## Essential Commands

```bash
make build               # Build binary -> bin/kubernetes-mcp-{os}-{arch}
make run                 # Run HTTP server with docs/config-http.yaml
make fmt                 # go fmt
make vet                 # go vet
make lint                # golangci-lint (auto-installs)
make lint-fix            # golangci-lint --fix
make docker-build IMG=…  # Build container image
make docker-push  IMG=…  # Push container image
make package             # Tarball the binary
make help                # All targets

# Tests
make test                # Unit tests (no cluster needed)
make kind-up             # Create the local Kind cluster used by e2e (idempotent)
make kind-down           # Delete it
make test-e2e            # Run the e2e suite (build tag 'e2e') against Kind
make test-e2e-clean      # down + up + tests + down (CI-style)
```

## Project Structure

```
kubernetes-mcp/
├── cmd/main.go                       # Entrypoint: wiring of middlewares,
│                                     #   ClientManager, Manager, transport.
├── api/config_types.go               # YAML configuration schema. Edit here
│                                     #   when adding new top-level config knobs.
├── internal/
│   ├── globals/globals.go            # ApplicationContext (config + logger)
│   ├── config/config.go              # YAML parsing with $VAR expansion
│   ├── handlers/                     # OAuth well-known endpoints (HTTP)
│   ├── middlewares/                  # ToolMiddleware / HttpMiddleware
│   │   ├── auth.go                   #   shared auth payload header (X-Auth-Payload)
│   │   ├── jwt_validation.go         #   JWT validation (JWKS + CEL allow_conditions)
│   │   ├── apikey_validation.go      #   Static API keys with attached payloads
│   │   ├── logging.go                #   AccessLogsMiddleware
│   │   ├── interfaces.go             #   Interfaces both kinds implement
│   │   └── utils.go / noop.go
│   ├── kubernetes/client.go          # ClientManager: per-context Client (Clientset
│   │                                 #   + DynamicClient + DiscoveryClient + RESTMapper).
│   │                                 #   Supports explicit kubeconfig, $KUBECONFIG,
│   │                                 #   ~/.kube/config and in-cluster, with inotify
│   │                                 #   reload and periodic discovery refresh.
│   ├── authorization/                # CEL-based RBAC for the MCP itself
│   │   ├── evaluator.go              #   Evaluator + AuthzRequest + ResourceInfo
│   │   ├── evaluator_test.go         #   Unit tests
│   │   ├── policy_safeops_test.go    #   "safe-ops" policy regression tests
│   │   └── integration_test.go       #   Cluster-discovery driven RBAC sanity
│   ├── k8stools/                     # The 25 MCP tools live here
│   │   ├── manager.go                #   Manager + RegisterAll()
│   │   ├── helpers.go                #   gvrFromArgs, validateGVR, RESTMapper
│   │   │                             #   resolvers, error/result helpers
│   │   ├── tools_read.go             #   get_resource, list_resources, describe_resource
│   │   ├── tools_modify.go           #   apply_manifest, patch_resource,
│   │   │                             #     delete_resource, delete_resources
│   │   ├── tools_scale_rollout.go    #   scale_resource, get_rollout_status,
│   │   │                             #     restart_rollout, undo_rollout
│   │   ├── tools_logs_exec.go        #   get_logs, exec_command, list_events
│   │   ├── tools_cluster.go          #   list_api_resources, list_api_versions,
│   │   │                             #     get_cluster_info, list_namespaces
│   │   ├── tools_context.go          #   get_current_context, list_contexts,
│   │   │                             #     switch_context
│   │   ├── tools_rbac_metrics.go     #   check_permission, get_pod_metrics,
│   │   │                             #     get_node_metrics
│   │   ├── tools_diff.go             #   diff_manifest
│   │   └── e2e_*_test.go             #   E2E tests (build tag 'e2e')
│   └── yqutil/evaluator.go           # yq expression engine used by yq_expressions
├── docs/
│   ├── config-http.yaml              # HTTP transport example
│   └── config-stdio.yaml             # Stdio transport example
├── chart/values.yaml                 # bjw-s app-template values for Helm install
├── .agents/                          # Internal docs (this directory)
│   ├── AGENTS.md
│   ├── CONFIG_DESIGN.md
│   ├── RESOURCE_FILTERING_DESIGN.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [achetronic/kubernetes-mcp](https://github.com/achetronic/kubernetes-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
