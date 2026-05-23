---
trigger: always_on
description: The application provides a graphical user interface (GUI) application that simplifies the creation of AI agents using PlanAI. The browser UI provides a node-based interface for configuring AI workflows, e.g. creating new Task Objects, various PlanAI TaskWorkers and connecting them with each other. The application uses Svelte 5 with SvelteKit for the frontend and integrate svelte-flow for the node-based graph interface, with a Python backend using flask and flask-socketio for executing PlanAI wor
---


# Purpose of the application
The application provides a graphical user interface (GUI) application that simplifies the creation of AI agents using PlanAI. The browser UI provides a node-based interface for configuring AI workflows, e.g. creating new Task Objects, various PlanAI TaskWorkers and connecting them with each other. The application uses Svelte 5 with SvelteKit for the frontend and integrate svelte-flow for the node-based graph interface, with a Python backend using flask and flask-socketio for executing PlanAI workflows.

The main entry point for the python backend is @app.py. The main entry point for the svelte 5 frontend is [+page.svelte](mdc:frontend/src/routes/+page.svelte)

# Main User Journeys
1. Create a new PlanAI graph in the UI and export it to Python for validation and further native code development.
2. Import an existing Python module that implements a PlanAI graph into the frontend where the user can modify and extend it
3. Export an imported Python module back to the Python backend. We apply the changes using the python ast module.


# PlanAI Best Practices for LLM-Assisted Coding

This document outlines best practices for using PlanAI, a Python framework for building complex, AI-enhanced workflows using a graph-based, data-flow architecture. PlanAI assumes TaskWorker classes to be unique. They don't get instantiated multiple times.

## Core Concepts & Best Practices

### 1. Graph-Based Architecture

* **Concept:** Workflows are defined as `planai.Graph` objects. A graph consists of nodes (`TaskWorker` subclasses) connected by dependencies, defining the data flow. Execution is parallel, constrained only by these data dependencies.
* **Best Practice:** Design workflows modularly. Break down complex processes into distinct `TaskWorker` units representing logical steps.
* **Instantiation:** Always instantiate a graph using `graph = Graph(name="YourWorkflowName")`.
* **Strict Mode:** Consider setting `strict=True` during graph initialization (`Graph(name="...", strict=True)`) to enforce stricter validation, like preventing the publishing of tasks that still hold provenance data, which can help catch bugs related to task reuse.

### 2. TaskWorkers: The Building Blocks

* **Concept:** Workers are Python classes inheriting from `planai.TaskWorker` (or specialized subclasses) that perform specific processing steps.
* **Core Logic:** Implement the primary logic within the `consume_work(self, task: YourInputTask)` method. Use type hints for the input `task` parameter for clarity and static analysis benefits.
* **Output:** Use `self.publish_work(output_task, input_task=task)` to send results downstream. It's crucial to pass the `input_task` to maintain provenance. Use `task.copy_public()` when creating `output_task` from the `input_task` if you don't want to pass along private state or provenance, especially in `strict` mode graphs[cite: 4654, 4655, 4656, 4657, 4658].
* **Input Type Declaration:** Define expected input types by type-hinting the `task` parameter in `consume_work`. For `LLMTaskWorker`, use `llm_input_type: Type[YourInputTask] = YourInputTask` if not overriding `consume_work`.
* **Output Type Declaration:** Explicitly declare output types using `output_types: List[Type[Task]] = [YourOutputTask1, YourOutputTask2]`. This enables type-aware routing.
* **Logging/Printing:** Use `self.print("Log message")` within workers instead of `print()`. This integrates with the graph's logging and dashboard.
* **Status Notifications:** Use `self.notify_status(task, "Descriptive status message")` to provide real-time updates visible in the dashboard or via callbacks.
* **Error Handling:** Implement `try...except` blocks within `consume_work` for robust error handling. Failed tasks are automatically tracked by the dispatcher.
* **Retries:** Set `num_retries: int = N` on a worker class to automatically retry failed tasks `N` times.

### 3. Tasks: Data Flow Units

* **Concept:** Data moves through the graph encapsulated in `planai.Task` objects, typically defined as Pydantic models.
* **Best Practice:** Define specific Pydantic models for each distinct data structure flowing between workers. This enforces type safety and improves code readability.
    ```python
    from planai import Task
    from pydantic import Field
    from typing import List

    class UserQuery(Task):
        query_text: str = Field(description="The user's input query")
        user_id: str

    class SearchResults(Task):
        query: str
        results: List[str] = Field(description="List of URLs")
    ```
* **Provenance:** Tasks automatically carry their execution history (`_provenance`) and the chain of tasks that led to them (`_input_provenance`).

### 4. Type Safety with Pydantic


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [provos/planaieditor](https://github.com/provos/planaieditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
