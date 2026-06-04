---
trigger: always_on
description: This document provides a comprehensive overview of the entire Reclaim Protocol documentation, covering all SDKs, integration methods, and technical details found in the `/content/docs` directory.
---

# Reclaim Protocol Documentation - Complete Reference Guide

This document provides a comprehensive overview of the entire Reclaim Protocol documentation, covering all SDKs, integration methods, and technical details found in the `/content/docs` directory.

## 🎯 Overview

Reclaim Protocol is a privacy-preserving identity and data verification protocol that enables users to prove facts about their online activity without revealing sensitive information. The protocol supports **2500+ data sources** and provides SDKs for web, mobile, and blockchain platforms.

### Key Features
- **Zero-Knowledge Proofs**: Verify data without exposing credentials
- **Cross-Platform SDKs**: Web, Mobile (iOS/Android/React Native/Flutter), and 15+ blockchains
- **Privacy-First**: End-to-end encryption with no credential storage
- **Seamless UX**: App Clips/Instant Apps - no installation required

## 🔑 Getting Started

### 1. Create Application
1. Register at [dev.reclaimprotocol.org](https://dev.reclaimprotocol.org)
2. Create new application with name, description, and logo
3. Copy `APP_ID` and `APP_SECRET` (⚠️ Secret shown only once)
4. Add providers to your application
5. Note down `PROVIDER_ID` for each provider

### 2. Core Concepts

#### **Proofs**
- Cryptographic evidence of user's online activity
- Generated through secure TLS communication
- Verifiable without revealing sensitive data

#### **Attestors**
- Opaque proxies between users and target servers
- Cannot decrypt user data (end-to-end TLS encryption)
- Verify and sign claims without accessing sensitive info

#### **Providers**
- Define what to verify and how
- Components: `loginUrl`, `requestData`, `responseMatches`, `responseRedactions`
- Create custom providers via DevTool

#### **Verifier Apps**
- Native mobile apps via App Clips (iOS) / Instant Apps (Android)
- No installation required - access via URL
- Handle proof generation seamlessly

## 🌐 Web SDK Integration

### Frontend (React/JS)

#### Method 1: `triggerReclaimFlow()` (Recommended)
```javascript
import { ReclaimProofRequest } from '@reclaimprotocol/js-sdk';

const handleVerification = async () => {
  const reclaimProofRequest = await ReclaimProofRequest.init(APP_ID, APP_SECRET, PROVIDER_ID);
  
  // Auto-detects environment (extension/QR/mobile)
  await reclaimProofRequest.triggerReclaimFlow();
  
  await reclaimProofRequest.startSession({
    onSuccess: (proofs) => {
      console.log('Verification successful:', proofs);
    },
    onError: (error) => {
      console.error('Verification failed', error);
    }
  });
};
```

#### Method 2: Manual QR Code
```javascript
const requestUrl = await reclaimProofRequest.getRequestUrl();
// Display as QR code or link
```

### Backend Verification

#### Node.js/Express
```javascript
import { ReclaimProofRequest } from '@reclaimprotocol/js-sdk';

app.post('/request-proof', async (req, res) => {
  const request = await ReclaimProofRequest.init(APP_ID, APP_SECRET, PROVIDER_ID);
  request.setCallbackUrl(`${BASE_URL}/receive-proofs`);
  
  const { requestUrl, statusUrl } = await request.createVerificationRequest();
  res.json({ requestUrl, statusUrl });
});

app.post('/receive-proofs', async (req, res) => {
  const proofs = req.body;
  const { isVerified, data, error } = await verifyProof(proofs, { providerId: PROVIDER_ID });
  if (isVerified) {
    const { context, extractedParameters } = data[0];
    // Process verified data
  }
});
```

#### Python/FastAPI
```python
from reclaim_python_sdk import ReclaimProofRequest

@app.post("/request-proof")
async def request_proof():
    request = ReclaimProofRequest(APP_ID, APP_SECRET, PROVIDER_ID)
    request.set_callback(f"{BASE_URL}/receive-proofs")
    
    request_url = await request.get_request_url()
    status_url = request.get_status_url()
    return {"requestUrl": request_url, "statusUrl": status_url}
```

## 📱 Mobile SDKs

### React Native

#### Installation (Expo)
```bash
npx expo install @reclaimprotocol/inapp-rn-sdk
```

Add to `app.json` plugins:
```json
"plugins": [
  "@reclaimprotocol/inapp-rn-sdk"
]
```

#### Usage
```javascript
import { ReclaimVerification } from '@reclaimprotocol/inapp-rn-sdk';

const handleVerify = async () => {
  const sessionId = await ReclaimVerification.startVerification({
    applicationId: APP_ID,
    applicationSecret: APP_SECRET,
    providerId: PROVIDER_ID,
  });
  
  // Handle success/failure
};
```

### iOS (Swift)

#### Installation
```swift
// Swift Package Manager
.package(url: "https://github.com/reclaimprotocol/reclaim-inapp-ios-sdk.git", from: "0.3.0")

// CocoaPods
pod 'ReclaimInAppSdk', '~> 0.3.0'
```

#### Performance Fix Required
Add to Xcode scheme environment variables:
- Key: `GODEBUG`
- Value: `asyncpreemptoff=1`

### Android (Kotlin)

#### Setup
```groovy
// settings.gradle
repositories {
    maven { url "https://reclaim-inapp-sdk.s3.ap-south-1.amazonaws.com/android/0.3.0/repo" }
}

// build.gradle
implementation "org.reclaimprotocol:inapp_sdk:0.3.0"
```

#### AndroidManifest.xml
```xml
<activity
    android:name="org.reclaimprotocol.inapp_sdk.ReclaimActivity"
    android:theme="@style/Theme.ReclaimInAppSdk.LaunchTheme"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reclaimprotocol/docs](https://github.com/reclaimprotocol/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
