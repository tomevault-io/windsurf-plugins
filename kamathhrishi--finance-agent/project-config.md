---
trigger: always_on
description: - Be concise and direct in responses
---

# Cursor AI Rules for StrataLens AI Project

## General Principles

- Be concise and direct in responses
- Prioritize code quality and maintainability
- Follow existing patterns and conventions in the codebase
- Think before acting - understand the context fully

## Documentation Rules

### ❌ DO NOT Create Documentation Unless Explicitly Asked

- **NEVER** create README.md, CHANGELOG.md, or other documentation files automatically
- **NEVER** create summary documents for routine changes
- **NEVER** create migration guides unless specifically requested
- Only create documentation when the user explicitly asks: "create a readme", "document this", etc.
- Prefer inline code comments for complex logic instead of separate docs

### ✅ DO Create Documentation When

- User explicitly requests: "create documentation", "write a README", "document this API"
- Adding new major features that require setup instructions
- Creating public APIs that external developers will use

## Code Modification Rules

### Making Changes

- **ALWAYS** prefer editing existing files over creating new ones
- **NEVER** create helper scripts or temporary files unless absolutely necessary
- If temporary files are created, clean them up immediately after use
- Ask for clarification if requirements are ambiguous rather than guessing

### Code Style

- Follow existing code style and patterns in the file
- Use existing helper functions and utilities
- Don't add emojis to code unless explicitly requested
- Maintain consistent indentation (spaces vs tabs as per file)

## Python/FastAPI Specific

### Backend Development

- Use type hints for all function parameters and returns
- Leverage FastAPI's dependency injection (Depends)
- Use Pydantic models for request/response validation
- Handle errors with proper HTTP status codes
- Use async/await for database operations
- Log important events with appropriate log levels

### Database Operations

- Always use parameterized queries to prevent SQL injection
- Use connection pooling via `get_db()` dependency
- Handle database errors gracefully
- Use transactions for multi-step operations

### Authentication & Security

- Use `get_current_user` for protected endpoints
- Use `get_optional_user` for endpoints that work both authenticated and anonymous
- Never log sensitive information (passwords, tokens, etc.)
- Validate and sanitize all user inputs

## Frontend/JavaScript Specific

### JavaScript Code

- Use modern ES6+ syntax (const/let, arrow functions, async/await)
- Handle errors with try/catch blocks
- Show user-friendly error messages
- Use existing utility functions from utils.js
- Follow the existing DOM manipulation patterns

### API Calls

- Always handle both success and error cases
- Include proper headers (Content-Type, Authorization)
- Use async/await instead of raw promises
- Show loading states during API calls

## Testing & Deployment

### Before Changes

- Check for existing similar functionality before adding new code
- Search codebase for related patterns and follow them
- Consider backward compatibility

### After Changes

- Run linter and fix any errors introduced
- Test the specific functionality changed
- Verify no breaking changes for existing features
- Don't commit automatically unless explicitly asked

## File Organization

### Project Structure

```
stratalens_ai/
├── routers/          # API route handlers
├── models/           # Pydantic models
├── rag/              # RAG system and AI logic
├── frontend/         # HTML/CSS/JavaScript files
├── evaluation/       # Testing and evaluation
├── migrations/       # Database migrations
└── utils/            # Shared utilities
```

### Naming Conventions

- Python files: `snake_case.py`
- Classes: `PascalCase`
- Functions/variables: `snake_case`
- Constants: `UPPER_SNAKE_CASE`
- Frontend files: `kebab-case.html`, `camelCase.js`

## Common Patterns in This Codebase

### API Endpoints

```python
@router.post("/endpoint", response_model=ResponseModel)
async def endpoint_function(
    request_data: RequestModel,
    current_user: dict = Depends(get_current_user),
    db: asyncpg.Connection = Depends(get_db)
):
    """Clear docstring explaining what this endpoint does"""
    try:
        # Implementation
        return ResponseModel(success=True, data=result)
    except Exception as e:
        logger.error(f"Error in endpoint: {e}")
        raise HTTPException(status_code=500, detail=str(e))
```

### Database Queries

```python
# Fetch one
result = await db.fetchrow('SELECT * FROM table WHERE id = $1', id)

# Fetch many
results = await db.fetch('SELECT * FROM table WHERE condition = $1', value)

# Execute (INSERT/UPDATE/DELETE)
await db.execute('INSERT INTO table (col) VALUES ($1)', value)
```

### Frontend API Calls

```javascript
async function apiCall() {
    try {
        const response = await fetch(`${CONFIG.apiBaseUrl}/endpoint`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${localStorage.getItem('authToken')}`
            },
            body: JSON.stringify(data)
        });
        

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kamathhrishi/finance-agent](https://github.com/kamathhrishi/finance-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
