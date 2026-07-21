---
trigger: always_on
description: This project contains configuration and infrastructure for an Amazon Bedrock AgentCore application.
---

# AgentCore Project

This project contains configuration and infrastructure for an Amazon Bedrock AgentCore application.

The `agentcore/` directory is a declarative model of the project. The `agentcore/cdk/` subdirectory uses the
`@aws/agentcore-cdk` L3 constructs to deploy the configuration to AWS.

## Mental Model

The project uses a **flat resource model**. Agents, memories, credentials, gateways, evaluators, and policies are
independent top-level arrays in `agentcore.json`. There is no binding between resources in the schema — each resource is
provisioned independently. Agents discover memories and credentials at runtime via environment variables or SDK calls.
Tags defined in `agentcore.json` flow through to deployed CloudFormation resources.

## Critical Invariants

1. **Schema-First Authority:** The `.json` files are the source of truth. Do not modify agent behavior by editing
   generated CDK code in `cdk/`.
2. **Resource Identity:** The `name` field determines the CloudFormation Logical ID.
   - **Renaming** a resource will **destroy and recreate** it.
   - **Modifying** other fields will update the resource **in-place**.
3. **Schema Validation:** If your JSON conforms to the types in `.llm-context/`, it will deploy successfully. Run
   `agentcore validate` to check.
4. **Resource Removal:** Use `agentcore remove` to remove resources. Run `agentcore deploy` after removal to tear down
   deployed infrastructure.

## Directory Structure

```
myProject/
├── AGENTS.md               # This file — AI coding assistant context
├── agentcore/
│   ├── agentcore.json      # Main project config (AgentCoreProjectSpec)
│   ├── aws-targets.json    # Deployment targets (account + region)
│   ├── .env.local          # Secrets — API keys (gitignored)
│   ├── .llm-context/       # TypeScript type definitions for AI assistants
│   │   ├── README.md       # Guide to using schema files
│   │   ├── agentcore.ts    # AgentCoreProjectSpec types
│   │   ├── aws-targets.ts  # AWS deployment target types
│   │   └── mcp.ts          # Gateway and MCP tool types
│   └── cdk/                # AWS CDK project (@aws/agentcore-cdk L3 constructs)
├── app/                    # Agent application code
└── evaluators/             # Custom evaluator code (if any)
```

## Schema Reference

The `agentcore/.llm-context/` directory contains TypeScript type definitions optimized for AI coding assistants. Each
file maps to a JSON config file and includes validation constraints as comments (`@regex`, `@min`, `@max`).

| JSON Config | Schema File | Root Type |
| --- | --- | --- |
| `agentcore/agentcore.json` | `agentcore/.llm-context/agentcore.ts` | `AgentCoreProjectSpec` |
| `agentcore/agentcore.json` (gateways) | `agentcore/.llm-context/mcp.ts` | `AgentCoreMcpSpec` |
| `agentcore/aws-targets.json` | `agentcore/.llm-context/aws-targets.ts` | `AwsDeploymentTarget[]` |

### Key Types

- **AgentCoreProjectSpec**: Root config with `runtimes`, `memories`, `credentials`, `agentCoreGateways`, `evaluators`, `onlineEvalConfigs`, `policyEngines` arrays
- **AgentEnvSpec**: Agent configuration (build type, entrypoint, code location, runtime version, network mode)
- **Memory**: Memory resource with strategies (SEMANTIC, SUMMARIZATION, USER_PREFERENCE, EPISODIC) and expiry
- **Credential**: API key or OAuth credential provider
- **AgentCoreGateway**: MCP gateway with targets (Lambda, MCP server, OpenAPI, Smithy, API Gateway)
- **Evaluator**: LLM-as-a-Judge or code-based evaluator
- **OnlineEvalConfig**: Continuous evaluation pipeline bound to an agent

### Common Enum Values

- **BuildType**: `'CodeZip'` | `'Container'`
- **NetworkMode**: `'PUBLIC'` | `'VPC'`
- **RuntimeVersion**: `'PYTHON_3_10'` | `'PYTHON_3_11'` | `'PYTHON_3_12'` | `'PYTHON_3_13'` | `'PYTHON_3_14'` | `'NODE_18'` | `'NODE_20'` | `'NODE_22'`
- **MemoryStrategyType**: `'SEMANTIC'` | `'SUMMARIZATION'` | `'USER_PREFERENCE'` | `'EPISODIC'`
- **GatewayTargetType**: `'lambda'` | `'mcpServer'` | `'openApiSchema'` | `'smithyModel'` | `'apiGateway'` | `'lambdaFunctionArn'`
- **ModelProvider**: `'Bedrock'` | `'Gemini'` | `'OpenAI'` | `'Anthropic'`

### Build Types

- **CodeZip**: Python source packaged as a zip and deployed directly to AgentCore Runtime.
- **Container**: Docker image built in CodeBuild (ARM64), pushed to a per-agent ECR repository. Requires a `Dockerfile`
  in the agent's `codeLocation` directory. For local development (`agentcore dev`), the container is built and run
  locally with volume-mounted hot-reload.

### Supported Frameworks (for template agents)

- **Strands** — Bedrock, Anthropic, OpenAI, Gemini
- **LangChain/LangGraph** — Bedrock, Anthropic, OpenAI, Gemini
- **GoogleADK** — Gemini
- **OpenAI Agents** — OpenAI
- **Autogen** — Bedrock, Anthropic, OpenAI, Gemini

### Protocols

- **HTTP** — Standard HTTP agent endpoint
- **MCP** — Model Context Protocol server
- **A2A** — Agent-to-Agent protocol (Google A2A)

## Deployment

Deployments are orchestrated through the CLI:

```bash
agentcore deploy    # Synthesizes CDK and deploys to AWS
agentcore status    # Shows deployment status
```

Alternatively, deploy directly via CDK:

```bash
cd agentcore/cdk
npm install
npx cdk synth
npx cdk deploy
```

## Editing Schemas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awslabs/agentcore-samples](https://github.com/awslabs/agentcore-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
