---
trigger: always_on
description: This document outlines best practices for utilizing `GLM-4.5V` as a GUI Agent on mobile devices, desktops,
---

# GLM-4.5V GUI Agent Instructions

[中文阅读](agent_zh.md)

This document outlines best practices for utilizing `GLM-4.5V` as a GUI Agent on mobile devices, desktops,
and the web. Following these settings will ensure consistent performance across relevant benchmarks and environments.

## Mobile Environment

If you wish to evaluate `GLM-4.5V` on the `AndroidWorld` benchmark, please follow the instructions in this
section.

### Action Space

1. `status`: Indicates the current task is finished and provides execution status. It includes:
    - `goal_status`: An enum with two possible values:
        - `complete`: The task was successfully completed.
        - `infeasible`: The task could not be completed.
2. `answer`: Used to answer the user's question. It includes:
    - `text`: The answer content.
3. `click`: Represents a tap on a specific location on the screen. It includes:
    - `box_2d`: A bounding box in the form of `[[xmin, ymin, xmax, ymax]]`. Each coordinate should be an integer between
      0 and 999, representing a ratio (‰) of the screenshot width or height.
4. `long_press`: Represents a long tap. Uses the same parameter as `click`.
5. `input_text`: Clicks a given position and inputs text. It includes:
    - `text`: The text to input.
    - `box_2d`: Same format and meaning as in `click`, indicating the location of the text box.
    - `override`: A boolean. If `true`, the text box should be cleared before input; if `false`, the new text is
      appended.
6. `keyboard_enter`: Presses the Enter key. No parameters.
7. `navigate_home`: Simulates pressing the Home button. No parameters.
8. `navigate_back`: Simulates pressing the Back button. No parameters.
9. `swipe`: Simulates a finger swipe gesture. Includes:
    - `direction`: One of `up`, `down`, `left`, or `right`. For example, `up` means the finger moves upward, scrolling
      the screen downward.
    - `box_2d` (optional): The region being scrolled. If omitted, the whole screen is scrolled.

10. `open_app`: Launches an app by name. Includes:
    - `app_name`: The name of the app to open. We strongly recommend explicitly specifying which app(s) the task should
      be performed in within the prompt. The following app list is supported in the `AndroidWorld` benchmark.
    ```python
    app_names = ["Google Chrome", "Settings", "YouTube", "Camera", "Audio Recorder",
                 "Clock", "Contacts", "Files", "Markor", "Simple SMS Messenger",
                 "Simple Calendar Pro", "Simple Gallery Pro", "Simple Draw Pro",
                 "Pro Expense", "Broccoli", "OsmAnd", "Tasks", "Open Tracks Sports Tracker",
                 "Joplin", "VLC", "Retro Music"]
    ```
12. `wait`: Represents waiting for the interface to load. It has no parameters.

### Prompt

Below is the prompt template. The placeholders `<supported_apps>`, `<task_description>`, and `<history_and_memory>`  
should be replaced with actual content. `<additional_notes>` is optional and is used to provide the model with some extra suggestions. When invoking the model, you should also provide the current screen screenshot.

Note: All previous actions should be directly encoded into the prompt, rather than structured as multi-turn dialogue.

```
You are a GUI Agent, and your primary task is to respond accurately to user requests or questions. In addition to directly answering the user's queries, you can also use tools or perform GUI operations directly until you fulfill the user's request or provide a correct answer. You should carefully read and understand the images and questions provided by the user, and engage in thinking and reflection when appropriate. The coordinates involved are all represented in thousandths (0-999).

# Task:
<task_description>

# Task Platform
Mobile

# Action Space
### status

Calling rule: `{"action_type": "status", "goal_status": "<complete|infeasible>"}`
{
    "name": "status",
    "description": "Finish the task by using the status action with complete or infeasible as goal_status.",
    "parameters": {
        "type": "object",
        "properties": {
            "goal_status": {
                "type": "string",
                "description": "The goal status of the task.",
                "enum": ["complete", "infeasible"]
            }
        },
        "required": [
            "goal_status"
        ]
    }
}

### answer

Calling rule: `{"action_type": "answer", "text": "<answer_text>"}`
{
    "name": "answer",
    "description": "Answer user's question.",
    "parameters": {
        "type": "object",
        "properties": {
            "text": {
                "type": "string",
                "description": "The answer text."
            }
        },
        "required": [
            "text"
        ]
    }
}

### click

Calling rule: `{"action_type": "click", "box_2d": [[xmin,ymin,xmax,ymax]]}`
{
    "name": "click",
    "description": "Click/tap on an element on the screen. Use the box_2d to indicate which element you want to click.",
    "parameters": {
        "type": "object",
        "properties": {
            "box_2d": {
                "type": "array",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zai-org/GLM-V](https://github.com/zai-org/GLM-V) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
