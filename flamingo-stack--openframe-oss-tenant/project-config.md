---
trigger: always_on
description: description: Authentication patterns and security practices for OpenFrame JWT cookie-based system
---


---
description: Authentication patterns and security practices for OpenFrame JWT cookie-based system
globs:
  - "openframe/services/*/src/main/java/**/security/**"
  - "openframe/services/*/src/main/java/**/controller/**"
  - "openframe/services/*/src/main/java/**/service/**"
  - "openframe/libs/openframe-jwt/**"
  - "openframe/services/openframe-frontend/src/stores/auth.ts"
alwaysApply: false
---

# Authentication Patterns in OpenFrame

OpenFrame uses a secure, cookie-based JWT authentication system with Spring Security OAuth2 Resource Server. Follow these patterns for consistent authentication implementation.

## Core Architecture Components

### JWT + HttpOnly Cookies Pattern
- **Access tokens**: Stored in `access_token` HttpOnly cookie with `Path=/`
- **Refresh tokens**: Stored in `refresh_token` HttpOnly cookie with `Path=/api/oauth/token`
- **Security**: Tokens are never exposed to client-side JavaScript
- **Reference**: [CookieService.java](mdc:openframe/libs/openframe-jwt/src/main/java/com/openframe/security/cookie/CookieService.java)

### Spring Security Configuration
Always use Spring Security OAuth2 Resource Server in API services:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http, JwtDecoder jwtDecoder) {
        return http
            .csrf(AbstractHttpConfigurer::disable)
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/oauth/token", "/oauth/register", "/.well-known/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.decoder(jwtDecoder))
            )
            .build();
    }
}
```

**Reference**: [SecurityConfig.java](mdc:openframe/services/openframe-api/src/main/java/com/openframe/api/config/SecurityConfig.java)

## Controller Patterns

### Use AuthPrincipal Instead of Raw JWT
Always use `@AuthenticationPrincipal AuthPrincipal principal` in controllers:

```java
@RestController
public class ApiController {
    @GetMapping("/api-keys")
    public List<ApiKeyResponse> getApiKeys(@AuthenticationPrincipal AuthPrincipal principal) {
        return apiKeyService.getApiKeysForUser(principal.getId());
    }
}
```

**Never use**:
- `@RequestHeader("X-User-Id") String userId`
- `@AuthenticationPrincipal Jwt jwt` directly

**Reference**: [AuthPrincipal.java](mdc:openframe/libs/openframe-jwt/src/main/java/com/openframe/security/authentication/AuthPrincipal.java)

### OAuth Controller Pattern
OAuth controllers should delegate cookie management to services:

```java
@PostMapping("/token")
public ResponseEntity<?> token(
        @RequestParam String grant_type,
        @RequestParam(required = false) String code,
        @RequestHeader(value = X_REFRESH_TOKEN, required = false) String refreshToken,
        HttpServletRequest httpRequest,
        HttpServletResponse httpResponse) {

    TokenResponse response = oauthService.processTokenRequest(
        grant_type, code, username, password, client_id, client_secret, refreshToken, httpRequest);

    oauthService.setAuthenticationCookies(response, httpResponse);
    return ResponseEntity.ok(response);
}
```

**Reference**: [OAuthController.java](mdc:openframe/services/openframe-api/src/main/java/com/openframe/api/controller/OAuthController.java)

## Service Layer Patterns

### Cookie Management
Always delegate cookie operations to `CookieService`:

```java
@Service
public class OAuthService {
    private final CookieService cookieService;

    public void setAuthenticationCookies(TokenResponse tokens, HttpServletResponse response) {
        cookieService.setAccessTokenCookie(tokens.getAccess_token(), response);
        cookieService.setRefreshTokenCookie(tokens.getRefresh_token(), response);
    }
}
```

**Reference**: [OAuthService.java](mdc:openframe/services/openframe-api/src/main/java/com/openframe/api/service/OAuthService.java)

### Token Processing Pattern
Separate refresh token handling from other grant types:

```java
public TokenResponse processTokenRequest(String grantType, String refreshToken, ...) {
    if ("refresh_token".equals(grantType)) {
        if (refreshToken == null) {
            throw new IllegalArgumentException("Refresh token not found");
        }
        return handleRefreshToken(refreshToken, clientId, clientSecret);
    }

    return token(grantType, code, username, password, clientId, clientSecret);
}
```

## Gateway Security Patterns

### Cookie-to-Header Filter
Use `CookieToHeaderFilter` to convert cookies to headers for Spring Security:

```java
@Component
public class CookieToHeaderFilter implements WebFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        String accessToken = cookieService.getAccessTokenFromCookies(exchange);
        if (accessToken != null) {
            ServerHttpRequest request = exchange.getRequest().mutate()
                .header(AUTHORIZATION, "Bearer " + accessToken)
                .build();
            return chain.filter(exchange.mutate().request(request).build());
        }
        return chain.filter(exchange);
    }
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-tenant](https://github.com/flamingo-stack/openframe-oss-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
