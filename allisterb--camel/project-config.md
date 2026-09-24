---
trigger: always_on
description: The Camel project is a 'code-mode' MCP server that allows LLMs to safely generate and execute code that calls command-line forensic tools, performs processing, and employs classical machine learning algorithms and probabilistic reasoning using [SIFT workstation](https://www.sans.org/tools/sift-workstation) for automating DFIR.
---

# About this project
The Camel project is a 'code-mode' MCP server that allows LLMs to safely generate and execute code that calls command-line forensic tools, performs processing, and employs classical machine learning algorithms and probabilistic reasoning using [SIFT workstation](https://www.sans.org/tools/sift-workstation) for automating DFIR. 
Camel is designed to leverage the massive amounts of program generation and analysis data LLMs are trained on and provides a typed SDK and constrained code execution environment for programmatically acquiring, filtering, querying, analyzing, and reasoning over forensic tool data, as a context-efficient alternative to high-level agentic reasoning over MCP tool outputs employed
by [Protocol SIFT](https://github.com/teamdfir/protocol-sift/tree/main) and other DFIR AI-automation projects. Code-mode is a technique for [programmatic tool calling](https://platform.claude.com/cookbook/tool-use-programmatic-tool-calling-ptc) by agents using a code execution environment described by [Cloudfare](https://blog.cloudflare.com/code-mode-mcp/) and [Anthropic](https://www.anthropic.com/engineering/code-execution-with-mcp)
that "substantially reduces end-to-end latency for multiple tool calls, and can dramatically reduce token consumption by allowing the model to write code that removes irrelevant context before it hits the model’s context window." In addition, many forensic analysis tasks are highly suited to lower-level machine learning techniques and algorithms like classification, decision trees, and time-series anomaly detection. 
Camel exposes an API for using implementations of these algorithms on forensic tool data as well as high-level workflows for acquiring and processing and analyzing forensic tool data, as an alternative to requiring the LLM to spend tokens and time on naively performing these low-level classification and analysis and inference tasks. Forensic analysis using Camel reduces to the task of generating the correct programs for ingesting, analyzing, and performing inference over forensic data using the provided API. 
Camel thus allows LLMs to efficiently and effectively reason over far higher-level forensic data features and measures than traditional DFIR AI-automation projects.

Camel is created as an entry into the [SANS Find Evil! AI Hackathon](https://findevil.devpost.com/).
	
## Project design and architecture
Camel is written in .NET and C#. It is designed to run either installed locally on the SIFT Workstation, or on a separate machine that can access a SIFT workstation over SSH. There are 7 main projects:
- Camel.Runtime at src/Camel.Runtime provides global base types and features like logging for all other projects.
- Camel.Environments at src/Camel.Runtime provides different **audit environments** that represent the local or remote machine SIFT workstation is running on. An audit environment allows common I/O operations like running commands and reading files to be abstracted so
the same code works locally or remotely over SSH.
- Camel.Toolkits at src/Camel.Runtime provides a strongly-typed, asynchronous API for the SIFT tools. 
- Camel.Workflows at src/Camel.Workflows codifies existing forensic tool knowledge into high-level workflows utilizing the SIFT tools API.
- Camel.Server at src/Camel.Server provides the constrained JavaScript execution engine and MCP server implementation.
- Camel.Training at src/Camel.Training For training/evaluating ML over forensic timelines and generating synthetic data: the embedding/novelty stack (TimelineNoveltyBaseline, ONNX embedders via Camel.Search, renderers), the eval harnesses (AnomalyDetectionEval metrics, DatasetEvaluator), SyntheticIntrusion, and the CSV/dataset loaders. References Camel.Inference. NOTE: Camel.Search is for the JS-SDK vector search only and must NOT be a dependency of any toolkit/Inference/Server — only Camel.Training (the experiment project) references it.
- Camel.Inference at src/Camel.Inference The lean runtime/inference ML core (no ONNX/Search dependency): the canonical event model (CanonicalEvent, EventCanonicalizer, ContentSignals, NoiseFilters), windowing, the (event_id, Δt) anomaly detectors + ensemble (EventDetectors), and the agent-facing AnomalyDetectionToolkit triage façade. Exposed to the code-mode agent's JS engine as `anomaly`.
- Camel.CLI at src/Camel.CLI provides the main interface for launching the Camel MCP server and other programs.

## Project milestones

- Implement local and SSH audit environments to be used by toolkits and workflows
- Define all toolkits and tools to be implemented in Camel.Toolkits, and the models that represent their outputs.
- Define higher-level workflows to be implemented in Camel.Workflows.
- Implement anomaly detection techniques in Camel.Inference.
- Implement the code-mode MCP server and robust audit logs.
- Run Claude/Camel on the given cases

## Project implementation

### Camel.Toolkits

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [allisterb/Camel](https://github.com/allisterb/Camel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
