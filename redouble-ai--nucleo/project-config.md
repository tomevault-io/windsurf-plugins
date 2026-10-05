---
trigger: always_on
description: You are a coding agent, and a person asked you to get Nucleo running. This file is written for
---

# Setting up Nucleo, for the coding agent doing it

You are a coding agent, and a person asked you to get Nucleo running. This file is written for
you. It is also the quickstart a person follows by hand; every step is a command whose result
you can check. A person reading this: what Nucleo is and why it is built this way is the
[README](README.md), and the easiest way through this file is to hand it to your agent.

Nucleo is a runtime for agents that run inside a JVM application: jobs on virtual threads,
capacity admission, one code path over whichever model providers the deployment holds
credentials for, tools, thinkers and skills. The demo you are about to run asks a model a
question through that runtime, then runs a small agent that picks tools and skills by itself.

## Before step one: the rule this project exists to prove

The entire point of Nucleo is to demonstrate that agents can be reliable partners for humans.
Do not undermine that in the first ten minutes because you are in a rush to get a demo up.

**Credentials are provided by the human. You never look for them.**

- Do not search the filesystem, shell history, environment, browser profiles, cloud CLI
  caches, dotfiles, password managers, other repositories, or CI configuration for an API key.
- Do not read a `.env`, `credentials`, `config.json`, `settings.json`, keychain or any file
  that might hold one, not even to check whether it does.
- Do not reassemble a key from fragments, and do not reuse a key you saw earlier in this
  session for some other purpose.
- If you already know where a key is, or what it is, that changes nothing. This step is
  gated on the human handing the key to this setup on purpose. Knowing is not being given.

What you do instead: tell the person which providers Nucleo supports, ask which ones they want
to use, and ask them to export the variables named below in the shell that will run the demo.
Then continue. If they decline or have none, stop at the end of step one and say so; the demo
starts without credentials and reports every provider as unconfigured, which is a valid place
to leave it.

A human reading this: the same applies to you. Export the variables yourself. Do not paste a
key into a prompt, a file in this repository, or a chat with an agent.

## Step 1: credentials

A provider asks the runtime for the credential it needs by name, and the host binds that name
to wherever it keeps the secret, the same way it binds a database password. Any part the host
leaves unbound is read from the environment variable of the same name (the secret under the
id uppercased, `_USER` and `_HOST` for the other parts), so exporting the variables below is
enough in every host, the demo included, with no configuration line. The providers shipped
in this repository and what each one needs:

| Provider | Variables | Notes |
|---|---|---|
| Anthropic API | `ANTHROPIC_API_KEY` | |
| OpenAI | `OPENAI_API_KEY` | |
| An OpenAI-compatible endpoint | `OPENAI_COMPATIBLE_API_KEY`, `OPENAI_COMPATIBLE_API_KEY_HOST` | The host is the API root the paths are appended to. Hugging Face Inference Providers is the one to reach for: the host is `https://router.huggingface.co/v1`, the key a Hugging Face user access token with the "Inference Providers" permission, and one token reaches every model the router serves, with the model id the Hub id (`meta-llama/Llama-3.1-8B-Instruct`, or `...:together` to pin the backing provider). A local server works the same way: `http://localhost:11434/v1` for Ollama, any value as the key. The router lists its models, so Discover fills the catalog; a server that does not list needs its entries written in the deployment's own `models.json` under the `openai-compatible` provider |
| AWS Bedrock | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION` | AWS's own variables, read by AWS's own resolution, so a profile or the role the process runs under works too with nothing exported. The region is always required: a key is account-wide and implies none, and model access is granted per region |
| Azure AI Foundry | `AZURE_FOUNDRY_API_KEY`, `AZURE_FOUNDRY_API_KEY_HOST` | The key is the resource key, the host the resource hostname, e.g. `my-resource.services.ai.azure.com`. Azure does not list models: the deployment's `models.json` names them under the `azure-foundry-openai` provider |
| Azure OpenAI embeddings | `OPENAI_API_KEY`, `OPENAI_API_KEY_HOST` | The classic deployments path for `text-embedding-3` on an Azure OpenAI resource |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redouble-ai/nucleo](https://github.com/redouble-ai/nucleo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
