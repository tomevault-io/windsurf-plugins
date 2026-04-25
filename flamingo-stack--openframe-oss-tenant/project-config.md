---
trigger: always_on
description: This document outlines the SSO (Single Sign-On) authentication patterns and best practices for the OpenFrame project.
---

# SSO Authentication in OpenFrame

This document outlines the SSO (Single Sign-On) authentication patterns and best practices for the OpenFrame project.

## Architecture Overview

OpenFrame implements OAuth 2.0 with PKCE (Proof Key for Code Exchange) for secure authentication with external providers like Google, Microsoft, and Slack.

### Key Components

- **SSOConfigService**: Manages SSO provider configurations
- **SocialAuthService**: Handles OAuth authentication flows
- **GoogleAuthStrategy**: Provider-specific authentication implementation
- **EncryptionService**: Encrypts/decrypts sensitive configuration data
- **Frontend SSOService**: Client-side OAuth flow management

## Backend Patterns

### SSO Configuration Management

Use the domain-driven approach for SSO configuration:

```java
@Service
public class SSOConfigService {
    private final SSOConfigRepository ssoConfigRepository;
    private final EncryptionService encryptionService;
    
    /**
     * Get SSO configuration domain model for internal service usage
     */
    public Optional<SSOConfig> getConfigByProvider(String provider) {
        return ssoConfigRepository.findByProvider(provider);
    }
    
    /**
     * Get list of enabled SSO providers - used by login components
     */
    public List<SSOConfigStatusResponse> getEnabledProviders() {
        return ssoConfigRepository.findByEnabledTrue().stream()
            .map(config -> SSOConfigStatusResponse.builder()
                .provider(config.getProvider())
                .enabled(true)
                .clientId(config.getClientId())
                .build())
            .collect(Collectors.toList());
    }
    
    /**
     * Get available SSO providers for admin dropdowns
     */
    public List<SSOProviderInfo> getAvailableProviders() {
        return authStrategies.stream()
            .map(strategy -> SSOProviderInfo.builder()
                .provider(strategy.getProvider().getProvider())
                .displayName(strategy.getProvider().getDisplayName())
                .build())
            .collect(Collectors.toList());
    }
}
```

### Provider Strategy Pattern

Implement provider-specific strategies using the Strategy pattern:

```java
public interface SocialAuthStrategy {
    SSOProvider getProvider();
    
    @Deprecated
    default String getProviderName() {
        return getProvider().getProvider();
    }
    
    TokenResponse authenticate(SocialAuthRequest request);
}

@Component
public class GoogleAuthStrategy implements SocialAuthStrategy {
    private final SSOConfigRepository ssoConfigRepository;
    private final EncryptionService encryptionService;
    
    @Override
    public SSOProvider getProvider() {
        return SSOProvider.GOOGLE;
    }
    
    @Override
    public TokenResponse authenticate(SocialAuthRequest request) {
        SSOConfig googleConfig = getGoogleConfig();
        String clientSecret = encryptionService.decryptClientSecret(googleConfig.getClientSecret());
        
        // Implementation...
    }
    
    private SSOConfig getGoogleConfig() {
        return ssoConfigRepository.findByProvider(SSOProvider.GOOGLE.getProvider())
            .filter(SSOConfig::isEnabled)
            .orElseThrow(() -> new SocialAuthException("provider_not_configured",
                    "Google OAuth is not configured or disabled"));
    }
}
```

### Provider Enum

Use type-safe enums for provider management:

```java
public enum SSOProvider {
    GOOGLE("google", "Google OAuth"),
    MICROSOFT("microsoft", "Microsoft OAuth"),
    SLACK("slack", "Slack OAuth");
    
    private final String provider;
    private final String displayName;
    
    SSOProvider(String provider, String displayName) {
        this.provider = provider;
        this.displayName = displayName;
    }
    
    public static SSOProvider fromProvider(String provider) {
        return Arrays.stream(values())
            .filter(p -> p.provider.equals(provider))
            .findFirst()
            .orElse(null);
    }
    
    // Getters...
}
```

### API Endpoint Design

Follow RESTful patterns for SSO endpoints:

```java
@RestController
@RequestMapping("/sso")
public class SSOConfigController {
    
    /**
     * Get enabled providers for login buttons
     */
    @GetMapping("/providers")
    public List<SSOConfigStatusResponse> getEnabledProviders() {
        return ssoConfigService.getEnabledProviders();
    }
    
    /**
     * Get available providers for admin dropdowns
     */
    @GetMapping("/providers/available")
    public List<SSOProviderInfo> getAvailableProviders() {
        return ssoConfigService.getAvailableProviders();
    }
    
    /**
     * Get full SSO configuration for admin forms
     */
    @GetMapping("/{provider}")
    public ResponseEntity<SSOConfigResponse> getConfig(@PathVariable String provider) {
        // Implementation...
    }
    
    /**
     * Create or update SSO configuration
     */
    @PostMapping("/{provider}")
    public ResponseEntity<SSOConfigResponse> createConfig(
            @PathVariable String provider,
            @Valid @RequestBody SSOConfigRequest request) {
        // Implementation...
    }
}
```

## Frontend Patterns

### Service Layer


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/flamingo-stack) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
