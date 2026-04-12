---
trigger: always_on
description: Error Handling and Log Patterns
---

# Error Handling & Logging Patterns

## Backend Error Handling

### Exception Hierarchy
Define a clear exception hierarchy for the application:

```python
from typing import Optional, Dict, Any

class PumpAnalyzerException(Exception):
    """Base exception for all application-specific errors."""

    def __init__(
        self,
        message: str,
        code: Optional[str] = None,
        details: Optional[Dict[str, Any]] = None
    ):
        self.message = message
        self.code = code
        self.details = details or {}
        super().__init__(self.message)

class ValidationError(PumpAnalyzerException):
    """Raised when input validation fails."""
    pass

class BusinessLogicError(PumpAnalyzerException):
    """Raised when business rules are violated."""
    pass

class ResourceNotFoundError(PumpAnalyzerException):
    """Raised when a requested resource doesn't exist."""
    pass

class RateLimitExceededError(PumpAnalyzerException):
    """Raised when rate limits are exceeded."""
    pass

class EngineError(PumpAnalyzerException):
    """Raised when the Pump analysis engine encounters an error."""
    pass

class DatabaseError(PumpAnalyzerException):
    """Raised when database operations fail."""
    pass
```

### FastAPI Exception Handlers
Create global exception handlers for consistent error responses:

```python
from fastapi import FastAPI, Request, HTTPException, status
from fastapi.responses import JSONResponse
import logging

logger = logging.getLogger(__name__)

app = FastAPI()

@app.exception_handler(ValidationError)
async def validation_error_handler(request: Request, exc: ValidationError):
    """Handle validation errors with 422 status."""
    logger.warning(f"Validation error: {exc.message}", extra={
        "path": request.url.path,
        "method": request.method,
        "code": exc.code,
        "details": exc.details
    })

    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={
            "error": {
                "type": "validation_error",
                "message": exc.message,
                "code": exc.code,
                "details": exc.details
            }
        }
    )

@app.exception_handler(ResourceNotFoundError)
async def not_found_error_handler(request: Request, exc: ResourceNotFoundError):
    """Handle resource not found errors with 404 status."""
    logger.info(f"Resource not found: {exc.message}", extra={
        "path": request.url.path,
        "method": request.method,
        "code": exc.code
    })

    return JSONResponse(
        status_code=status.HTTP_404_NOT_FOUND,
        content={
            "error": {
                "type": "not_found",
                "message": exc.message,
                "code": exc.code
            }
        }
    )

@app.exception_handler(RateLimitExceededError)
async def rate_limit_error_handler(request: Request, exc: RateLimitExceededError):
    """Handle rate limiting with 429 status."""
    logger.warning(f"Rate limit exceeded: {exc.message}", extra={
        "path": request.url.path,
        "method": request.method,
        "client_ip": request.client.host,
        "details": exc.details
    })

    return JSONResponse(
        status_code=status.HTTP_429_TOO_MANY_REQUESTS,
        content={
            "error": {
                "type": "rate_limit_exceeded",
                "message": exc.message,
                "code": exc.code
            }
        },
        headers={
            "Retry-After": str(exc.details.get("retry_after", 60)),
            "X-RateLimit-Limit": str(exc.details.get("limit", 100)),
            "X-RateLimit-Reset": str(exc.details.get("reset_time", ""))
        }
    )

@app.exception_handler(Exception)
async def general_exception_handler(request: Request, exc: Exception):
    """Catch-all handler for unexpected errors."""
    logger.exception(f"Unhandled exception: {str(exc)}", extra={
        "path": request.url.path,
        "method": request.method,
        "client_ip": request.client.host,
        "exception_type": type(exc).__name__
    })

    # Don't expose internal error details in production
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={
            "error": {
                "type": "internal_error",
                "message": "An internal error occurred",
                "code": "INTERNAL_ERROR"
            }
        }
    )
```

### Structured Error Handling in Routers
Use consistent error handling patterns in route handlers:

```python
from sqlmodel import Session
from fastapi import Depends, HTTPException, status
import traceback

async def get_stream_with_error_handling(
    stream_id: UUID,
    session: Session = Depends(get_session)
) -> LiveStream:
    """Get stream with comprehensive error handling."""
    try:
        # Validate UUID format
        if not isinstance(stream_id, UUID):
            raise ValidationError(
                message="Invalid stream ID format",
                code="INVALID_UUID",
                details={"stream_id": str(stream_id)}
            )

        # Attempt to fetch from database
        stmt = select(LiveStream).where(LiveStream.id == stream_id)
        result = await session.exec(stmt)
        stream = result.first()

        if not stream:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MJE43) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
