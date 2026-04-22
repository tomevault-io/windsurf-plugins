---
trigger: always_on
description: Security patterns, vulnerabilities, and protection strategies
---


# Security Patterns & Protection

> **Instructions**: Document security requirements, patterns, and vulnerabilities specific to your project.

## Authentication & Authorization

### Authentication Patterns
- **Primary Auth**: [e.g., Supabase Auth, Auth0, custom JWT]
- **Multi-Factor**: Required for admin functions and sensitive operations
- **Session Management**: [e.g., 24-hour expiry, refresh tokens, secure cookies]
- **Password Policy**: [e.g., 12+ chars, complexity requirements, breach checking]

### Authorization Patterns
- **Role-Based Access Control (RBAC)**: [Define roles and permissions]
- **Resource-Level Permissions**: [Who can access what data]
- **API Endpoint Protection**: [Authentication required for all endpoints]
- **Admin Functions**: [Two-person approval for critical operations]

### Example Implementation
```typescript
// Authentication middleware
export function requireAuth(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) return res.status(401).json({ error: 'Authentication required' });
  
  // Verify JWT and extract user context
  const user = verifyToken(token);
  req.user = user;
  next();
}

// Authorization middleware
export function requireRole(role: string) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user?.roles?.includes(role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}
```

## Input Validation & Sanitization

### Validation Rules
- **All user inputs** must be validated on both client and server
- **SQL injection prevention**: Use parameterized queries only
- **XSS prevention**: Sanitize all user-generated content
- **File uploads**: Validate file types, scan for malware, limit size
- **API rate limiting**: Prevent abuse and DoS attacks

### Validation Patterns
```typescript
// Input validation with Zod
const userSchema = z.object({
  email: z.string().email().max(255),
  password: z.string().min(12).regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/),
  role: z.enum(['user', 'admin', 'moderator'])
});

// Sanitization
function sanitizeHtml(input: string): string {
  return DOMPurify.sanitize(input, { 
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong'],
    ALLOWED_ATTR: []
  });
}
```

## Data Protection

### Encryption Requirements
- **Data at Rest**: [e.g., AES-256 encryption for sensitive data]
- **Data in Transit**: [e.g., TLS 1.3 for all communications]
- **Database Encryption**: [e.g., Transparent Data Encryption (TDE)]
- **Key Management**: [e.g., AWS KMS, Azure Key Vault, or similar]

### Sensitive Data Handling
- **PII Classification**: [Define what constitutes PII in your domain]
- **Data Minimization**: [Collect only necessary data]
- **Data Retention**: [Define retention periods for different data types]
- **Data Anonymization**: [Remove identifiers for analytics]

### Example Data Protection
```typescript
// Encrypt sensitive data before storage
async function encryptSensitiveData(data: any): Promise<string> {
  const key = await getEncryptionKey();
  return await encrypt(JSON.stringify(data), key);
}

// Hash passwords with salt
async function hashPassword(password: string): Promise<string> {
  const salt = await bcrypt.genSalt(12);
  return await bcrypt.hash(password, salt);
}
```

## API Security

### API Protection Patterns
- **HTTPS Only**: All API endpoints must use HTTPS
- **CORS Configuration**: [Define allowed origins and methods]
- **API Versioning**: [Version your APIs for security updates]
- **Request Validation**: [Validate all request parameters and body]
- **Response Sanitization**: [Remove sensitive data from responses]

### API Security Headers
```typescript
// Security headers middleware
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  next();
});
```

## Vulnerability Management

### Common Vulnerabilities to Prevent
- **OWASP Top 10**: [Address all OWASP vulnerabilities]
- **Dependency Vulnerabilities**: [Regular security scanning]
- **Code Injection**: [Prevent all forms of code injection]
- **Broken Authentication**: [Secure session management]
- **Sensitive Data Exposure**: [Encrypt and protect sensitive data]

### Security Testing
- **Static Analysis**: [Use tools like ESLint security plugins]
- **Dependency Scanning**: [npm audit, Snyk, etc.]
- **Penetration Testing**: [Regular security assessments]
- **Code Review**: [Security-focused code reviews]

### Example Security Checks
```typescript
// Prevent NoSQL injection
function sanitizeMongoQuery(query: any): any {
  // Remove potentially dangerous operators
  const dangerousOps = ['$where', '$regex', '$ne', '$gt', '$lt'];
  return Object.keys(query).reduce((clean, key) => {
    if (!dangerousOps.includes(key)) {
      clean[key] = query[key];
    }
    return clean;
  }, {});
}

// Validate file uploads
function validateFileUpload(file: Express.Multer.File): boolean {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/harrysayers7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
