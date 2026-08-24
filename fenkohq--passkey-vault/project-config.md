---
trigger: always_on
description: Passkey Vault is a sophisticated Chrome extension designed to securely store, manage, and retrieve passkeys (WebAuthn credentials) in a fully standalone manner. The extension operates invisibly in the background, providing automated passkey management with robust offline backup capabilities.
---

# Passkey Vault - Chrome Extension Agents

## Project Overview

Passkey Vault is a sophisticated Chrome extension designed to securely store, manage, and retrieve passkeys (WebAuthn credentials) in a fully standalone manner. The extension operates invisibly in the background, providing automated passkey management with robust offline backup capabilities.

## Core Agents

### 1. Storage Agent

**Purpose**: Manages encrypted storage of passkeys

**Responsibilities**:

- Encrypt passkey data using AES-256-GCM
- Store encrypted data in Chrome's local storage
- Manage storage quotas and cleanup
- Handle data integrity verification

**Technical Requirements**:

```typescript
interface StorageAgent {
  encryptAndStore(passkey: PasskeyData): Promise<void>;
  retrieveAndDecrypt(passkeyId: string): Promise<PasskeyData | null>;
  listStoredPasskeys(): Promise<PasskeyMetadata[]>;
  deletePasskey(passkeyId: string): Promise<void>;
  exportEncryptedBackup(): Promise<EncryptedBackup>;
  importEncryptedBackup(backup: EncryptedBackup): Promise<void>;
}
```

**Security Considerations**:

- Key derivation using PBKDF2 with random salt
- Separate encryption keys for each session
- Secure key storage in Chrome's storage API
- Memory cleanup after operations

### 2. WebAuthn Agent

**Purpose**: Interfaces with WebAuthn API for passkey operations

**Responsibilities**:

- Intercept WebAuthn navigator credentials requests
- Proxy authentication requests to stored passkeys
- Generate new passkey registrations
- Handle attestation and assertion flows

**Technical Requirements**:

```typescript
interface WebAuthnAgent {
  interceptCreateRequest(options: CredentialCreationOptions): Promise<Credential>;
  interceptGetRequest(options: CredentialRequestOptions): Promise<Credential>;
  generateNewPasskey(
    rp: PublicKeyCredentialRpEntity,
    user: PublicKeyCredentialUserEntity
  ): Promise<void>;
  validatePasskeyOwnership(assertion: PublicKeyCredential): Promise<boolean>;
}
```

**Implementation Details**:

- Content script injection for WebAuthn API interception
- Background script coordination for secure operations
- Compatibility with FIDO2/WebAuthn standards
- Support for various authenticator formats

### 3. UI Agent (Hidden)

**Purpose**: Manages the invisible user interface for emergency access

**Responsibilities**:

- Handle activation sequences (konami codes or specific patterns)
- Display emergency interface when triggered
- Manage master password input
- Provide backup/restore functionality

**Technical Requirements**:

```typescript
interface UIAgent {
  setupActivationListener(): void;
  showEmergencyInterface(): Promise<void>;
  hideEmergencyInterface(): void;
  handleMasterPassword(password: string): Promise<boolean>;
}
```

**Activation Methods**:

- Keyboard sequence detection
- Specific page URL patterns
- Browser action combinations
- Time-based activation windows

### 4. Backup Agent

**Purpose**: Manages offline backup creation and restoration

**Responsibilities**:

- Generate encrypted backup files
- Create backup schedules
- Validate backup integrity
- Handle backup restoration

**Technical Requirements**:

```typescript
interface BackupAgent {
  createBackup(password: string): Promise<BackupFile>;
  scheduleBackup(interval: number): void;
  validateBackup(backup: BackupFile): Promise<boolean>;
  restoreBackup(backup: BackupFile, password: string): Promise<void>;
  exportBackupToFile(): Promise<Blob>;
  importBackupFromFile(file: File): Promise<void>;
}
```

**Backup Formats**:

- JSON with encrypted payload
- QR code representation
- Text-based recovery codes
- Split knowledge backups

### 5. Security Agent

**Purpose**: Enforces security policies and monitors for threats

**Responsibilities**:

- Monitor for suspicious activities
- Implement rate limiting
- Detect potential compromise attempts
- Manage security policies

**Technical Requirements**:

```typescript
interface SecurityAgent {
  validateOperation(operation: SecurityOperation): Promise<boolean>;
  detectSuspiciousActivity(activity: ActivityEvent): Promise<void>;
  enforceRateLimit(operation: string): Promise<void>;
  auditLog(event: AuditEvent): void;
}
```

**Security Features**:

- Biometric verification when available
- Device fingerprinting
- Anomaly detection
- Automatic lockdown on threats

## Data Models

### PasskeyData

```typescript
interface PasskeyData {
  id: string;
  name: string;
  rpId: string;
  rpName: string;
  userId: string;
  userName: string;
  publicKey: string;
  privateKey: string;
  counter: number;
  createdAt: Date;
  lastUsed: Date;
  metadata: PasskeyMetadata;
}
```

### EncryptedBackup

```typescript
interface EncryptedBackup {
  version: string;
  algorithm: string;
  salt: string;
  iv: string;
  data: string; // Encrypted payload
  checksum: string;
  timestamp: number;
}
```

## Communication Protocols

### Internal Messaging

```typescript
interface ExtensionMessage {
  type: 'STORE_PASSKEY' | 'RETRIEVE_PASSKEY' | 'BACKUP' | 'RESTORE';
  payload: any;
  requestId: string;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FenkoHQ/passkey-vault](https://github.com/FenkoHQ/passkey-vault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
