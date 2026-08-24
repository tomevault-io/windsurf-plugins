---
trigger: always_on
description: This is an advanced, experimental rule. It instructs the AI Agent to check for, potentially create, and then execute a Python script (`final_review_gate.py`) in your project's root directory to facilitate an interactive review loop. By using this rule, you acknowledge that the AI will attempt file system operations (read, write, execute) as directed. Ensure the AI's execution environment has appropriate permissions if you expect script auto-creation to work, and be aware of the security implicat
---

# AI Agent Protocol: Self-Contained Mandatory Interactive Final Review Loop with Auto-Script Creation (Version 3)

## 0. User Advisory & Safety:
This is an advanced, experimental rule. It instructs the AI Agent to check for, potentially create, and then execute a Python script (`final_review_gate.py`) in your project's root directory to facilitate an interactive review loop. By using this rule, you acknowledge that the AI will attempt file system operations (read, write, execute) as directed. Ensure the AI's execution environment has appropriate permissions if you expect script auto-creation to work, and be aware of the security implications.

## 1. Core Principle & Context:
Your primary task processing includes a **mandatory final interactive review stage** driven by user input through a specific Python script (`final_review_gate.py`). You are NOT to consider your overall task complete until this review stage explicitly signals completion. This protocol leverages your known capability to run commands in a sub-terminal (displayed within the chat interface) and read their output.

## Phase 0: Ensure `final_review_gate.py` Script Exists
(This phase is executed ONCE per user request that triggers this overall protocol, or if the script is missing or its content is incorrect.)

1.  **Define Script Details:**
    * **Script Name:** `final_review_gate.py`
    * **Target Location:** Directly in the root of the current project/workspace.
    * **Python Script Content (ensure this exact content is used):**
        ```python
        # final_review_gate.py
        import sys
        import os

        if __name__ == "__main__":
            # Try to make stdout unbuffered for more responsive interaction.
            # This might not work on all platforms or if stdout is not a TTY,
            # but it's a good practice for this kind of interactive script.
            try:
                sys.stdout = os.fdopen(sys.stdout.fileno(), 'w', buffering=1)
            except Exception:
                pass # Ignore if unbuffering fails, e.g., in certain environments

            try:
                sys.stderr = os.fdopen(sys.stderr.fileno(), 'w', buffering=1)
            except Exception:
                pass # Ignore

            print("--- FINAL REVIEW GATE ACTIVE ---", flush=True)
            print("AI has completed its primary actions. Awaiting your review or further sub-prompts.", flush=True)
            print("Type your sub-prompt, or one of: 'TASK_COMPLETE', 'Done', 'Quit', 'q' to signal completion.", flush=True) # MODIFIED
            
            active_session = True
            while active_session:
                try:
                    # Signal that the script is ready for input.
                    # The AI doesn't need to parse this, but it's good for user visibility.
                    print("REVIEW_GATE_AWAITING_INPUT:", end="", flush=True) 
                    
                    line = sys.stdin.readline()
                    
                    if not line:  # EOF
                        print("--- REVIEW GATE: STDIN CLOSED (EOF), EXITING SCRIPT ---", flush=True)
                        active_session = False
                        break
                    
                    user_input = line.strip()

                    # Check for exit conditions
                    if user_input.upper() in ['TASK_COMPLETE', 'DONE', 'QUIT', 'Q']: # MODIFIED: Empty string no longer exits
                        print(f"--- REVIEW GATE: USER SIGNALED COMPLETION WITH '{user_input.upper()}' ---", flush=True)
                        active_session = False
                        break
                    elif user_input: # If there's any other non-empty input (and not a completion command)
                        # This is the critical line the AI will "listen" for.
                        print(f"USER_REVIEW_SUB_PROMPT: {user_input}", flush=True)
                    # If user_input was empty (and not a completion command),
                    # the loop simply continues, and "REVIEW_GATE_AWAITING_INPUT:" will be printed again.
                    
                except KeyboardInterrupt:
                    print("--- REVIEW GATE: SESSION INTERRUPTED BY USER (KeyboardInterrupt) ---", flush=True)
                    active_session = False
                    break
                except Exception as e:
                    print(f"--- REVIEW GATE SCRIPT ERROR: {e} ---", flush=True)
                    active_session = False
                    break
                    
            print("--- FINAL REVIEW GATE SCRIPT EXITED ---", flush=True)
        ```

2.  **Ensure Script Existence and Correctness:**
    a.  Use your file system tools to check if the file `final_review_gate.py` exists in the project root.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lishuoqwq/PixelBeans](https://github.com/lishuoqwq/PixelBeans) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
