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
3. **Schema Validation:** Run `agentcore validate` before deploying configuration changes.
4. **Resource Removal:** Use `agentcore remove` to remove resources. Run `agentcore deploy` after removal to tear down
   deployed infrastructure.
5. **Invocation Input:** Validate runtime payloads and require text prompts to be strings. If a Strands app accepts a
   caller-supplied message history, normalize the history tail with `strip_trailing_tool_use()` before invocation.

## Directory Structure

```
myProject/
├── AGENTS.md               # This file — AI coding assistant context
├── agentcore/
│   ├── agentcore.json      # Main project config (AgentCoreProjectSpec)
│   ├── aws-targets.json    # Deployment targets (account + region)
│   ├── .env.local          # Secrets — API keys (gitignored)
│   └── cdk/                # AWS CDK project (@aws/agentcore-cdk L3 constructs)
├── app/                    # Agent application code
└── evaluators/             # Custom evaluator code (if any)
```

## Configuration Reference

- **AgentCoreProjectSpec**: Root config with runtimes, memories, knowledge bases, credentials, evaluators, online evals
  and insights, gateways, policy engines, config bundles, A/B tests, harness registrations, datasets, and payment
  managers
- **AgentEnvSpec**: Agent configuration (build type, entrypoint, code location, runtime version, network mode)
- **Memory**: Memory resource with strategies (SEMANTIC, SUMMARIZATION, USER_PREFERENCE, EPISODIC) and expiry
- **Credential**: API key or OAuth credential provider
- **AgentCoreGateway**: MCP gateway with targets (Lambda, MCP server, OpenAPI, Smithy, API Gateway, web-search,
  knowledge-base)
- **Evaluator**: LLM-as-a-Judge or code-based evaluator
- **OnlineEvalConfig**: Continuous evaluation pipeline bound to an agent
- **OnlineInsightsConfig** _[preview]_: Continuous failure-pattern analysis bound to an agent
- **KnowledgeBase**: Managed Bedrock Knowledge Base auto-wired to a gateway
- **Harness**: Declarative agent — runtime + tools + skills + memory + observability without writing agent code
- **PolicyEngine** + **Policy**: Cedar policy engine with form-based guardrails (Bedrock content filters, prompt-attack,
  sensitive-info) or raw Cedar policies
- **PaymentManager** + **PaymentConnector**: x402-protocol payment orchestration with provider credentials (CoinbaseCDP,
  StripePrivy)
- **ConfigBundle**: Versioned runtime configuration as a separately-deployable resource
- **Dataset**: Curated session dataset for batch evaluation and recommendation runs
- **RuntimeEndpoint**: Named endpoint (e.g. `PROMPT_V1`) targeting a specific runtime version

### Common Enum Values

- **BuildType**: `'CodeZip'` | `'Container'`
- **NetworkMode**: `'PUBLIC'` | `'VPC'`
- **RuntimeVersion**: `'PYTHON_3_10'` | `'PYTHON_3_11'` | `'PYTHON_3_12'` | `'PYTHON_3_13'` | `'PYTHON_3_14'` |
  `'NODE_18'` | `'NODE_20'` | `'NODE_22'`
- **MemoryStrategyType**: `'SEMANTIC'` | `'SUMMARIZATION'` | `'USER_PREFERENCE'` | `'EPISODIC'`
- **GatewayTargetType**: `'lambda'` | `'mcpServer'` | `'openApiSchema'` | `'smithyModel'` | `'apiGateway'` |
  `'lambdaFunctionArn'` | `'connector'` (web-search, bedrock-knowledge-bases)
- **ModelProvider**: `'Bedrock'` | `'Gemini'` | `'OpenAI'` | `'Anthropic'`
- **PaymentProvider**: `'CoinbaseCDP'` | `'StripePrivy'`
- **PolicyEnforcementMode**: `'ACTIVE'` | `'PASSIVE'`
- **GuardrailContentFilter**: `'VIOLENCE'` | `'HATE'` | `'SEXUAL'` | `'MISCONDUCT'` | `'INSULTS'`

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws/agentcore-cli](https://github.com/aws/agentcore-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
