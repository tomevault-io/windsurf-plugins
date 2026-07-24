---
trigger: always_on
description: > **Scope**: Repository-wide standards, conventions, and Microsoft style guide.
---

# AKS Repository - GitHub Copilot Instructions

> **Scope**: Repository-wide standards, conventions, and Microsoft style guide.  
> **Module-Specific**: See `AGENTS.md` files in subdirectories.  
> **File-Specific**: See `.github/instructions/*.instructions.md` for targeted patterns.

---

## Part 1: Repository Standards

### Repository Overview

This repository contains resources, examples, and documentation for the Azure Kubernetes Service (AKS) Engineering team:

- **Production Website**: <https://blog.aks.azure.com> (Docusaurus blog in `website/`)
- **Examples**: Real-world AKS scenarios, troubleshooting guides, and configurations
- **VHD Notes**: Node image release notes for AKS Ubuntu, Windows, Mariner, and Azure Linux
- **AI Conformance**: AKS service version compliance profiles
- **Community**: Open-source collaboration with AKS users and contributors

#### Repository Structure

```text
AKS/
├── .github/
│   ├── copilot-instructions.md          # This file (repo-wide standards)
│   └── instructions/                    # File-specific patterns
│       └── website.blog.instructions.md # Blog post guidelines
├── website/                             # Docusaurus blog site
│   ├── AGENTS.md                       # Website module guide
│   ├── blog/                           # Blog posts
│   ├── src/                            # React components
│   └── package.json
├── examples/                            # AKS configuration examples
│   ├── fleet/                          # Azure Kubernetes Fleet Manager
│   ├── istio-based-service-mesh/       # Service mesh examples
│   ├── kube-prometheus/                # Monitoring setup
│   └── vnet/                           # Networking examples
├── vhd-notes/                          # Node image release notes
│   ├── aks-ubuntu/
│   ├── AKSMariner/
│   ├── AKSWindows/
│   └── AzureLinux/
├── README.md
└── LICENSE.MD
```

---

### Code Style and File Naming

#### Code Style
- **Markdown**: Follow CommonMark spec
- **YAML**: 2-space indentation, no tabs
- **Shell scripts**: Use shellcheck-compliant bash with `set -euo pipefail`
- **TypeScript**: Follow TypeScript ESLint recommended rules
- **React**: Functional components with hooks (no class components)

#### File Naming
| Type | Convention | Example |
|:---|:---|:---|
| Markdown | `kebab-case.md` | `getting-started.md` |
| TypeScript/React components | `PascalCase.tsx` | `BlogPost.tsx` |
| TypeScript utilities | `camelCase.ts` | `analytics.ts` |
| CSS | `kebab-case.css` or `ComponentName.module.css` | `blog-post.module.css` |
| YAML | `kebab-case.yaml` or `.yml` | `deployment.yaml` |
| Shell scripts | `kebab-case.sh` | `remediate.sh` |
| VHD notes | `YYYYMMDD.VV.V.txt` | `202401.03.0.txt` |

---

### Build and Development

#### Website (`website/`)

See `website/AGENTS.md` for detailed build instructions and troubleshooting.

```bash
cd website
npm install        # Install dependencies
npm start          # Dev server with hot reload
npm run build      # Production build (must succeed)
npm run typecheck  # TypeScript validation
```

#### Examples

Most examples are standalone YAML files or scripts:

```bash
# Validate Kubernetes YAML
kubectl apply --dry-run=client -f examples/fleet/kuard/deployment.yaml

# Run example script
bash examples/kernel-1095-issue/remediate.sh
```

**Shell script template**:
```bash
#!/bin/bash
set -euo pipefail  # Exit on error, undefined vars, pipe failures

# Check prerequisites
if ! command -v kubectl &> /dev/null; then
    echo "Error: kubectl not found" >&2
    exit 1
fi

# Use environment variables with defaults
CLUSTER_NAME="${CLUSTER_NAME:-my-aks-cluster}"
RESOURCE_GROUP="${RESOURCE_GROUP:-my-resource-group}"
```

#### VHD Notes

Text files documenting node image changes. No build process required.

1. Locate directory: `vhd-notes/{image-type}/`
2. Create/edit file: `YYYYMMDD.VV.V.txt`
3. List changes chronologically with package versions
4. Commit with: `docs(vhd-notes): update {image-type} {version}`

---

### Git Workflow

#### Branch Strategy
- **master**: Main branch, stable code (protected)
- **Feature branches**: Short-lived, descriptive names (`add-fleet-examples`, `fix-blog-typo`)

#### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```text
<type>(<scope>): <subject>
```

| Type | Use for |
|:---|:---|
| `feat` | New feature or post |
| `fix` | Bug fix (broken links, typos, build errors) |
| `docs` | Documentation changes (examples, README, VHD notes) |
| `style` | Formatting (no functional change) |
| `refactor` | Code restructuring |
| `chore` | Maintenance tasks |

**Scope examples**: `website`, `examples`, `vhd-notes`, `blog`, `github-actions`

**Examples**:
```text
feat(website): add AKS MCP server blog post
docs(examples): add Fleet Manager multi-cluster setup guide
fix(website): correct broken link in webinar metadata
```

#### Pull Requests
- **Title**: Follows conventional commit format
- **Description**: What changed and why, link to related issues
- **Testing**: Describe validation (e.g., "npm run build passed")
- **Screenshots**: Include for UI changes

---

### Azure/Kubernetes Best Practices

#### YAML Manifests

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
  labels:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/AKS](https://github.com/Azure/AKS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
