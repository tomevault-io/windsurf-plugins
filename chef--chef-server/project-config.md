---
trigger: always_on
description: This document provides comprehensive development guidelines and best practices for working on the Chef Server codebase. It helps GitHub Copilot provide more accurate and contextually appropriate suggestions while ensuring consistency across all team members and contributors.
---

# GitHub Copilot Instructions for Chef Server

This document provides comprehensive development guidelines and best practices for working on the Chef Server codebase. It helps GitHub Copilot provide more accurate and contextually appropriate suggestions while ensuring consistency across all team members and contributors.

## Quick Reference

### Essential Commands
```bash
# Development Environment
cd dev && vagrant up              # Start development VM
dvm load <service> && dvm start <service>  # Load and start service

# Building & Testing
make all                         # Erlang: clean, compile, eunit, dialyzer
rebar3 do clean, compile, ct     # Erlang: full test suite
bundle exec rspec                # Ruby: run RSpec tests
make ci                         # Run continuous integration suite

# Code Quality
./scripts/elvis rock            # Erlang style checking
bundle exec rubocop             # Ruby style checking (where applicable)
```

### File Patterns to Recognize
- `rebar.config` / `rebar.lock` → Erlang project dependencies
- `Gemfile` / `Gemfile.lock` → Ruby project dependencies  
- `*.erl` / `*.hrl` → Erlang source/header files
- `*_SUITE.erl` → Common Test suites
- `*_tests.erl` → EUnit test files
- `*_spec.rb` → RSpec test files

## Project Overview

Chef Server is a comprehensive infrastructure automation platform built as a distributed system with multiple services. The codebase is primarily written in **Erlang**, **Ruby**, and includes **Rails applications**, with build systems using **Omnibus** and **Habitat**.

## Architecture & Components

### Core Services (located in `src/`)

- **oc_erchef**: Core REST API server (Erlang/OTP)
- **bookshelf**: S3-compatible storage engine for cookbook data (Erlang/OTP)
- **oc_bifrost**: Authorization service (Erlang/OTP)
- **oc-id**: OAuth2 provider (Ruby on Rails)
- **chef-server-ctl**: Command-line management utility (Ruby)
- **nginx**: Web server with custom configurations

### Build & Packaging

- **omnibus/**: Omnibus-based packaging for production builds
- **habitat/**: Habitat-based containerization
- **dev/**: Vagrant-based development environment (DVM - Development Virtual Machine)

## Language-Specific Guidelines

### Erlang/OTP Development

#### Project Structure
- Use **rebar3** as the build tool for all Erlang projects
- Follow OTP application structure with proper supervision trees
- Each service has its own `rebar.config` and `rebar.lock` files

#### Code Standards

##### Formatting & Style
- **Line Length**: Maximum 120 characters per line
- **Indentation**: Use 4 spaces (no tabs) for consistency
- **Module Layout**: Follow standard OTP module structure:
  ```erlang
  %% Header comments and module documentation
  -module(module_name).
  
  %% Behaviors and includes
  -behaviour(gen_server).
  -include("module_name.hrl").
  
  %% Exports grouped by functionality
  %% API exports
  -export([start_link/0, stop/0]).
  %% gen_server callbacks
  -export([init/1, handle_call/3, handle_cast/2, handle_info/2]).
  
  %% Type definitions
  -type state() :: #state{}.
  
  %% Macros and records
  -define(DEFAULT_TIMEOUT, 5000).
  ```

##### Naming Conventions
- **Functions/Variables**: `snake_case` (e.g., `process_request`, `UserId`)
- **Modules**: `snake_case` (e.g., `chef_server_api`, `user_manager`)
- **Atoms**: `snake_case` (e.g., `:ok`, `:error`, `:not_found`)
- **Constants/Macros**: `UPPER_SNAKE_CASE` (e.g., `?DEFAULT_TIMEOUT`)
- **Records**: `snake_case` with descriptive names (e.g., `#user_data{}`, `#request_context{}`)

##### Modern Language Features
- **Maps over Records**: Prefer maps for simple data structures, records for complex state
- **Binary Strings**: Use binary strings for text data: `<<"string">>`
- **List Comprehensions**: Use for data transformation when appropriate
- **Guards**: Utilize guards for input validation and optimization
- **Pattern Matching**: Leverage pattern matching for control flow and data extraction

##### Function Design
- Use **lager** for logging with appropriate log levels (`debug`, `info`, `warning`, `error`)
- Implement proper error handling with pattern matching: `{ok, Result} | {error, Reason}`
- Use appropriate OTP behaviors (gen_server, gen_statem, supervisor)
- Include comprehensive function documentation with `-spec` attributes
- Use meaningful variable names that describe the data being processed
- Avoid deeply nested case statements; prefer helper functions
- **Function Length**: Keep functions under 50 lines; extract complex logic into helper functions
- **Arity**: Prefer functions with fewer than 5 parameters; use records/maps for complex data

#### Logging Patterns
```erlang
%% Use lager macros for different log levels
?LOG_DEBUG("Processing request: ~p", [Request]),
?LOG_INFO("User ~s authenticated successfully", [Username]),
?LOG_ERROR("Database connection failed: ~p", [Error])

%% Include request IDs for tracing
?LOG_INFO("Request ~s: Operation completed", [ReqId])
```

#### Error Handling Patterns
```erlang
%% Prefer pattern matching over exceptions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chef/chef-server](https://github.com/chef/chef-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
