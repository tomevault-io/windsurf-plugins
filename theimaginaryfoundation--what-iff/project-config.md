---
trigger: always_on
description: This project is a personal assistant AI agent. It's designed to conduct research, help write emails, documents, and other content. It's also designed to be extensible via MCP to support integrations with external services to enable additional agent capabilities.
---

# General Project Guidelines

## Project Overview
This project is a personal assistant AI agent. It's designed to conduct research, help write emails, documents, and other content. It's also designed to be extensible via MCP to support integrations with external services to enable additional agent capabilities. 

## Project Structure
The project follows standard Go project layout conventions:
- `cmd/` - Main applications for this project
  - `api-server/` - Backend API server for the personal assistant
- `internal/` - Library code that can be used by other applications
  - `datastore/` - Database access layer using Ent ORM
  - `models/` - Data structures shared across the application
  - `server/` - Main startup logic for API server
  - `handlers/` - API endpoint handlers for API server
  - `database/` - Database connection and configuration utilities
- `ent/` - Auto-generated Ent ORM code for database entities

## Development Best Practices

### Go Coding Standards
1. **Idiomatic Go**: Follow the Go proverbs and standard conventions
   - Use proper error handling (no panics in production code)
   - Embrace interfaces for dependency inversion
   - Prefer composition over inheritance
   - Use meaningful variable names (`i` for indexes, `err` for errors)

2. **Code Organization**:
   - Keep functions small and focused on a single responsibility
   - Group related functions in packages with clear boundaries
   - Use interfaces to define behavior at package boundaries
   - Extract complex logic into well-named helper functions immediately
   - Avoid deeply nested loops and conditionals (aim for cyclomatic complexity < 10)
   - Use early returns to reduce nesting levels

3. **Error Handling**:
   - Always check errors, never use `_` to ignore errors without justification
   - Return errors rather than using panic
   - Use custom error types or error wrapping for context
   - Use `fmt.Errorf()` with `%w` for wrapping errors

4. **Concurrency**:
   - Use channels and goroutines appropriately
   - Always ensure proper goroutine termination
   - Use context for cancellation and timeouts
   - Use sync primitives (Mutex, WaitGroup) when appropriate

5. **Function Design and Complexity Management**:
   - Write small, focused functions from the start (avoid large monolithic functions)
   - When a function grows beyond ~50 lines, consider extracting helper functions
   - Use helper functions to make code self-documenting through clear naming
   - Avoid nesting more than 3 levels deep - extract nested logic into separate functions
   - Use early returns to reduce nesting and improve readability
   - Replace complex `if-else` chains with `switch` statements or lookup tables
   
   ```go
   // Good: Clear, focused functions with minimal nesting
   func handleRequest(req Request) error {
       if !req.Valid {
           return ErrInvalidRequest
       }
       
       for _, item := range req.Items {
           if err := processItem(item); err != nil {
               return err
           }
       }
       return nil
   }
   
   func processItem(item Item) error {
       switch item.Type {
       case "A":
           return processTypeA(item)
       case "B":
           return processTypeB(item)
       default:
           return ErrUnknownType
       }
   }
   ```

6. **Data Structure Design for Maintainability**:
   - Design structures that are self-contained and capture all needed data at creation
   - Avoid parallel arrays or slices that must stay synchronized by index
   - When processing data, capture all context needed for downstream operations
   - Prefer structures that eliminate the need for data merging or complex lookups
   
   ```go
   // Good: Self-contained structure with all needed data
   type ProcessingResult struct {
       ID        string  // Captured at creation
       ToolName  string  // Captured at creation
       ToolInput string  // Captured at creation
       Output    string
       Error     error
   }
   
   func executeAndCapture(toolCall ToolCall) ProcessingResult {
       var result string
       var err error
       // ... execution logic ...
       
       return ProcessingResult{
           ID:        toolCall.ID,
           ToolName:  toolCall.Name,
           ToolInput: toolCall.Arguments,
           Output:    result,
           Error:     err,
       }
   }
   
   // No merging needed - all data is in one place
   func convertToModels(results []ProcessingResult) []*Model {
       models := make([]*Model, len(results))
       for i, result := range results {
           models[i] = &Model{
               Name:   result.ToolName,
               Input:  result.ToolInput,
               Output: result.Output,
               Error:  result.Error,
           }
       }
       return models
   }
   ```

7. **Designing for Testability**:
   - Avoid tight coupling to third-party SDK types that are difficult to construct in tests
   - Create intermediate data structures that capture only the data you need

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theimaginaryfoundation/what-iff](https://github.com/theimaginaryfoundation/what-iff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
