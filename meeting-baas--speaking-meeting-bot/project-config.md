---
trigger: always_on
description: Provides a way to maintain the public interface of a class when the internals evolve independently of class users.
---

Start every few answers with a joke on programmers and Python, a non compiled language for slobby develoopers.

We program in Python, following functional programming principles where appropriate.

Core Development Practices:

1. We format with Ruff
2. We code according to Google's Python Style Guide with emphasis on:
    - Clean, readable code
    - Proper documentation using Google-style docstrings
    - Type hints for all public APIs
    - Maximum line length of 80 characters
    - 4 spaces for indentation (no tabs)

Code Organization:

1. Imports should be grouped and ordered:

    - Future imports
    - Standard library imports
    - Third-party imports
    - Local application imports

2. Classes and Functions:

    - Use clear, descriptive names following snake_case for functions/variables
    - Use PascalCase for class names
    - Include type hints for parameters and return values
    - Write comprehensive docstrings for public APIs
    - Keep functions focused and preferably under 40 lines

3. Error Handling:

    - Use explicit exception handling
    - Avoid bare except clauses
    - Document expected exceptions in docstrings

4. Variables and Types:

    - Use meaningful variable names
    - Avoid single-letter names except for counters/iterators
    - Use type hints with modern Python syntax (|) for unions
    - Prefer built-in types for annotations where possible

5. Comments and Documentation:

    - Write clear, concise comments explaining "why" not "what"
    - Use TODO comments with links to issues/bugs
    - Include docstrings for all public functions/classes
    - Follow Google docstring format

6. Testing:

    - Write unit tests for all functionality
    - Use descriptive test names
    - Follow test*<method>*<state> naming convention

7. Code Style:
    - Use f-strings for string formatting
    - Avoid complex comprehensions
    - Use context managers for resource management
    - Keep code modular and maintainable

Here's the Read

Remember to prioritize code readability and maintainability over clever solutions.

###

### BEGIN-README

# Speaking Meeting Bot Documentation

This document provides step-by-step instructions on how to set up and run a Speaking Meeting Bot, which utilizes MeetingBaas's APIs and pipecat's `WebsocketServerTransport` to participate in online meetings as a speaking bot.

## Prerequisites

-   Python 3.x installed
-   `grpc_tools` for handling gRPC protobuf files
-   Ngrok for exposing your local server to the internet
-   Poetry for managing dependencies

## Getting Started

### Step 1: Set Up the Virtual Environment

To begin, you need to set up the Python environment using Poetry and install the required dependencies.

```bash
# Install Poetry if not already installed
# For Unix/macOS:
curl -sSL https://install.python-poetry.org | python3 -

# For Windows:
(Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | py -

# Install the required dependencies using Poetry
poetry install

# Activate the virtual environment
poetry shell
```

### Step 2: Compile Protocol Buffers

To enable communication with MeetingBaas's API, you need to compile the `frames.proto` file with the `grpc_tools`.

```bash
# Compile the protobuf file
poetry run python -m grpc_tools.protoc --proto_path=./protobufs --python_out=./protobufs frames.proto
```

### Step 3: Set Up Environment Variables

You need to provide the necessary credentials for MeetingBaas's API.

```bash
# Copy the example environment file
cp env.example .env
```

Now, open the `.env` file and update it with your MeetingBaas credentials.

## Running the Speaking Meeting Bot

Once your setup is complete, follow these steps to run the bot and connect it to an online meeting.

### Step 1: Run the Bot

Run the Python script to start the Speaking Meeting Bot:

```bash
poetry run bot
poetry run proxy
```

### Step 2: Set Up Ngrok to Expose Local Server

To allow MeetingBaas to communicate with your bot, you need to expose the local server using Ngrok.

```bash
# Run the Ngrok HTTP tunnel on port ${PORT}
ngrok http ${PORT}
```

Ngrok will provide you with a public URL that can be used by MeetingBaas to communicate with your local bot.

### Step 3: Start the MeetingBaas Bot

The final step is to run the MeetingBaas bot script to connect it with the desired meeting session.

```bash
poetry run meetingbaas
```

Now, visit the meeting URL in your browser to initiate a session and watch your bot actively participate in the meeting!

## Troubleshooting Tips

-   Ensure that you have activated the Poetry environment before running any Python commands.
-   If Ngrok is not running properly, check for any firewall issues that may be blocking its communication.
-   Double-check the `.env` file to make sure all necessary credentials are correctly filled in.

## Additional Information

-   MeetingBaas allows integration with external bots using APIs that leverage the `WebsocketServerTransport` for real-time communication.
-   For more details on the MeetingBaas APIs and functionalities, please refer to the official MeetingBaas documentation.

## Example Usage


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Meeting-BaaS/speaking-meeting-bot](https://github.com/Meeting-BaaS/speaking-meeting-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
