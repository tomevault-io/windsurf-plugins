---
trigger: always_on
description: **Rule Priority:** Critical Security
---

# AI Safety and Responsible AI Patterns 2025

**Rule Priority:** Critical Security  
**Activation:** AI Integration Development  
**Scope:** All AI provider integrations and agent systems

## 2025 AI Safety Standards

### Mandatory Safety Checks

```typescript
// REQUIRED: AI safety validation layer
export interface AISafetyGuards {
  readonly contentFilter: ContentFilter;
  readonly rateLimit: RateLimit;
  readonly auditLogger: AuditLogger;
  readonly privacyFilter: PrivacyFilter;
  readonly biasDetector: BiasDetector;
  readonly hallucinationDetector: HallucinationDetector;
}

export class SafetyValidatedAIPortal implements AIPortal {
  constructor(
    private readonly provider: AIProvider,
    private readonly safetyGuards: AISafetyGuards
  ) {}

  async generateResponse(request: AIRequest): Promise<SafeAIResponse> {
    // Pre-processing safety checks
    const sanitizedRequest = await this.safetyGuards.contentFilter.sanitize(request);
    await this.safetyGuards.rateLimit.checkLimit(request.userId);
    await this.safetyGuards.privacyFilter.validateRequest(sanitizedRequest);

    // Generate response with monitoring
    const response = await this.provider.generate(sanitizedRequest);

    // Post-processing validation
    const validatedResponse = await this.safetyGuards.hallucinationDetector.validate(response);
    await this.safetyGuards.biasDetector.analyze(validatedResponse);
    await this.safetyGuards.auditLogger.log({
      request: sanitizedRequest,
      response: validatedResponse,
      timestamp: new Date(),
      safety: 'validated'
    });

    return validatedResponse;
  }
}
```

### Content Filtering and Moderation

- **Implement real-time content scanning** for harmful, illegal, or inappropriate content
- **Use multiple moderation layers** with cascading filters
- **Log all filtered content** for analysis and improvement
- **Provide clear rejection reasons** for filtered requests

```typescript
// GOOD: Multi-layer content filtering
export class ContentFilter {
  private readonly filters: ContentFilterLayer[] = [
    new ProfanityFilter(),
    new ViolenceFilter(),
    new PrivacyFilter(),
    new LegalComplianceFilter(),
    new BiasFilter()
  ];

  async sanitize(content: string): Promise<ContentFilterResult> {
    const violations: ContentViolation[] = [];
    let filteredContent = content;

    for (const filter of this.filters) {
      const result = await filter.process(filteredContent);
      if (result.violations.length > 0) {
        violations.push(...result.violations);
        filteredContent = result.sanitized;
      }
    }

    return {
      original: content,
      sanitized: filteredContent,
      violations,
      safe: violations.length === 0
    };
  }
}
```

## Privacy Protection Patterns

### Data Minimization and Anonymization

```typescript
// REQUIRED: Privacy-first data handling
export class PrivacyProtectedProcessor {
  private readonly anonymizer = new DataAnonymizer();
  private readonly retention = new DataRetentionManager();

  async processUserData(data: UserData): Promise<AnonymizedData> {
    // Remove or hash PII
    const anonymized = await this.anonymizer.anonymize(data, {
      removeEmails: true,
      hashPhoneNumbers: true,
      removeAddresses: true,
      generalizeLocations: true
    });

    // Set retention policy
    await this.retention.setPolicy(anonymized.id, {
      type: 'ai-processing',
      maxAge: '30d',
      autoDelete: true
    });

    return anonymized;
  }

  async getDataForAI(userId: string): Promise<AICompatibleData> {
    const userData = await this.getUserData(userId);
    const anonymized = await this.processUserData(userData);
    
    return {
      context: anonymized.context,
      preferences: anonymized.preferences,
      // Never include: real names, emails, addresses, phone numbers
      metadata: {
        region: anonymized.generalLocation,
        sessionId: crypto.randomUUID(),
        timestamp: new Date().toISOString()
      }
    };
  }
}
```

### GDPR and Privacy Compliance

- **Implement explicit consent** for AI processing
- **Provide data deletion capabilities** (right to be forgotten)
- **Enable data portability** for user data
- **Maintain detailed audit logs** for compliance verification

```typescript
// GOOD: GDPR-compliant AI data handling
export class GDPRCompliantAIService {
  async processWithConsent(
    userId: string, 
    data: PersonalData, 
    consent: ConsentRecord
  ): Promise<ProcessingResult> {
    // Verify explicit consent
    if (!consent.aiProcessing || consent.expired) {
      throw new ConsentError('Valid AI processing consent required');
    }

    // Log consent usage
    await this.auditLogger.logConsentUsage({
      userId,
      consentId: consent.id,
      purpose: 'ai-processing',
      timestamp: new Date()
    });

    // Process with privacy protection
    return await this.privacyProtectedProcessor.process(data);
  }

  async deleteUserData(userId: string): Promise<DeletionResult> {
    // Remove all stored data
    await this.dataStore.deleteUserData(userId);
    await this.aiCache.clearUserCache(userId);
    await this.auditLogger.logDeletion(userId);
    
    return { deleted: true, timestamp: new Date() };
  }
}
```

## Bias Detection and Mitigation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
