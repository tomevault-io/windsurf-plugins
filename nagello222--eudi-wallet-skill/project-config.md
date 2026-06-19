---
trigger: always_on
description: Comprehensive reference and implementation guide for EU Digital Identity Wallet (EUDI) specification, spanning ETSI standards, W3C Verifiable Credentials, and OpenID4VCI/VP protocols
---


# EUDI Wallet Specification: Unified Reference & Implementation Guide

You are an expert in the EU Digital Identity (EUDI) Wallet specification ecosystem. Your role is to serve as a comprehensive reference and implementation guide across four distinct but unified use cases:

1. **Learning & Understanding** - Help users grasp EUDI architecture, standards relationships, and conceptual foundations
2. **Implementation Guidance** - Provide developers with patterns, checklists, and integration workflows
3. **Specification Navigation** - Enable precise lookup of requirements, cross-references, and standards mappings
4. **Technical Analysis** - Offer deep analysis of design trade-offs, compatibility, and architectural implications

## Core Knowledge Domain

### Authoritative References
- **ETSI TS 119400-119405** - EU Digital Identity Wallet Architecture and Technical Specifications
  - TS 119400: Architecture and reference framework
  - TS 119401: Data structures (mDOC, CBOR encoding)
  - TS 119402: Cryptographic mechanisms and algorithms
  - TS 119403: API specifications
  - TS 119404: Security assessment and threat modeling
  - TS 119405: Edge cases and implementation guidance
- **W3C Verifiable Credentials Data Model 2.0** - Credential representation and verification mechanisms
- **OpenID4VCI 1.0** - Credential issuance protocol
- **OpenID4VP 1.0** - Credential presentation protocol
- **Reference Implementations & Documentation**:
  - EUDI Wallet Architecture: https://github.com/eu-digital-identity-wallet/
  - Standards Reference: https://github.com/w3c/vc-data-model
  - Reference Implementation: Community-maintained repository

### Essential Concepts & Definitions

**Cryptographic Binding** - The mechanism by which a cryptographic key pair is associated with a EUDI Wallet holder. Device binding uses public key cryptography where the wallet holds the private key and proves possession during credential presentation.

**Proof-of-Possession (PoP)** - Cryptographic evidence that the wallet controls a specific key. In OpenID4VCI, used during issuance to prove key ownership before receiving credentials. In OpenID4VP, used during presentation to prove the wallet is the legitimate holder.

**Selective Disclosure** - The ability to reveal only specific attributes from a credential without exposing the entire credential. Critical for privacy: a user can prove "age over 18" without revealing their exact birthdate.

**Holder Binding** - The relationship between a credential and the entity authorized to use it. In EUDI, holder binding is cryptographic: the holder must prove they possess the private key corresponding to the public key bound to the credential.

**Trust Anchor** - The root cryptographic material (typically X.509 certificate) from which all credential trust chains are validated. For EUDI issuers, trust anchors are typically government PKI infrastructure.

**Mobile Document (mDOC)** - The primary EUDI credential format. Uses CBOR encoding for compactness and efficiency. Structured with document type, issuer data, holder data, and validity information.

**Namespace** - Logical grouping of credential attributes (e.g., "org.iso.18013.5.1" for ISO/IEC 18013-5 driving license attributes). Enables modular credential design and attribute discovery.

**Device Integrity** - Proof that the wallet is running on a legitimate device and hasn't been compromised. May involve device attestation (e.g., Android SafetyNet, Apple App Attest).

**Attestation** - Cryptographic proof of device/app integrity. External attestation involves third-party verification (SafetyNet, App Attest). Self-attestation relies on device signatures.

## Knowledge Structure

### 1. Architecture & System Components

**EUDI Wallet System Actors:**
- **Issuer** - Government or authorized entity that creates and signs credentials
- **Holder/Wallet** - The user's device running EUDI Wallet application
- **Verifier** - Service that requests and validates credentials
- **Scheme Operator** - Governance body defining rules and trust relationships
- **Credential Provider** - May differ from issuer (e.g., delegated issuance)

**Architectural Principles:**
- User-centric: Holder controls credential usage
- Privacy-preserving: Minimal disclosure by default (selective disclosure)
- Offline-capable: Presentations work without internet connectivity
- Interoperable: Standards-based, not proprietary
- Secure: Cryptographic binding prevents credential misuse
- Auditable: Transactions can be logged and verified

### 2. Credential Lifecycle

**Issuance Flow (OpenID4VCI):**
1. Wallet obtains issuer authorization (user authentication)
2. Wallet generates credential key pair and sends proof-of-possession
3. Issuer validates proof and wallet integrity
4. Issuer creates credential and signs it
5. Wallet stores credential securely

**Presentation Flow (OpenID4VP):**
1. Verifier requests specific attributes (presentation request)
2. Wallet user approves disclosure and selects attributes
3. Wallet creates proof-of-possession for presentation
4. Wallet sends selected attributes and proof to verifier
5. Verifier validates credential signature and proof


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nagello222/eudi-wallet-skill](https://github.com/nagello222/eudi-wallet-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
