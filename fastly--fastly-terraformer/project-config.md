---
trigger: always_on
description: fastly-terraformer is a specialized Go CLI tool that generates Terraform import blocks for Fastly Edge Cloud resources. It automatically discovers existing Fastly infrastructure and creates the necessary Terraform import statements to bring those resources under Terraform management.
---

# GitHub Copilot Instructions for fastly-terraformer

## Project Overview

fastly-terraformer is a specialized Go CLI tool that generates Terraform import blocks for Fastly Edge Cloud resources. It automatically discovers existing Fastly infrastructure and creates the necessary Terraform import statements to bring those resources under Terraform management.

### Key Features
- Comprehensive resource discovery for Fastly services, NGWAF resources, and more
- Two import modes: "all" (complete infrastructure) and "ngwaf" (NGWAF-only)
- Intelligent resource naming with sanitization for Terraform compatibility
- Workspace-scoped NGWAF support with conflict prevention
- Robust error handling and progress reporting

## Architecture

### Core Components

1. **Main Application** (`main.go`): ~3,772 lines containing:
   - CLI argument parsing and validation
   - Fastly API client initialization
   - Resource discovery orchestration
   - HCL import block generation

2. **Resource Import Functions**: Pattern-based functions for different resource types:
   - `importNGWAFWorkspaces()` - NGWAF workspace discovery
   - `importServices()` - Fastly service discovery
   - `importServiceDynamicSnippets()` - Service-specific resources
   - `importTLSSubscriptions()` - TLS/security resources
   - Various logging endpoint importers

3. **Utility Functions**:
   - `sanitizeForTerraformResourceName()` - Terraform-compatible naming
   - `generateWorkspacePrefixedResourceName()` - Workspace-scoped naming
   - `validateImportMode()` - Input validation

### Dependencies
- `github.com/fastly/go-fastly/v11` - Fastly API client
- `github.com/hashicorp/hcl/v2` - HCL generation
- `github.com/zclconf/go-cty` - Type system for HCL

## Development Guidelines

### Code Style and Patterns

#### Function Naming Convention
- Import functions: `import{ResourceType}()` (e.g., `importNGWAFWorkspaces`)
- Use descriptive, resource-specific function names
- Maintain consistent return patterns: `(count int, error)` or `(resource, count, error)`

#### Resource Naming Patterns
```go
// For regular resources
resourceName := sanitizeForTerraformResourceName(name, "fallback_prefix")

// For workspace-scoped resources (prevents conflicts)
resourceName := generateWorkspacePrefixedResourceName(
    workspaceID, resourceName, resourceID, "base_prefix"
)
```

#### Import Block Generation Pattern
```go
func importResourceType(client *fastly.Client, body *hclwrite.Body) (int, error) {
    resources, err := client.ListResourceType(&fastly.ListInput{})
    if err != nil {
        return 0, fmt.Errorf("error listing resources: %w", err)
    }

    count := 0
    for _, resource := range resources {
        if resource.ID == "" {
            continue // Skip resources with empty IDs
        }

        resourceName := sanitizeForTerraformResourceName(resource.Name, "resource")
        
        importBlock := body.AppendNewBlock("import", nil)
        importBlockBody := importBlock.Body()
        importBlockBody.SetAttributeValue("id", cty.StringVal(resource.ID))
        importBlockBody.SetAttributeValue("to", cty.StringVal(
            fmt.Sprintf("fastly_resource_type.%s", resourceName)
        ))
        
        count++
    }
    
    return count, nil
}
```

### Error Handling Principles

1. **Graceful Degradation**: Continue processing other resources if one fails
2. **Detailed Logging**: Log errors but don't stop the entire process
3. **Empty ID Validation**: Always check for empty/invalid resource IDs
4. **API Error Handling**: Wrap API errors with context

Example:
```go
resources, err := client.ListResources(&input)
if err != nil {
    fmt.Printf("Warning: Error listing resources: %v\n", err)
    return 0, nil // Don't fail the entire process
}

for _, resource := range resources {
    if resource.ID == "" {
        fmt.Printf("Skipping resource with empty ID: %+v\n", resource)
        continue
    }
    // Process resource...
}
```

### Testing Patterns

#### Function Signature Testing
```go
func TestImportFunction(t *testing.T) {
    // Verify function signature exists
    var fn func(*fastly.Client, *hclwrite.Body) (int, error)
    fn = importFunction
    if fn == nil {
        t.Error("importFunction should exist")
    }
}
```

#### Edge Case Testing
```go
func TestWithEmptyInput(t *testing.T) {
    count, err := importFunction(nil, nil)
    if count != 0 {
        t.Error("Expected 0 count with nil input")
    }
    if err != nil {
        t.Errorf("Expected no error with nil input, got %v", err)
    }
}
```

#### Name Sanitization Testing
```go
func TestSanitizeForTerraformResourceName(t *testing.T) {
    tests := []struct {
        name     string
        input    string
        prefix   string
        expected string
    }{
        {"normal_name", "My Resource", "resource", "my_resource"},
        {"empty_name", "", "resource", "resource_unnamed"},
        {"special_chars", "Resource-with@special#chars!", "resource", "resource_with_special_chars"},
    }
    
    for _, tt := range tests {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fastly/fastly-terraformer](https://github.com/fastly/fastly-terraformer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
