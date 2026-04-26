---
trigger: always_on
description: This document provides context and guidelines for AI coding assistants working on the Karpenter Optimizer project.
---

# AGENTS.md - AI Assistant Guidelines

This document provides context and guidelines for AI coding assistants working on the Karpenter Optimizer project.

## Project Overview

**Karpenter Optimizer** is a tool for optimizing Karpenter NodePool configurations based on actual cluster usage data. It analyzes Kubernetes workloads and node capacity to provide cost-optimized instance type recommendations.

### Key Purpose
- Analyze actual node capacity and usage (CPU/Memory) from Kubernetes cluster
- Generate cost-optimized NodePool recommendations using AWS instance types
- Provide AI-enhanced explanations of recommendations and their benefits
- Support both spot and on-demand instance type optimization

## Architecture

### Backend (Go)
- **Location**: `cmd/api/` (main entry), `internal/` (core logic)
- **Framework**: Gin (HTTP router)
- **Key Components**:
  - `internal/api/server.go` - REST API endpoints
  - `internal/recommender/recommender.go` - Legacy recommendation logic (cluster-summary based)
  - `internal/recommender/node_pool_recommender.go` - **PRIMARY** recommendation engine (NodePool-based)
  - `internal/kubernetes/client.go` - Kubernetes API client (nodes, pods, NodePools)
  - `internal/awspricing/client.go` - AWS Pricing API client for instance pricing
  - `internal/ollama/client.go` - Ollama LLM client for AI explanations

### Frontend (React)
- **Location**: `frontend/src/`
- **Key Components**:
  - `App.js` - Main application component
  - `components/GlobalClusterSummary.js` - Cluster overview and recommendation trigger
  - `components/NodePoolCard.js` - Displays individual NodePool recommendations
  - `components/NodeUsageView.js` - Real-time node resource usage visualization
  - `components/DisruptionTracker.js` - Node disruption tracking (on-demand)

## Key Data Structures

### NodePoolCapacityRecommendation (Primary Format)
```go
type NodePoolCapacityRecommendation struct {
    NodePoolName             string   `json:"nodePoolName"`
    CurrentNodes             int      `json:"currentNodes"`
    CurrentInstanceTypes     []string `json:"currentInstanceTypes"` // Format: "m6g.2xlarge (27)"
    CurrentCPUUsed           float64  `json:"currentCPUUsed"`
    CurrentCPUCapacity       float64  `json:"currentCPUCapacity"`
    CurrentMemoryUsed        float64  `json:"currentMemoryUsed"`
    CurrentMemoryCapacity    float64  `json:"currentMemoryCapacity"`
    CurrentCost              float64  `json:"currentCost"`
    RecommendedNodes         int      `json:"recommendedNodes"`
    RecommendedInstanceTypes []string `json:"recommendedInstanceTypes"`
    RecommendedTotalCPU      float64  `json:"recommendedTotalCPU"`
    RecommendedTotalMemory   float64  `json:"recommendedTotalMemory"`
    RecommendedCost          float64  `json:"recommendedCost"`
    CostSavings              float64  `json:"costSavings"`
    CostSavingsPercent       float64  `json:"costSavingsPercent"`
    Reasoning                string   `json:"reasoning"` // AI-generated explanation
    Architecture             string   `json:"architecture"` // arm64 or amd64
    CapacityType             string   `json:"capacityType"` // spot or on-demand
}
```

### NodeInfo (from Kubernetes client)
```go
type NodeInfo struct {
    Name          string
    InstanceType  string
    Architecture  string
    CapacityType  string // "spot" or "on-demand"
    NodePool      string
    CPUUsage      *ResourceUsage
    MemoryUsage   *ResourceUsage
    PodCount      int
    // ...
}
```

## API Endpoints

### Primary Recommendation Endpoints
- `GET /api/v1/recommendations` - NodePool-based recommendations (uses `GenerateRecommendationsFromNodePools`)
- `GET /api/v1/nodepools/recommendations` - Same as above, returns recommendations array
- `GET /api/v1/recommendations/cluster-summary` - **Enhanced with Ollama explanations** (uses nodepools endpoint + Ollama)
- `GET /api/v1/recommendations/cluster-summary/stream` - SSE version with progress updates

### Data Endpoints
- `GET /api/v1/nodepools` - List all NodePools with actual node data
- `GET /api/v1/nodes` - Get all nodes with usage data
- `GET /api/v1/cluster/summary` - Cluster-wide statistics

### Karpenter Log Analysis Endpoints
- `POST /api/v1/karpenter/logs/analyze` - Analyze Karpenter error logs and provide explanations

## Important Patterns and Conventions

### 1. Recommendation Generation Flow
```
1. Fetch NodePools with actual nodes (ListNodePools)
2. For each NodePool:
   a. Calculate current capacity (CPU/Memory allocatable from nodes)
   b. Calculate current cost (based on actual instance types and capacity types)
   c. Find optimal instance types (findOptimalInstanceTypesWithCapacityType)
   d. Try both spot and on-demand to find best cost
   e. Only recommend if cost savings > 0
3. Enhance with Ollama explanations (if Ollama available)
```

### 2. Capacity Type Handling
- **Always analyze actual node capacity types** (spot vs on-demand)
- **Try both spot and on-demand** when finding optimal instance types
- **Prefer spot** if nodes are already spot, or if converting on-demand to spot saves money
- **Default to on-demand** if capacity type is unknown

### 3. Cost Calculation
- **Current Cost**: Sum of individual node costs based on:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaskol10/karpenter-optimizer](https://github.com/kaskol10/karpenter-optimizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
