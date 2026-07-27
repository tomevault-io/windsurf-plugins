---
trigger: always_on
description: This document introduces the usage and development process of the Agent class.
---

# Agent Introduction

This document introduces the usage and development process of the Agent class.

## 1. Agent Usage
The Agent class serves as a higher-level interface for Qwen-Agent, where an Agent object integrates the interfaces for tool calls and LLM (Large Language Model).
The Agent receives a list of messages as input and produces a generator that yields a list of messages, effectively providing a stream of output messages.

Different Agent classes have various workflows. In the [agents](../../../../../../qwen_agent/agents) directory, we provide several different fundamental Agent subclasses.
For instance, the [ArticleAgent](../../../../../../qwen_agent/agents/article_agent.py) returns a message that includes an article;
the [BasicDocQA](../../../../../../qwen_agent/agents/doc_qa/basic_doc_qa.py) returns a message that contains the results of a document Q&A Results.

These types of Agents have relatively fixed response patterns and are suited for fairly specific use cases.

### 1.1. Assistant Class
We offer a generic Agent class: the [Assistant](../../../../../../qwen_agent/agents/assistant.py) class,
which, when directly instantiated, can handle the majority of Single-Agent tasks.
Features:
- It supports role-playing;
- It provides automatic planning and tool calls abilities;
- RAG (Retrieval-Augmented Generation): It accepts documents input, and can use an integrated RAG strategy to parse the documents.

For example, in the following scenario, we instantiate a assistant Agent by specifying the LLM, a list of tools, and the role instruction.
Then we can interact with the Agent.

From the responses, we can see that after the user requests 'a cute cat',
the Agent, in keeping with the role-play instruction, automatically plans and executes the necessary tools for drawing the cat, downloading it and flipping it.

```py
import os
from qwen_agent.agents import Assistant
llm_cfg = {'model': 'qwen-max'}
tools = ['image_gen', 'amap_weather']  # image_gen and code_interpreter is a built-in tool in Qwen-Agent
system = 'According to the user\'s request, you first draw a picture and then automatically run code to download the picture ' + \
          'and select an image operation from the given document to process the image'

bot = Assistant(llm=llm_cfg,
                system_message=system,
                function_list=tools,
                files=[os.path.abspath('doc.pdf')])


messages = [{'role': 'user', 'content': 'a cute cat'}]
for response in bot.run(messages=messages):
    print('bot response:', response)

"""Running results：
[
    {'role': 'assistant', 'content': '', 'function_call': {'name': 'my_image_gen', 'arguments': '{"prompt": "a cute cat"}'}},
    {'role': 'function', 'content': '{"image_url": "https://image.pollinations.ai/prompt/a%20cute%20cat"}', 'name': 'my_image_gen'},
    {'role': 'assistant', 'content': "Here is the image of a cute cat based on your description:\n\n![](https://image.pollinations.ai/prompt/a%20cute%20cat)\n\nNow, let's proceed with downloading this image and performing an operation such as flipping it."},
    {'role': 'assistant', 'content': '', 'function_call': {'name': 'code_interpreter', 'arguments': '```py\nfrom PIL import Image\nimport requests\n\n# Downloading the cute cat image\nurl = "https://image.pollinations.ai/prompt/a%20cute%20cat"\nresponse = requests.get(url)\nwith open("cute_cat.jpg", \'wb\') as file:\n    file.write(response.content)\n\n# Flipping the image horizontally\ndef flip_image_horizontally(filename):\n    img = Image.open(filename)\n    flipped_img = img.transpose(Image.FLIP_LEFT_RIGHT)\n    flipped_img.save("flipped_cute_cat.jpg")\n    return flipped_img\n\n# Performing the horizontal flip\nflipped_cat = flip_image_horizontally("cute_cat.jpg")\n```'}},
    {'role': 'function', 'content': 'Finished execution.', 'name': 'code_interpreter'},
    {'role': 'assistant', 'content': 'The image of the cute cat has been downloaded and flipped horizontally. The flipped image has been saved as "flipped_cute_cat.jpg". Since we\'re in a text-based environment, I can\'t display the actual image here, but you can check it out at the location where the script was executed.'}
]
"""
```

In the [examples](../../../../../../examples) directory,
we provide more Single-Agent use cases developed based on the Assistant class.

### 1.2. GroupChat Class
We also provide a generic Multi-Agent class: the [GroupChat](../../../../../../qwen_agent/agents/group_chat.py) class. This class manages a list of Agents and automatically maintains their speech orders.
The features of this class include:
- Upon receiving external input, it automatically coordinates the speaking order of the built-in Agents and sequentially returns their responses to the user;
- Human-in-the-loop: The user is also defined as an Agent, and the group chat may request feedback from the user when necessary;
- The user can interrupt the group chat at any time.

In the [examples](../../../../../../examples) directory, we provide a Gradio [Demo](../../../../../../examples/group_chat_demo.py) for creating and experiencing group chats,
where you can further explore the group chat functionality.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QwenLM/Qwen-Agent](https://github.com/QwenLM/Qwen-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
