---
trigger: always_on
description: > **Attention AI Agent:** This is your primary instruction manual. Follow these rules strictly to generate correct, robust, and bug-free PHP code for the PocketFlow-PHP framework. Failure to adhere to these rules will result in errors.
---

# Agentic Coding with PocketFlow-PHP

> **Attention AI Agent:** This is your primary instruction manual. Follow these rules strictly to generate correct, robust, and bug-free PHP code for the PocketFlow-PHP framework. Failure to adhere to these rules will result in errors.

## The Golden Rules

1.  **THE CORE FRAMEWORK IS READ-ONLY.** The file `src/PocketFlow.php` is the engine. You are **strictly forbidden** from ever editing it.

2.  **USE THE THREE MAIN FILES.** All your application code **must** be written in these three files in the project root:
    -   `nodes.php`: For all `Node`, `BatchNode`, and `AsyncNode` class definitions.
    -   `flow.php`: For functions that create and wire up `Flows`.
    -   `main.php`: The single entry point that calls a function from `flow.php`.

3.  **UTILITIES GO IN `utils/`.** Any function that communicates with the outside world (e.g., calling an LLM API, a database, or a web search) **must** be placed in a new file inside the `utils/` directory.

4.  **STATE MANAGEMENT IS CRITICAL.**
    -   **DO** use the `$shared` object (`stdClass`) to manage all **mutable application state** (e.g., results, counters, lists of items).
    -   **DO NOT** use `static` properties or variables inside Node classes to manage state. This will fail in loops.
    -   **DO NOT** use `$this->params` to store mutable state. It is for **immutable configuration** only.

5.  **STRICT TYPE COMPATIBILITY & SCOPE.**
    -   The `post()` method of a `Node` **must** have the return type `?string`. If it does not decide the next action, it **must** end with `return null;`.
    -   All `_async` methods (`prep_async`, `exec_async`, `post_async`) **must** have the return type `React\Promise\PromiseInterface`.
    -   To return a promise, **always** use the pattern `return async(function() { ... })();`. Do not forget the final `()`.
    -   **DO NOT** use `use ($this)` in closures. To access `$this->params` inside a `post_async` or `exec_async` closure, read the required values into local variables *before* the closure and pass them in with `use()`.

6.  **ALWAYS IMPORT CLASSES WITH `use`.**
    -   Any file that references a class (e.g., `AsyncNode`, `Flow`, `PromiseInterface`) **must** include a `use` statement for that class at the top of the file.

7.  **DEFINE NODE CONNECTIONS *BEFORE* CREATING THE FLOW.**
    -   The `Flow` class does not have a public method to retrieve its start node. Therefore, you **must** configure all node connections (`->next()`, `->on()->next()`) *before* passing the start node to the `Flow` constructor. This is especially important for creating loops.
    -   **Correct Pattern:**
        ```php
        $myStartNode = new MyNode();
        $nextNode = new NextNode();
        $myStartNode->on('continue')->next($myStartNode); // Define loop on the node itself
        $myStartNode->on('finish')->next($nextNode);
        $flow = new Flow($myStartNode); // Then create the flow with the fully wired node
        ```

8.  **USE PHP 8.3 FEATURES WISELY.**
    -   **DO** use constructor property promotion for cleaner `Node` classes.
    -   **DO** use the `match` expression inside `post()` for clean, readable action routing.
    -   **DO NOT** use features like Fibers directly. Rely on the `async()` and `await()` functions provided by `react/async`.

9.  **ENABLE STRICT TYPES.**
    -   To prevent common type-related errors, **every new PHP file** you create (`nodes.php`, `flow.php`, `main.php`, and all utility files) **must** start with the following declaration on the very first line:
        ```php
        <?php
        declare(strict_types=1);
        ```

10. **MANAGE DEPENDENCIES WITH COMPOSER COMMANDS.**
    -   You **must not** edit the `composer.json` file manually.
    -   When a new library is needed for a utility function (e.g., `symfony/yaml` for parsing YAML), you **must** instruct the user to run the `composer require` command.
    -   **Example Instruction:** "To parse YAML, the `symfony/yaml` package is required. Please run: `composer require symfony/yaml`"

## The Development Workflow

Follow these steps in order. Refer to the correct code examples in this documentation.
1.  **Human: Define Requirements.** The human provides a high-level goal.
    - > "Build a system that summarizes news articles from a list of URLs."

2.  **AI: Propose a Plan.** Based on the documentation, the AI proposes a plan using PocketFlow concepts.
    > "I will create a `BatchFlow` to process each URL. The sub-flow will have two nodes: `FetchArticleNode` to download the content, and `SummarizeNode` to call an LLM. The final summaries will be stored in `$shared->summaries`."

3.  **Human: Approve the Plan.** The human confirms the logic.

4.  **AI: Generate the Code.** The AI writes the code for `utils/`, `nodes.php`, `flow.php`, and `main.php`, strictly following the Golden Rules and the Mandatory Project Structure.

5.  **Human & AI: Test and Iterate.** The human runs `php main.php`. If there are errors, the AI debugs them by reviewing the Golden Rules.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/weise25) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
