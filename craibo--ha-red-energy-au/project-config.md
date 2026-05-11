---
trigger: always_on
description: Developer reference for implementing and maintaining OAuth2 PKCE authentication with Red Energy's Okta-based API. This document provides implementation details, code references, and patterns for working with the authentication system.
---

# Red Energy Authentication Reference

## Purpose

Developer reference for implementing and maintaining OAuth2 PKCE authentication with Red Energy's Okta-based API. This document provides implementation details, code references, and patterns for working with the authentication system.

**Use this reference when:**
- Implementing authentication flows
- Debugging authentication issues
- Understanding token lifecycle management
- Modifying credential handling
- Implementing error recovery

## Authentication Architecture

### Flow Overview

The Red Energy API uses a 5-step OAuth2 PKCE (Proof Key for Code Exchange) authentication flow:

```
1. Username/Password → Okta Session Token
2. Session Token → OAuth2 Authorization URL (with PKCE challenge)
3. Authorization Redirect → Extract Authorization Code
4. Authorization Code + PKCE Verifier → Access/Refresh Tokens
5. Access Token → API Calls (Bearer Authentication)
```

### Implementation Location

**Primary Implementation**: `custom_components/red_energy/api.py`

```python
class RedEnergyAPI:
    """Main authentication flow in authenticate() method (lines 46-83)"""
```

### State Management

Authentication state is managed through instance variables in `RedEnergyAPI`:

```python
self._access_token: Optional[str]      # Bearer token for API calls
self._refresh_token: Optional[str]     # Token for refreshing access
self._token_expires: Optional[datetime] # Expiration timestamp
```

**Lines**: 41-43 in `api.py`

## Authentication Steps - Implementation Details

### Step 1: Okta Session Token

**Method**: `_get_session_token(username: str, password: str) -> tuple[str, str]`  
**Lines**: 104-146 in `api.py`

**Implementation**:
```python
# POST to Okta with username/password
payload = {
    "username": username,
    "password": password,
    "options": {
        "warnBeforePasswordExpired": False,
        "multiOptionalFactorEnroll": False
    }
}
# Returns: (session_token, expires_at)
```

**Endpoint**: `https://redenergy.okta.com/api/v1/authn`  
**Constant**: `RedEnergyAPI.OKTA_AUTH_URL` (line 36)

**Error Handling**:
- HTTP != 200: Parse Okta error response, raise `RedEnergyAuthError`
- Status != "SUCCESS": Handle MFA/locked account scenarios
- All errors logged with full context for debugging

### Step 2: OAuth2 Discovery

**Method**: `_get_discovery_data() -> Dict[str, Any]`  
**Lines**: 85-90 in `api.py`

**Endpoint**: `https://login.redenergy.com.au/oauth2/default/.well-known/openid-configuration`  
**Constant**: `RedEnergyAPI.DISCOVERY_URL` (line 33)

**Returns**:
- `authorization_endpoint`: URL for authorization code request
- `token_endpoint`: URL for token exchange

### Step 3: PKCE Parameters

**Code Verifier Generation**:  
**Method**: `_generate_code_verifier() -> str`  
**Lines**: 92-97 in `api.py`

```python
# Generates 48-character random string
# Character set: [a-zA-Z0-9\-\.\_\~] per RFC 7636
alphabet = string.ascii_letters + string.digits + '-._~'
return ''.join(secrets.choice(alphabet) for _ in range(48))
```

**Code Challenge Generation**:  
**Method**: `_generate_code_challenge(verifier: str) -> str`  
**Lines**: 99-102 in `api.py`

```python
# SHA256 hash of verifier, base64url encoded
digest = hashlib.sha256(verifier.encode()).digest()
return base64.urlsafe_b64encode(digest).decode().rstrip('=')
```

### Step 4: Authorization Code Retrieval

**Method**: `_get_authorization_code(...) -> str`  
**Lines**: 148-223 in `api.py` (approximate)

**Process**:
1. Build authorization URL with session token, client_id, PKCE challenge
2. Follow redirects to capture authorization code
3. Parse code from redirect URL query parameters

**Redirect URI**: `au.com.redenergy://callback`  
**Constant**: `RedEnergyAPI.REDIRECT_URI` (line 34)

### Step 5: Token Exchange

**Method**: `_exchange_code_for_tokens(...) -> None`  
**Lines**: 225-259 in `api.py` (approximate)

**Token Exchange Parameters**:
```python
{
    'grant_type': 'authorization_code',
    'code': auth_code,
    'redirect_uri': REDIRECT_URI,
    'client_id': client_id,
    'code_verifier': code_verifier  # PKCE verifier
}
```

**Sets State Variables**:
- `self._access_token` - Used for API authentication
- `self._refresh_token` - Used for token refresh
- `self._token_expires` - Calculated from `expires_in` (default 3600s)

## Token Lifecycle Management

### Token Expiration

**Default Expiration**: 1 hour (3600 seconds)

**Expiration Check**: Before every API call  
**Method**: `_ensure_authenticated() -> None`  
**Lines**: 370-380 in `api.py` (approximate)

```python
if self._token_expires and datetime.now() >= self._token_expires:
    if self._refresh_token:
        await self._refresh_access_token()
    else:
        raise RedEnergyAuthError("Token expired and no refresh token available")
```

### Token Refresh

**Method**: `_refresh_access_token() -> None`  
**Lines**: 382-415 in `api.py`

**Process**:
1. Get token endpoint from discovery URL
2. POST with `grant_type=refresh_token` and refresh token
3. Update `_access_token`, `_refresh_token`, and `_token_expires`

**Refresh Parameters**:
```python
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [craibo/ha-red-energy-au](https://github.com/craibo/ha-red-energy-au) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
