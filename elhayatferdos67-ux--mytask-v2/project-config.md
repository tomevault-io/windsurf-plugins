---
trigger: always_on
description: - Use Python 3.11+ features and type hints consistently
---


# Backend Development Guidelines

## Python Standards & Best Practices

### Language Features

- Use Python 3.11+ features and type hints consistently
- Follow PEP 8 style guidelines with black formatting
- Use async/await for all I/O operations
- Leverage dataclasses and Pydantic models for structure
- Use context managers for resource management

### Type Safety

- Comprehensive type hints for all functions and classes
- Use `typing` module for complex types (Union, Optional, List, Dict)
- Define custom types for domain concepts
- Use `typing.Protocol` for interface definitions

## FastAPI Architecture Patterns

### API Design

- Use dependency injection for database connections and services
- Implement proper request/response models with Pydantic v2
- Follow RESTful API design principles
- Use FastAPI's automatic OpenAPI documentation
- Implement proper HTTP status codes and error responses

### Route Organization

- Group related routes in separate modules
- Use APIRouter for modular route organization
- Implement consistent error handling middleware
- Use dependencies for authentication and authorization

### Example Patterns

```python
# Route with proper dependency injection
@router.post("/agents", response_model=AgentResponse)
async def create_agent(
    agent_data: AgentCreateRequest,
    db: DBConnection = Depends(get_db),
    user: UserClaims = Depends(get_current_user)
) -> AgentResponse:
    try:
        agent = await agent_service.create_agent(agent_data, user.id)
        return AgentResponse.from_orm(agent)
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
```

## Database Integration

### Supabase Patterns

- Use proper SQL migrations in `backend/supabase/migrations/`
- Follow established schema patterns with UUID primary keys
- Implement row-level security (RLS) for all user-accessible tables
- Use proper indexing for performance optimization

### Migration Best Practices

```sql
-- Idempotent migration pattern
BEGIN;

-- Create table with proper constraints
CREATE TABLE IF NOT EXISTS example_table (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),

    CONSTRAINT example_table_name_not_empty CHECK (LENGTH(TRIM(name)) > 0)
);

-- Create indexes for performance
CREATE INDEX IF NOT EXISTS idx_example_table_user_id ON example_table(user_id);
CREATE INDEX IF NOT EXISTS idx_example_table_created_at ON example_table(created_at);

-- Enable RLS
ALTER TABLE example_table ENABLE ROW LEVEL SECURITY;

-- Create RLS policies
CREATE POLICY "Users can manage their own records" ON example_table
    FOR ALL USING (auth.uid() = user_id);

-- Create trigger for updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ language 'plpgsql';

DROP TRIGGER IF EXISTS update_example_table_updated_at ON example_table;
CREATE TRIGGER update_example_table_updated_at
    BEFORE UPDATE ON example_table
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

COMMIT;
```

## Tool Development Framework

### Tool Base Classes

- Extend `AgentBuilderBaseTool` for agent builder tools
- Extend `Tool` for general agent tools
- Use proper inheritance patterns and method overrides

### Tool Schema Implementation

```python
class ExampleTool(AgentBuilderBaseTool):
    @openapi_schema({
        "type": "function",
        "function": {
            "name": "example_action",
            "description": "Perform an example action with detailed description",
            "parameters": {
                "type": "object",
                "properties": {
                    "param1": {
                        "type": "string",
                        "description": "First parameter with clear explanation"
                    },
                    "param2": {
                        "type": "integer",
                        "description": "Second parameter with default value",
                        "default": 0
                    }
                },
                "required": ["param1"]
            }
        }
    })
    async def example_action(self, param1: str, param2: int = 0) -> ToolResult:
        try:
            logger.debug(f"Executing example_action with params: {param1}, {param2}")

            # Implementation logic here
            result = await self.perform_action(param1, param2)

            return self.success_response(
                result=result,
                message=f"Successfully completed action for {param1}"
            )
        except Exception as e:
            logger.error(f"Tool execution failed: {e}", exc_info=True)
            return self.fail_response(f"Failed to perform action: {str(e)}")
```

### Tool Registration

- Use `AgentBuilderToolRegistry` pattern for registering tools
- Follow `MCPToolWrapper` patterns for external tool integration
- Use `DynamicToolBuilder` for runtime tool creation
- Implement proper tool discovery and validation

## LLM Integration & Agent System

### LiteLLM Usage

- Use LiteLLM for multi-provider support (Anthropic, OpenAI, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elhayatferdos67-ux/mytask.v2](https://github.com/elhayatferdos67-ux/mytask.v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
