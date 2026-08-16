---
trigger: always_on
description: This is the canonical repository guide for coding agents. Tool-specific files
---

# Agent instructions

This is the canonical repository guide for coding agents. Tool-specific files
should point here instead of restating these rules.

## Repository map

| Path                                  | Purpose                                                                                                              |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `catalog.json`                        | Machine-readable index of every runnable recipe, including paths, entrypoints, integrations, and supported commands. |
| `examples/<task>/`                    | Self-contained TypeScript programs that call the Notion API and run locally with Node.js.                            |
| `workers/<integration>-<capability>/` | Self-contained Notion Worker syncs, agent tools, and webhooks.                                                       |
| `skills/`                             | Reusable instructions and evaluations for AI-assisted Notion workflows.                                              |
| `docs/`                               | Longer developer guides that are not standalone runnable projects.                                                   |
| `scripts/`                            | Repository-wide installation, validation, and maintenance commands.                                                  |

Do not infer a recipe's category from a path segment that no longer exists.
`examples/` and `workers/` are both flat collections; `kind` in `catalog.json`
is the authoritative classification.

## Find the right recipe

1. Translate the request into an outcome, integration, and recipe kind.
2. Search `catalog.json` before scanning the tree. For example:

   ```sh
   jq '.recipes[] | select(.integrations | index("linear"))' catalog.json
   jq '.recipes[] | select(.kind == "worker-tool") | {id, summary, path}' catalog.json
   ```

3. Read the selected recipe's README, `package.json`, and listed entrypoints.
4. Check nearby recipes only when they demonstrate a pattern the selected
   project does not cover.

Prefer the narrowest working recipe. For example, adapt `linear-sync` for a new
Linear field; do not start from a generic Worker if the integration already has
an implemented sync.

## Run a recipe

Recipes are independent projects. Run commands from the recipe directory and
install dependencies there. Never assume every project uses `npm start` or has
the same environment variables.

1. Use the commands declared for the recipe in `catalog.json`.
2. Read its README before creating `.env` or configuring an external service.
3. Copy the provided environment template when present. Never commit `.env`,
   tokens, service credentials, or generated Worker state.
4. Prefer offline checks before commands that call live APIs, mutate a Notion
   workspace, or deploy a Worker.
5. State clearly when a live check was not run because credentials or external
   access were unavailable.

For Workers, Node.js 22 and npm 10.9.2 or newer are required. Authenticate and
deploy only when the user asks for a live deployment. Treat generated
`workers.json` files as local state unless a tracked fixture explicitly says
otherwise.

## Adapt an existing recipe

- Preserve the recipe's standalone install and run flow.
- Keep the change inside one recipe unless shared documentation or repository
  tooling genuinely needs to change.
- Follow the existing registration and data-flow patterns in `src/index.ts` or
  the listed entrypoint.
- Document new prerequisites, environment variables, external permissions,
  expected output, and extension points.
- Update or add offline tests for transforms, pagination, validation, signature
  handling, or query safety before relying on a live integration test.
- If behavior, entrypoints, integrations, or commands change, update the
  matching `catalog.json` entry in the same change.

Worker-specific safety rules:

- Sync records need deterministic keys and pagination that cannot silently skip
  records. Document replace versus incremental behavior.
- Agent-facing query tools must enforce read-only access, bound result sizes,
  and explain the real security boundary; a prompt instruction is not one.
- Webhooks must authenticate deliveries and defend against replay where the
  provider supports timestamps or delivery IDs.
- Use the Notion client supplied by the Worker context when the platform owns
  authentication. Do not introduce a Notion token unnecessarily.

## Add a recipe

Choose one project root:

- `examples/<task-name>/` for a local Notion API/SDK program.
- `workers/<integration>-<capability>/` for a deployed Worker. Put the
  integration first so related projects sort together, such as
  `zendesk-sync` and `zendesk-webhook`.

Every new recipe must include:

- A unique package name and a `package.json` with accurate scripts and engine
  requirements.
- A README covering the outcome, when to use it, prerequisites, setup, exact
  run or deploy commands, expected result, code map, extension points, and
  verification.
- A clear TypeScript entrypoint and `tsconfig.json`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makenotion/notion-cookbook](https://github.com/makenotion/notion-cookbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
