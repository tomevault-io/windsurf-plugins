---
trigger: always_on
description: This project is a comprehensive performance benchmark comparing AWS Lambda ARM (Graviton2/3) vs x86 architectures across modern runtimes (Python 3.14/3.13/3.12/3.11, Node.js 22/20, Rust). It extends the [2023 AWS blog post](https://aws.amazon.com/blogs/apn/comparing-aws-lambda-arm-vs-x86-performance-cost-and-analysis-2/) with current runtime versions and best practices.
---

# AWS Lambda ARM vs x86 Performance Benchmark

## Project Overview

This project is a comprehensive performance benchmark comparing AWS Lambda ARM (Graviton2/3) vs x86 architectures across modern runtimes (Python 3.14/3.13/3.12/3.11, Node.js 22/20, Rust). It extends the [2023 AWS blog post](https://aws.amazon.com/blogs/apn/comparing-aws-lambda-arm-vs-x86-performance-cost-and-analysis-2/) with current runtime versions and best practices.

---

## ⚠️ CRITICAL: Git Workflow Rules

**NEVER commit or push changes without explicit user approval.**

- Make code changes when requested
- Show what changed and explain the modifications
- **WAIT for explicit user approval** before running `git add`, `git commit`, or `git push`
- Let the user review and commit changes themselves
- This applies to ALL changes, including bug fixes, optimizations, and documentation updates

**Rust Support:** AWS officially announced Rust Lambda support on November 14, 2025. This benchmark uses the [`cargo-lambda-cdk`](https://github.com/cargo-lambda/cargo-lambda-cdk) construct library for deployment. See [AWS blog post](https://aws.amazon.com/blogs/compute/building-serverless-applications-with-rust-on-aws-lambda/) for details.

**Key Innovation:** Forced cold start technique (memory toggling) dramatically reduces test execution time compared to waiting for natural cold starts.

**Infrastructure:**
- 42 Lambda functions (7 runtimes × 2 architectures × 3 workloads)
  - Python: 3.14, 3.13, 3.12, 3.11
  - Node.js: 22, 20
  - Rust: provided.al2023 runtime with cargo-lambda
- DynamoDB tables for results storage and test data (with TTL)
- CloudWatch Logs with short retention for cost optimization


---

## Quick Start Commands

### Deploy Infrastructure (idempotent)
```bash
# Full deployment from root
npm run deploy

# Or deploy from CDK directory
cd cdk
npm ci && npm run build
cdk bootstrap  # First time only
cdk deploy LambdaBenchmarkStack --require-approval never
```

### Run Benchmarks

**IMPORTANT:** Always ask user for confirmation before running benchmarks. Show:
- Mode (test vs balanced vs production)
- Configuration details (cold/warm starts, memory sizes)
- Estimated duration and cost

**Test Mode** (quick validation - 2 cold + 2 warm, ~10 min):
```bash
uv run python scripts/benchmark_orchestrator.py --test
```

**Balanced Mode** (publication-quality - 50 cold + 200 warm, ~6-8 hours, ~$2-4):
```bash
uv run python scripts/benchmark_orchestrator.py --balanced
```

**Production Mode** (maximum rigor - 100 cold + 500 warm, ~18-24 hours, ~$5-10):
```bash
uv run python scripts/benchmark_orchestrator.py --production
```

Configuration details in `scripts/benchmark_orchestrator.py` (TEST_CONFIG, BALANCED_CONFIG, PRODUCTION_CONFIG) and `scripts/benchmark_utils.py` (MEMORY_CONFIGS).

**EC2 Execution (Recommended for Balanced/Production):**
For Balanced (~1 hour) and Production (several hours) modes, AWS SSO tokens may expire mid-test. Use EC2 instance with IAM instance profile to avoid credential expiration:

```bash
# Launch EC2 that auto-runs benchmark and terminates when complete
uv run python scripts/run_benchmark_on_ec2.py --mode balanced
uv run python scripts/run_benchmark_on_ec2.py --mode production
uv run python scripts/run_benchmark_on_ec2.py --mode production --s3-bucket my-results
```

Benefits: No SSO token expiration, immune to laptop sleep/network issues, auto-terminates.

### Analyze Results
```bash
uv run python scripts/analyze_results.py <test-run-id>
uv run python scripts/analyze_results.py <test-run-id> --runtime python3.13
uv run python scripts/analyze_results.py <test-run-id> --workload cpu-intensive
```

---

## Critical Invariants

1. **Zero-overhead metrics collection**: CPU/Memory workloads MUST NOT import AWS SDK. All metrics extracted from CloudWatch REPORT line using `LogType='Tail'`.
2. **Forced cold starts**: Achieved by toggling memory size via `UpdateFunctionConfiguration`, NOT by waiting for natural cold starts.
3. **Dynamic memory configuration**: Memory settings changed at runtime via `UpdateFunctionConfiguration` API, NOT by deploying separate functions.
4. **Benchmark purity**: Only Light workload (DynamoDB) imports AWS SDK; CPU/Memory workloads are pure computation.
5. **Function discovery**: Orchestrator discovers functions via `list_stack_resources()`, not CloudFormation outputs.

**Configuration Authority:**
- Lambda configs: `cdk/lib/config/lambda-config.ts` (runtimes, architectures, workloads)
- Memory configs: `scripts/benchmark_utils.py` (`MEMORY_CONFIGS` dict)
- Memory-intensive workload: Fixed 100 MB array (constant across all Lambda memory sizes)

---

## Project Structure

**Infrastructure (`cdk/`):**
- `lib/config/lambda-config.ts` - 36 function configs (6 runtimes × 2 architectures × 3 workloads)
- `lib/constructs/` - Lambda, DynamoDB table constructs
- `lib/cdk-stack.ts` - Main CDK stack

**Lambda Handlers (`lambdas/`):**
- `python/` - Python 3.13/3.12/3.11 handlers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cebert/aws-lambda-performance-benchmarks](https://github.com/cebert/aws-lambda-performance-benchmarks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
