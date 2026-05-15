---
trigger: always_on
description: Use when the sub-resource shares the same parent IDs for all operations.
---

# BlindPay Swift SDK -- Agent Reference

This document is optimized for an AI agent modifying this codebase. It describes the project structure, conventions, and step-by-step procedures for common changes.

## 1. Project Structure

```
blindpay-swift/
  Package.swift                         # SPM manifest. swift-tools-version: 5.7. Platforms: iOS 15+, macOS 12+, tvOS 15+, watchOS 8+.
  Sources/BlindPay/
    blindpay_swift.swift                # Empty entry point (ASCII art only).
    Core/
      BlindPay.swift                    # Main client class. Facade that duplicates every service method as a flat convenience method AND exposes service objects.
      Configuration.swift               # Configuration struct (baseURL only).
    API/
      APIClient.swift                   # Internal HTTP client. Generic request<T: Codable>() method. Handles APIResponse<T> envelope and error decoding.
      Errors.swift                      # BlindPayError enum (invalidURL, invalidResponse, encodingError, decodingError, networkError, httpError).
    Models/
      APIResponse.swift                 # APIResponse<T> and APIError. Generic response envelope.
      Common.swift                      # Country enum (ISO 3166-1 alpha-2), PaginationParams, PaginationMetadata.
      Available.swift                   # Rail, BankAccountType, AccountClass, RecipientRelationship, NaicsCode, RailResponse, BankDetailKey, BankDetailItem, BankDetailField, SwiftCodeResponse.
      Instance.swift                    # InstanceMemberRole, InstanceMember, UpdateInstanceInput, UpdateMemberRoleInput, VoidResponse.
      Receiver.swift                    # KYCType, ReceiverType, IDDocType, ProofOfAddressDocType, SourceOfFundsDocType, PurposeOfTransactions, OwnerRole, ReceiverOwner, CreateReceiverInput, Receiver (full model), UpdateReceiverInput, response types, ListReceiversResponse.
      ApiKey.swift                      # ApiKeyPermission, ApiKey, CreateApiKeyInput, CreateApiKeyResponse.
      BankAccount.swift                 # BankAccount, CreateBankAccountInput, CreateBankAccountResponse, SpeiProtocol, TransfersType + related enums.
      BlockchainWallet.swift            # BlockchainWallet, CreateBlockchainWalletInput, BlockchainWalletSignMessageResponse, asset trustline types, mint USDB types, Solana delegate types.
      CustodialWallet.swift             # CustodialWallet, CreateCustodialWalletInput, WalletTokenBalance, CustodialWalletBalanceResponse.
      Fee.swift                         # FeeOptions, FeesResponse (per-rail fee structure).
      OfframpWallet.swift               # OfframpWallet, CreateOfframpWalletInput, CreateOfframpWalletResponse.
      PartnerFee.swift                  # PartnerFee, CreatePartnerFeeInput, CreatePartnerFeeResponse.
      Payin.swift                       # PayinStatus, PaymentMethod, PayinType, PayinTrackingTransaction, Payin, CreatePayinInput, ListPayinsInput, response types.
      PayinQuote.swift                  # PayinQuotePayerRules, PayinQuote, CreatePayinQuoteInput, ListPayinQuotesInput, response types.
      Payout.swift                      # PayoutStatus, TrackingStep, ProviderStatus, PaymentType, Payout, CreatePayoutEvmInput, CreatePayoutStellarInput, CreatePayoutSolanaInput, AuthorizeStellarInput/Response, AuthorizeSolanaInput/Response, ListPayoutsInput, response types.
      Quote.swift                       # CurrencyType, Network, StablecoinToken, TransactionDocumentType, Currency, CreateQuoteInput, QuoteNetwork, QuoteContract, CreateQuoteResponse, GetFxRateInput, GetFxRateResponse, AnyCodable helper.
      ReceiverLimits.swift              # SupportingDocumentType, LimitIncreaseRequestStatus, PayinLimits, PayoutLimits, ReceiverLimitsResponse, RequestLimitIncreaseInput/Response, LimitIncreaseRequest.
      TermsOfService.swift              # InitiateTosInput, InitiateTosResponse.
      Transaction.swift                 # TransactionStatus, TrackingTransaction, TrackingPayment, TrackingLiquidity, TrackingComplete, TrackingPartnerFee.
      Transfer.swift                    # TransferTrackingStep, TransferTrackingTransactionMonitoring, Transfer.
      VirtualAccount.swift              # BankingPartner, BlockchainWalletRef, VirtualAccountAccountType, VirtualAccountACH, VirtualAccount, CreateVirtualAccountInput, UpdateVirtualAccountInput, response types.
      WebhookEndpoint.swift             # WebhookEvent, WebhookEndpoint, CreateWebhookEndpointInput, CreateWebhookEndpointResponse, WebhookEndpointSecret, WebhookPortalAccess, DeleteWebhookEndpointResponse.
    Services/
      AvailableService.swift            # Top-level service. No instanceId. Methods: getRails, getBankDetails, getSwiftCode.
      InstancesService.swift            # Instance-scoped service. Holds sub-services (apiKeys, partnerFees, quotes, webhookEndpoints, payins, payouts, termsOfService). Direct methods for members, receivers, asset trustline, mint, delegate. Factory method: receivers(receiverId:) -> ReceiversService.
      ReceiversService.swift            # Receiver-scoped service. Holds sub-services (blockchainWallets, virtualAccounts, bankAccounts, custodialWallets). No direct methods -- purely a sub-service container.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blindpaylabs/blindpay-swift](https://github.com/blindpaylabs/blindpay-swift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
