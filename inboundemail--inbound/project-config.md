---
trigger: always_on
description: Amazon SES API v2 is an Amazon Web Services service for sending email messages to customers. This reference provides a comprehensive overview of the SESv2Client commands and configuration options.
---

# AWS SES v2 Client Reference

Amazon SES API v2 is an Amazon Web Services service for sending email messages to customers. This reference provides a comprehensive overview of the SESv2Client commands and configuration options.

## Installation

Install the AWS SDK for JavaScript v3 SES v2 client:

```bash
# Using bun (preferred for this project)
bun add @aws-sdk/client-sesv2

# Alternative package managers
npm install @aws-sdk/client-sesv2
yarn add @aws-sdk/client-sesv2
pnpm add @aws-sdk/client-sesv2
```

## Client Configuration

The SESv2Client accepts the following configuration parameters:

| Parameter | Type | Description |
|-----------|------|-------------|
| `region` | `string \| Provider<string>` | **Required.** The AWS region to send requests to |
| `credentials` | `AwsCredentialIdentity \| Provider<AwsCredentialIdentity>` | AWS credentials for authentication |
| `maxAttempts` | `number \| Provider<number>` | Maximum number of retry attempts (default: 3) |
| `retryMode` | `string \| Provider<string>` | Retry algorithm to use (`legacy`, `standard`, `adaptive`) |
| `logger` | `Logger` | Logger for debug/info/warn/error messages |
| `requestHandler` | `__HttpHandlerUserInput` | HTTP handler (Fetch in browser, Https in Node.js) |
| `defaultsMode` | `DefaultsMode \| Provider<DefaultsMode>` | How default configuration options are resolved |
| `profile` | `string` | AWS profile name for credential resolution |
| `useDualstackEndpoint` | `boolean \| Provider<boolean>` | Enable IPv6/IPv4 dualstack endpoint |
| `useFipsEndpoint` | `boolean \| Provider<boolean>` | Enable FIPS-compliant endpoints |
| `disableHostPrefix` | `boolean` | Disable dynamic endpoint modification |
| `extensions` | `RuntimeExtension[]` | Optional runtime extensions |

## Commands by Category

### Account Management
| Command | Purpose |
|---------|---------|
| `GetAccountCommand` | Get account email-sending status and capabilities |
| `PutAccountDetailsCommand` | Update Amazon SES account details |
| `PutAccountSendingAttributesCommand` | Enable/disable account email sending |
| `PutAccountSuppressionAttributesCommand` | Configure account-level suppression list |
| `PutAccountVdmAttributesCommand` | Update account VDM (Virtual Deliverability Manager) attributes |
| `PutAccountDedicatedIpWarmupAttributesCommand` | Configure automatic IP warm-up |

### Email Identities
| Command | Purpose |
|---------|---------|
| `CreateEmailIdentityCommand` | Start verifying an email address or domain |
| `DeleteEmailIdentityCommand` | Delete an email identity |
| `GetEmailIdentityCommand` | Get identity verification status and settings |
| `ListEmailIdentitiesCommand` | List all email identities in account |
| `PutEmailIdentityConfigurationSetAttributesCommand` | Associate identity with configuration set |
| `PutEmailIdentityDkimAttributesCommand` | Enable/disable DKIM authentication |
| `PutEmailIdentityDkimSigningAttributesCommand` | Configure DKIM signing attributes |
| `PutEmailIdentityFeedbackAttributesCommand` | Configure bounce/complaint feedback |
| `PutEmailIdentityMailFromAttributesCommand` | Configure custom Mail-From domain |

### Email Sending
| Command | Purpose |
|---------|---------|
| `SendEmailCommand` | Send a single email (simple, raw, or templated) |
| `SendBulkEmailCommand` | Send email to multiple destinations |
| `SendCustomVerificationEmailCommand` | Send custom verification email |

### Configuration Sets
| Command | Purpose |
|---------|---------|
| `CreateConfigurationSetCommand` | Create a configuration set |
| `DeleteConfigurationSetCommand` | Delete a configuration set |
| `GetConfigurationSetCommand` | Get configuration set details |
| `ListConfigurationSetsCommand` | List all configuration sets |
| `PutConfigurationSetArchivingOptionsCommand` | Configure email archiving |
| `PutConfigurationSetDeliveryOptionsCommand` | Associate with dedicated IP pool |
| `PutConfigurationSetReputationOptionsCommand` | Enable/disable reputation tracking |
| `PutConfigurationSetSendingOptionsCommand` | Enable/disable sending for configuration set |
| `PutConfigurationSetSuppressionOptionsCommand` | Configure suppression list preferences |
| `PutConfigurationSetTrackingOptionsCommand` | Configure open/click tracking domain |
| `PutConfigurationSetVdmOptionsCommand` | Configure VDM preferences |

### Event Destinations
| Command | Purpose |
|---------|---------|
| `CreateConfigurationSetEventDestinationCommand` | Create event destination |
| `DeleteConfigurationSetEventDestinationCommand` | Delete event destination |
| `GetConfigurationSetEventDestinationsCommand` | List event destinations |
| `UpdateConfigurationSetEventDestinationCommand` | Update event destination configuration |

### Email Templates
| Command | Purpose |
|---------|---------|
| `CreateEmailTemplateCommand` | Create an email template |
| `DeleteEmailTemplateCommand` | Delete an email template |
| `GetEmailTemplateCommand` | Get template details |
| `ListEmailTemplatesCommand` | List all email templates |
| `UpdateEmailTemplateCommand` | Update an email template |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inboundemail/inbound](https://github.com/inboundemail/inbound) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
