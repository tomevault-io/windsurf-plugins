---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Description

This is an **MCP on Rails template** - a Rails application template that seamlessly integrates the [Model Context Protocol (MCP)](https://github.com/anthropics/model-context-protocol) with Ruby on Rails applications using the [`mcp` gem](https://rubygems.org/gems/mcp).

The template supports two modes — **plain MCP** (open endpoint) and **OAuth MCP** (Devise + Doorkeeper with OAuth 2.1, PKCE, dynamic client registration, and resource indicators). The mode is selected interactively during setup.

The project consists of two main components:
1. **`mcp`** - The Rails application template file
2. **`mcp_template/`** - Directory containing template files copied to new Rails applications

The `mcp` template bootstraps a new Rails app with MCP server capabilities. When run with `rails new myapp -m mcp`, it:

1. Adds the `mcp` gem (and optionally `devise` + `doorkeeper`) to the Gemfile
2. Copies template files from `mcp_template/` directory
3. Creates an `McpController` that handles MCP protocol requests at `/mcp` endpoint (streamable HTTP transport)
4. Sets up Rails generator hooks for automatic MCP tool generation during scaffolding
5. Adds a `to_mcp_response` method to ActiveRecord models for consistent MCP formatting
6. Configures Rails to ignore generators from autoloading
7. Optionally sets up OAuth 2.1 with Devise authentication, Doorkeeper OAuth provider, PKCE enforcement, dynamic client registration (RFC 7591), authorization server metadata (RFC 8414), protected resource metadata (RFC 9728), and resource indicators (RFC 8707)

When you scaffold new models (`rails generate scaffold Post title:string`), MCP tools are automatically generated alongside standard Rails files (the generator prompts to select All, Some, or None), providing AI assistants with structured access to CRUD operations.

## How Rails generators work:
Read contents of https://raw.githubusercontent.com/rails/rails/refs/heads/main/guides/source/generators.md to get context about Rails generators.

## MCP Tools Architecture

MCP Tools are Ruby classes that inherit from `MCP::Tool` and provide AI assistants with structured access to application functionality:

- **Location**: `app/tools/` directory
- **Structure**: Each tool defines `description`, `input_schema`, and `call` method. Scaffold-generated tools also set `tool_name`.
- **Autoloading**: Tools are automatically loaded via `config/initializers/mcp.rb`
- **Generation**: Use `rails generate mcp_tool ToolName field:type` to create new tools
- **Response Format**: Tools return `MCP::Tool::Response` objects with text content
- **Auto-generation**: Scaffold prompts to generate CRUD tools per model (All/Some/None): show, index, create, update, delete

### Generated Tool Types

For each scaffolded model, these tools are automatically created:
- **Show Tool**: Retrieve single record by ID
- **Index Tool**: List records with filtering by references and pagination (count parameter, default: 10)
- **Create Tool**: Create new records with validation
- **Update Tool**: Update existing records with validation
- **Delete Tool**: Delete records by ID

### Tool Structure Example:
```ruby
module Posts
  class CreateTool < MCP::Tool
    tool_name "post-create-tool"
    description "Create a new Post entity"
    
    input_schema(
      properties: {
        title: { type: "string" },
        content: { type: "string" }
      }
    )

    def self.call(title: nil, content: nil, server_context:)
      post = Post.new(title: title, content: content)
      
      if post.save
        MCP::Tool::Response.new([{ type: "text", text: "Created #{post.to_mcp_response}" }])
      else
        MCP::Tool::Response.new([{ type: "text", text: "Post was not created due to the following errors: #{post.errors.full_messages.join(', ')}" }])
      end
    rescue StandardError => e
      MCP::Tool::Response.new([{ type: "text", text: "An error occurred, what happened was #{e.message}" }])
    end
  end
end
```

### Type Mapping
- **String/Text fields**: `type: "string"`
- **Integer/Reference fields**: `type: "integer"`
- **Boolean fields**: `type: "boolean"`
- **References**: Automatically included in filtering and as required fields

## MCP Prompts Architecture

MCP Prompts are Ruby classes that inherit from `MCP::Prompt` and provide reusable prompt templates for AI assistants:

- **Location**: `app/prompts/` directory
- **Structure**: Each prompt defines `prompt_name`, `description`, `arguments`, and `template` method
- **Autoloading**: Prompts are automatically loaded via `config/initializers/mcp.rb`
- **Generation**: Use `rails generate mcp_prompt PromptName arg arg:required` to create new prompts
- **Not auto-generated**: Unlike tools, prompts are only created explicitly via the generator

## Template Architecture

The template uses Rails' generator hook system to extend the standard scaffold controller generator. This approach is:
- **Lightweight**: Only adds MCP-specific code, doesn't override Rails generators
- **Maintainable**: Works with Rails updates automatically  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pstrzalk/mcp-on-rails](https://github.com/pstrzalk/mcp-on-rails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
