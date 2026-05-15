---
trigger: always_on
description: This document provides comprehensive context for AI assistants (Claude, Copilot, Cursor, etc.) working on the prmana project. It captures our philosophy, architecture decisions, security invariants, and the reasoning behind key design choices.
---

# CLAUDE.md - AI Assistant Guide for prmana

This document provides comprehensive context for AI assistants (Claude, Copilot, Cursor, etc.) working on the prmana project. It captures our philosophy, architecture decisions, security invariants, and the reasoning behind key design choices.

## Project Philosophy

### Security Should Not Be Annoying

The fundamental premise of prmana is that **security and usability are not opposing forces**. When security is annoying, people circumvent it. When it's an IT burden, organizations disable it. We believe:

- **Authentication should be invisible when possible** - Single sign-on means users authenticate once and work seamlessly
- **Step-up authentication should feel natural** - When elevated privileges are needed, the flow should be quick and intuitive
- **Configuration should have sensible defaults** - Secure out of the box, with knobs for enterprise customization
- **Failure modes should be clear** - When something goes wrong, the user should understand why and how to fix it

**Important clarification**: "Security shouldn't be annoying" does NOT mean "if security is annoying, disable it." It means **find a better UX**. The response to friction is always to improve the experience while maintaining security, never to remove the security check.

### Conservative Security, Pragmatic Usability

We follow the principle of **defense in depth with graceful degradation**:

1. **Default to the most secure option** that doesn't break legitimate use cases
2. **Warn before rejecting** when encountering edge cases (e.g., missing JTI claims)
3. **Make security configurable** for enterprises with different risk profiles (see issue #10)
4. **Never silently fail** - if a security check can't be performed, log it prominently

This means we might accept a token with a missing JTI claim (with a warning) rather than lock out a user whose IdP doesn't implement that optional field - but we make it configurable so strict environments can enforce it.

### The Problem We're Solving

Traditional Unix authentication has a fundamental disconnect with modern identity:

- **SSH keys get copied, shared, and never rotated** - That key on a developer's laptop from 2019? Still works.
- **When someone leaves, finding all their access is archaeology** - authorized_keys files scattered across servers
- **Enterprise MFA stops at the browser** - You need MFA for email but not for root access to production?
- **Compliance is painful** - "Show me who accessed what" requires parsing logs from dozens of sources

prmana bridges this gap by bringing OIDC (the same protocol behind "Sign in with Google/Microsoft/Okta") to Linux PAM, with DPoP token binding to prevent token theft.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         User's Machine                               │
│  ┌─────────────┐    ┌──────────────────┐    ┌───────────────────┐  │
│  │ SSH Client  │───▶│ oidc-ssh-agent   │───▶│ Identity Provider │  │
│  └─────────────┘    │ (token + DPoP)   │    │ (Okta/Azure/etc)  │  │
│                     └──────────────────┘    └───────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              │ SSH with token in env/keyboard-interactive
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         Linux Server                                 │
│  ┌─────────────┐    ┌──────────────────┐    ┌───────────────────┐  │
│  │   sshd      │───▶│  PAM Module      │───▶│ Token Validation  │  │
│  └─────────────┘    │ (pam_prmana)  │    │ + DPoP Verify     │  │
│                     └──────────────────┘    └───────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### Key Components

| Component | Location | License | Purpose |
|-----------|----------|---------|---------|
| `prmana-core` | `/prmana-core/` | Apache-2.0 | Shared OIDC discovery, JWKS types, common validation primitives |
| `pam-prmana` | `/pam-prmana/` | Apache-2.0 | Core PAM module (Rust) — depends on prmana-core |
| `prmana-agent` | `/prmana-agent/` | Apache-2.0 | Client-side agent daemon (token acquisition, DPoP, device flow) |
| DPoP Libraries | `/*-oauth-dpop/` | Apache-2.0 | Cross-language DPoP implementation |
| Enterprise crates | `/enterprise/crates/` | Proprietary | SCIM, kubectl, sudo step-up, CIBA, failover, token exchange, SPIFFE |

### Why DPoP (Demonstrating Proof of Possession)?

Standard OAuth tokens are bearer tokens - anyone who has them can use them. DPoP (RFC 9449) adds cryptographic binding:

1. Client generates an ephemeral key pair
2. Each request includes a proof signed with the private key
3. Server verifies the proof matches the token's thumbprint
4. **Even if an attacker intercepts the token, they can't use it** without the private key

This is the same protection used by banking APIs and is critical for SSH where tokens might traverse untrusted networks.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prodnull/prmana](https://github.com/prodnull/prmana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
