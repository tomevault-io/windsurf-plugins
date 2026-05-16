---
trigger: always_on
description: ENFORCE security standards when handling authentication or sensitive data
---


# Security and Authentication Patterns

## Security Architecture Overview

SYMindX implements defense-in-depth security patterns across all layers of the agent framework, with particular focus on AI provider API key management, multi-platform authentication, and secure inter-agent communication.

### Core Security Principles

**🔐 Zero Trust Architecture**

- All components authenticate and authorize every request
- No implicit trust between internal services
- Continuous verification of security posture

**🛡️ Least Privilege Access**

- Agents receive minimum permissions needed for their role
- Platform extensions isolated with specific scopes
- API keys restricted to required endpoints only

**🔄 Security by Design**

- Security controls integrated into development workflow
- Automated security testing in CI/CD pipeline
- Regular security audits and penetration testing

## API Key Management

### Secure Storage Patterns

```typescript
// Environment-based configuration (development)
interface SecureConfig {
  vault: {
    enabled: boolean;
    url: string;
    token: string;
  };
  encryption: {
    algorithm: 'aes-256-gcm';
    keyDerivation: 'pbkdf2';
    saltRounds: 100000;
  };
}

// Production vault integration
class SecureConfigManager {
  private vault: VaultClient;
  private cache: Map<string, EncryptedValue> = new Map();
  
  async getAPIKey(provider: string): Promise<string> {
    const cacheKey = `api_key_${provider}`;
    
    // Check cache first (with TTL)
    if (this.cache.has(cacheKey)) {
      const cached = this.cache.get(cacheKey)!;
      if (cached.expiresAt > Date.now()) {
        return this.decrypt(cached.value);
      }
    }
    
    // Fetch from vault
    const secret = await this.vault.read(`secret/ai-providers/${provider}`);
    const encrypted = this.encrypt(secret.data.api_key);
    
    // Cache with 1-hour TTL
    this.cache.set(cacheKey, {
      value: encrypted,
      expiresAt: Date.now() + 3600000
    });
    
    return secret.data.api_key;
  }
}
```

### Key Rotation Strategies

```typescript
interface KeyRotationConfig {
  rotationSchedule: {
    openai: '30d';      // Rotate every 30 days
    anthropic: '30d';
    groq: '60d';
    localProviders: 'never';
  };
  gracePeriod: '24h';   // Keep old keys valid during rotation
  notification: {
    webhooks: string[];
    email: string[];
  };
}

class KeyRotationManager {
  async rotateProviderKey(provider: string): Promise<void> {
    const oldKey = await this.configManager.getAPIKey(provider);
    
    // Generate new key via provider API
    const newKey = await this.generateNewKey(provider);
    
    // Store new key in vault
    await this.vault.write(`secret/ai-providers/${provider}`, {
      api_key: newKey,
      previous_key: oldKey,
      rotated_at: new Date().toISOString()
    });
    
    // Update all active agents gradually
    await this.updateAgentConfigurations(provider, newKey);
    
    // Schedule old key deletion after grace period
    setTimeout(() => {
      this.revokeOldKey(provider, oldKey);
    }, this.parseTimespan(this.config.gracePeriod));
  }
}
```

## Authentication Flows

### Agent Authentication

```typescript
interface AgentIdentity {
  agentId: string;
  characterId: string;
  platform: string;
  permissions: Permission[];
  sessionToken: string;
  expiresAt: Date;
}

class AgentAuthenticator {
  async authenticateAgent(credentials: AgentCredentials): Promise<AgentIdentity> {
    // Validate agent exists and is active
    const agent = await this.agentRegistry.getAgent(credentials.agentId);
    if (!agent || agent.status === 'disabled') {
      throw new UnauthorizedError('Agent not found or disabled');
    }
    
    // Verify character permissions
    const character = await this.characterRegistry.getCharacter(agent.characterId);
    const permissions = this.calculatePermissions(character, credentials.platform);
    
    // Generate session token with platform-specific scope
    const sessionToken = await this.tokenManager.generateSessionToken({
      agentId: credentials.agentId,
      platform: credentials.platform,
      permissions,
      expiresIn: '24h'
    });
    
    return {
      agentId: credentials.agentId,
      characterId: agent.characterId,
      platform: credentials.platform,
      permissions,
      sessionToken,
      expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000)
    };
  }
}
```

### Platform Extension Authentication

```typescript
// Platform-specific authentication patterns
abstract class PlatformAuthenticator {
  abstract authenticate(credentials: PlatformCredentials): Promise<PlatformSession>;
  abstract refreshToken(session: PlatformSession): Promise<PlatformSession>;
  abstract validatePermissions(session: PlatformSession, action: string): boolean;
}

// Telegram authentication
class TelegramAuthenticator extends PlatformAuthenticator {
  async authenticate(credentials: TelegramCredentials): Promise<TelegramSession> {
    // Validate bot token with Telegram API
    const botInfo = await this.telegram.getMe(credentials.botToken);
    
    // Verify webhook signature for incoming updates
    const isValidWebhook = this.verifyWebhookSignature(
      credentials.webhookData,
      credentials.secretToken
    );
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
