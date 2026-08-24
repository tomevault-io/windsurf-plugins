---
trigger: always_on
description: External integration rules — providers, idempotency, bulkhead, observability
---


# External Integration Rules

Every external boundary in this codebase is shaped by 5 rules. Apply them every time you add or modify an integration.

## 1. Anti-Corruption Layer (ACL): provider returns `GenerateResult | ProviderError`

Provider adapters MUST translate vendor responses into our domain types. Never return `dict`, never let the vendor's field names cross the boundary.

```python
# app/providers/_types.py
from dataclasses import dataclass
from decimal import Decimal

@dataclass(frozen=True)
class GenerateResult:
    url: str
    cost_usd: Decimal
    latency_ms: int
    provider_request_id: str

class ProviderError(Exception):
    def __init__(self, message: str, *, retryable: bool, code: str | None = None):
        super().__init__(message); self.retryable = retryable; self.code = code

class ProviderTimeout(ProviderError):
    def __init__(self, message: str): super().__init__(message, retryable=True, code="timeout")

class ProviderQuotaExceeded(ProviderError):
    def __init__(self, message: str): super().__init__(message, retryable=False, code="quota")

class ProviderInvalidRequest(ProviderError):
    def __init__(self, message: str): super().__init__(message, retryable=False, code="invalid_request")
```

```python
# ✅ GOOD — adapter maps to ACL
class FalImageAdapter:
    def __init__(self, client: FalClient): self._c = client

    async def generate(self, req: ImageRequest) -> GenerateResult | ProviderError:
        try:
            data = await self._c.request("/v1/image", req.model_dump())
        except httpx.TimeoutException as e:
            return ProviderTimeout(str(e))
        except httpx.HTTPStatusError as e:
            if e.response.status_code == 429:
                return ProviderQuotaExceeded(e.response.text)
            return ProviderError(e.response.text, retryable=False, code=str(e.response.status_code))
        return GenerateResult(
            url=data["images"][0]["url"],
            cost_usd=Decimal(str(data["billing"]["cost_usd"])),
            latency_ms=int(data["meta"]["latency_ms"]),
            provider_request_id=data["request_id"],
        )
```

```python
# ❌ BAD — leaking the vendor response shape
async def generate(self, req: dict) -> dict:
    return await self._c.request("/v1/image", req)
```

## 2. Per-Provider Bulkhead: one `httpx.AsyncClient` per provider

Each external provider has its own `httpx.AsyncClient` instance, with its own `Limits` and `timeout`. Defined ONCE in `app/core/http.py`.

```python
# app/core/http.py
import httpx
from app.core.config import settings

FAL_HTTP = httpx.AsyncClient(
    base_url=settings.FAL_BASE_URL,
    timeout=httpx.Timeout(connect=5.0, read=60.0, write=10.0, pool=5.0),
    limits=httpx.Limits(max_connections=20, max_keepalive_connections=10),
    headers={"Authorization": f"Key {settings.FAL_API_KEY}"},
)

OPENAI_HTTP = httpx.AsyncClient(
    base_url="https://api.openai.com/v1",
    timeout=httpx.Timeout(connect=5.0, read=30.0, write=10.0, pool=5.0),
    limits=httpx.Limits(max_connections=50, max_keepalive_connections=20),
    headers={"Authorization": f"Bearer {settings.OPENAI_API_KEY}"},
)
```

**Rationale:** if Fal is hung, its 20 connections fill up — but OpenAI's 50 are untouched. A single global client with `max_connections=100` would let one slow provider freeze every integration.

```python
# ❌ BAD
HTTP = httpx.AsyncClient()  # shared across all providers — no bulkhead
```

## 3. Idempotency Keys

Every side-effect operation accepts an idempotency key (UUID v4). For external API calls:

- If the provider supports an `Idempotency-Key` header, forward it.
- If not, persist `(user_id, key) → provider_request_id` in `provider_logs`. Look up before retrying.

```python
class FalImageAdapter:
    async def generate(self, req: ImageRequest, *, idempotency_key: UUID) -> GenerateResult | ProviderError:
        existing = await self._logs.find(user_id=req.user_id, key=idempotency_key)
        if existing is not None:
            return GenerateResult(**existing.cached_result)

        result_or_err = await self._call(req, header_key=str(idempotency_key))

        if isinstance(result_or_err, GenerateResult):
            await self._logs.persist(
                user_id=req.user_id, key=idempotency_key,
                provider_request_id=result_or_err.provider_request_id,
                cached_result=result_or_err.__dict__,
            )
        return result_or_err
```

Routers accept the key from a header:

```python
@router.post("/generate/image", response_model=GenerateResponse)
async def generate_image(
    req: GenerateRequest,
    user_id: str = Depends(get_current_user_id),
    idempotency_key: UUID = Header(default_factory=uuid4, alias="Idempotency-Key"),
    svc: ImageGenerationService = Depends(get_image_generation_service),
) -> GenerateResponse:
    return GenerateResponse.from_domain(await svc.generate(user_id, req, idempotency_key))
```

## 4. Observability Context (`contextvars`)

Use `contextvars.ContextVar` to thread `provider`, `user_id`, `request_id` through async call stacks. The JSON formatter reads them automatically.

```python
# app/core/logging.py
import contextvars, json, logging

provider_var   = contextvars.ContextVar[str | None]("provider", default=None)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yerdaulet-damir/vibe-coding-rules](https://github.com/yerdaulet-damir/vibe-coding-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
