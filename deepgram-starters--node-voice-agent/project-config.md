---
trigger: always_on
description: deepgram.toml Requirements
---

# deepgram.toml Requirements

## File Purpose

The `deepgram.toml` file configures project metadata and build steps for the Deepgram CLI. It enables automated setup and deployment of starter applications.

## TOML  Config Sections

### [meta] Section

Every deepgram.toml must have this section. Describes the project, doesn't affect build/runtime.

[meta]
title = "Project Title"                    # REQUIRED: Short, descriptive title
description = "Brief description"          # REQUIRED: One-line description
author = "Deepgram DX Team <devrel@deepgram.com> (https://developers.deepgram.com)"  # REQUIRED: Always use this
useCase = "STT"                           # REQUIRED: See valid values below
language = "TypeScript"                    # REQUIRED: Primary language
framework = "Node"                         # OPTIONAL: Framework/runtime (omit if none)
sdk = "3.8.12"                            # OPTIONAL: Deepgram SDK version (omit if not used)**Valid `useCase` values:**

**UseCases**
- `STT` - Speech-to-Text / Transcription
- `TTS` - Text-to-Speech / Audio AI
- `Agent` - Voice Agent applications
- `Text Intel` - Text Intelligence / Analyze API
- `Live STT` - Live streaming transcription
- `Live TTS` - Live streaming text-to-speech

### Installation and Build Steps

Use when dependencies need installation before building.

[pre-build]
command = "pnpm run install:all"           # Command to install dependencies
message = "Dependencies installed"         # Success message to display### [build] Section (OPTIONAL)
Use when project requires compilation/build step.

[build]
command = "pnpm build"                     # Build command
message = "Build completed"                # Success message### [build.config] Section (OPTIONAL)
Use when project needs environment file setup.

[build.config]
sample = "sample.env"                      # Template env file
output = ".env"                            # Output env file### [post-build] Section (OPTIONAL)
Display instructions after build completes.

[post-build]
message = "Run `pnpm start` to get started."  # Instructions for user## Language-Specific Examples

### Language Specific Examples

#### Node.js / TypeScript

[meta]
title = "Node Transcription Starter"
description = "Get started using Deepgram's Transcription with this Node demo app"
author = "Deepgram DX Team <devrel@deepgram.com> (https://developers.deepgram.com)"
useCase = "STT"
language = "TypeScript"
framework = "Node"
sdk = "3.8.12"

[pre-build]
command = "pnpm run install:all"
message = "Dependencies installed (root + frontend)"

[build.config]
sample = "sample.env"
output = ".env"

[post-build]
message = "Run `pnpm start` to get up and running."

#### Python (Flask/Django)

[meta]
title = "Python TTS Starter"
description = "Get started with Deepgram's Text-to-Speech API in Python"
author = "Deepgram DX Team <devrel@deepgram.com> (https://developers.deepgram.com)"
useCase = "TTS"
language = "Python"
framework = "Flask"
sdk = "3.8.0"

[pre-build]
command = "pip install -r requirements.txt"
message = "Python dependencies installed"

[build.config]
sample = "sample.env"
output = ".env"

[post-build]
message = "Run `python app.py` to start the server."

#### Go

[meta]
title = "Go Voice Agent Starter"
description = "Build voice agents with Deepgram in Go"
author = "Deepgram DX Team <devrel@deepgram.com> (https://developers.deepgram.com)"
useCase = "Agent"
language = "Go"
sdk = "1.2.3"

[pre-build]
command = "go mod download"
message = "Go modules downloaded"

[build]
command = "go build -o agent main.go"
message = "Build completed"

[build.config]
sample = "sample.env"
output = ".env"

#### Ruby

[post-build]
message = "Run `./agent` to start the voice agent."### Ruby
[meta]
title = "Ruby Live Transcription"
description = "Real-time transcription with Deepgram in Ruby"
author = "Deepgram DX Team <devrel@deepgram.com> (https://developers.deepgram.com)"
useCase = "Live-STT"
language = "Ruby"
framework = "Sinatra"

[pre-build]
command = "bundle install"
message = "Ruby gems installed"

[build.config]
sample = "sample.env"
output = ".env"

[post-build]
message = "Run `ruby server.rb` to start the app."

#### Java

[meta]
title = "Java Text Intelligence"
description = "Analyze text with Deepgram in Java"
author = "Deepgram DX Team <devrel@deepgram.com> (https://developers.deepgram.com)"
useCase = "TextIntel"
language = "Java"
framework = "Spring Boot"

[pre-build]
command = "mvn clean install"
message = "Maven dependencies installed"

[build]
command = "mvn package"
message = "JAR built successfully"

[build.config]
sample = "sample.env"
output = ".env"

[post-build]
message = "Run `java -jar target/app.jar` to start."

#### Rust

[meta]
title = "Rust Speech-to-Text"
description = "Transcribe audio with Deepgram in Rust"
author = "Deepgram DX Team <devrel@deepgram.com> (https://developers.deepgram.com)"
useCase = "STT"
language = "Rust"

[pre-build]
command = "cargo fetch"
message = "Cargo dependencies fetched"

[build]
command = "cargo build --release"
message = "Release build completed"

[build.config]
sample = "sample.env"
output = ".env"

[post-build]
message = "Run `cargo run --release` to start."

#### PHP

[meta]
title = "PHP TTS Example"
description = "Generate speech with Deepgram in PHP"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deepgram-starters/node-voice-agent](https://github.com/deepgram-starters/node-voice-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
