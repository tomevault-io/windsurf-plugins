---
trigger: always_on
description: Handles request parsing and calls the inference pipeline.
---

# CLAUDE.md

## Project

This Python tool generates vision AI pipelines.

It works by combining:

- inference backend templates (e.g. Deepstream, TensorRT, vLLM),
- reusable common libraries,
- custom user-defined processors,

into a complete executable inference pipeline.

## Build and Run

A Python virtual environment must exist in the project root directory.

Example:

python3 -m venv .venv
source .venv/bin/activate
pip3 install -r requirements.txt

## Project Structure

The project consists of three main modules:

- builder/ - A Python command-line tool that generates inference pipeline source code.
- lib/ - Shared libraries that will be included in the generated pipeline.
- templates/ - Jinja template files used to generate code for different servers and inference backends.

Additional modules:

- doc/ - Documentation for end users
- schemas/ - Schema definitions describing pipeline objects, configurations, and data structures used by the pipeline.
- mcp/ - MCP server implementation that exposes this tool and its functionality to AI agents.

## Working Flow

### Adding support for a new inference backend

To add a new inference backend, follow these steps:

1. Create a new template file under:
   templates/backend/<backend_name>.jinja.py

2. In this file, define a new backend class:
   <backend_name>Backend

3. The backend class must inherit from:
   ModelBackend

4. Implement the following methods:

   __init__(self, config)
       Initialize the backend using the provided configuration.

   __call__(self, *args, **kwargs)
       Run inference for one batch and yield results when available.

       Batch input behavior:
       - Explicit batch mode:
         Input tensors are provided through *args as a list.
       - Implicit batch mode:
         Input tensors are provided through **kwargs as named tensors.

       The method should yield inference results as soon as they are available.

5. Add a sample configuration and corresponding test configurations for the new backend under:
   builder/samples/<example_name>/

6. Update documentation and schema files.

### Adding support for a new API server

To implement a new API server:

Directory structure:
    templates/api_server/<api_server_name>/

Required templates:
    templates/api_server/<api_server_name>/route/inference.jinja2
    templates/api_server/<api_server_name>/responder.jinja.py

Implementation requirements:

1. inference.jinja2
    Defines the API route for inference requests.
    Handles request parsing and calls the inference pipeline.

2. responder.jinja.py
    Defines the Responder class.

    Responder responsibilities:
        - Convert incoming API requests into pipeline inputs.
        - Convert pipeline outputs into API responses.

### Adding a new responder

A responder implements a function that can be associated with a specific API endpoint.
Responder templates live under templates/responder/ and are Jinja templates that render
into async methods embedded in the Responder class.

To add a new responder:

1. Create a new template file:
   templates/responder/<responder_name>.jinja.py

2. The template must define a single async method using the Jinja variable `{{ name }}`:

       async def {{ name }}(self, request, ...):
           ...
           return <status_code>, <response>

   The method receives `self` (the Responder instance) and `request` (the HTTP request).
   Additional parameters depend on the endpoint (e.g. `body` for inference, `file` for uploads).
   It must return a tuple of (HTTP status code, response body).

3. Available helpers from the Responder base class:
   - self.process_request(name, body) - converts the API request into pipeline input
   - self.process_response(name, request, result) - converts pipeline output into an API response
   - self._inference.execute(data) - runs the inference pipeline (async generator)
   - self._asset_manager - manages file assets (save, list, delete)
   - self.logger - logger instance

4. Register the responder in the pipeline config YAML under server.responders:

       server:
         responders:
           <responder_name>:
             operation: <openapi_operation_id>
             requests:
               ...
             responses:
               ...

   The key must match the template filename (without .jinja.py).

Existing responder templates for reference:
- infer.jinja.py - runs inference with streaming support
- healthy_ready.jinja.py - health/readiness check
- add_file.jinja.py / del_file.jinja.py / list_files.jinja.py - file asset management
- add_live_stream.jinja.py / del_live_stream.jinja.py / list_live_streams.jinja.py - live stream management

### Extending the MCP Server

To add a new tool:

1. Add the tool definition to the `list_tools` method in `mcp/mcp_server.py` — specify `name`, `description`, and `inputSchema`.
2. Add a dispatch branch in `call_tool` that routes the tool name to an implementation method.
3. Implement the tool logic as an `async` method on `InferenceBuilderMCPServer`.
4. Add tests in `mcp/test_mcp_server.py`.

## Coding Convention

### Documentation and Schema Consistency

Any code change that affects behavior, configuration, or APIs MUST include
corresponding updates to documentation and schemas. This is critical:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-AI-IOT/inference_builder](https://github.com/NVIDIA-AI-IOT/inference_builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
