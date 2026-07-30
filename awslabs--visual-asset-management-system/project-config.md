---
trigger: always_on
description: This document provides comprehensive guidelines for developing and extending VAMS backend APIs and CDK infrastructure. Follow these rules to ensure consistency, quality, and maintainability across all backend and infrastructure implementations.
---

# VAMS Backend + CDK Development Workflow & Rules

This document provides comprehensive guidelines for developing and extending VAMS backend APIs and CDK infrastructure. Follow these rules to ensure consistency, quality, and maintainability across all backend and infrastructure implementations.

## 🏗️ **Architecture Overview**

### **File Structure Standards**

```
backend/
├── backend/
│   ├── handlers/                # Lambda function handlers (one per API domain)
│   │   ├── assets/             # Asset-related handlers
│   │   │   ├── assetService.py # GOLD STANDARD implementation
│   │   │   ├── createAsset.py  # Asset creation handler
│   │   │   └── uploadFile.py   # File upload handler
│   │   ├── databases/          # Database-related handlers
│   │   └── [domain]/           # Other domain handlers
│   ├── models/                 # Pydantic request/response models
│   │   ├── assetsV3.py        # Asset API models (GOLD STANDARD)
│   │   ├── common.py          # Common response models
│   │   └── [domain].py        # Domain-specific models
│   ├── common/                # Shared utilities
│   │   ├── constants.py       # Constants and configuration
│   │   ├── validators.py      # Input validation functions
│   │   └── dynamodb.py        # DynamoDB utilities
│   └── customLogging/         # Logging utilities
├── tests/                     # Test files (mirror handler structure)
│   ├── handlers/              # Handler tests
│   ├── models/                # Model tests
│   └── conftest.py           # Test configuration
└── requirements.txt          # Python dependencies

infra/
├── lib/
│   ├── nestedStacks/
│   │   ├── apiLambda/         # API Gateway and Lambda definitions
│   │   │   ├── apiBuilder-nestedStack.ts  # API route definitions
│   │   │   └── constructs/    # Custom constructs
│   │   └── storage/           # Storage resource definitions
│   │       └── storageBuilder-nestedStack.ts  # DynamoDB, S3, SNS
│   ├── lambdaBuilder/         # Lambda function builders
│   │   ├── assetFunctions.ts  # Asset lambda builders
│   │   └── [domain]Functions.ts  # Domain lambda builders
│   └── helper/                # CDK helper utilities
└── config/                   # Configuration files
```

## 📋 **Development Workflow Checklist**

### **Phase 1: Pre-Implementation**

-   [ ] **Analyze Requirements**: Understand the new API/feature requirements
-   [ ] **Review Gold Standard**: Study `assetService.py` for implementation patterns
-   [ ] **Plan API Design**: Design request/response models and endpoints
-   [ ] **Plan CDK Changes**: Identify required infrastructure changes
-   [ ] **Plan Authorization**: Determine permission requirements and object types
-   [ ] **Plan Frontend Integration**: Identify frontend service changes needed
-   [ ] **Plan CLI Integration**: Identify CLI command changes needed
-   [ ] **Plan Documentation**: Identify documentation updates required

### **Phase 2: Implementation**

#### **Step 1: Backend Models (Pydantic)**

-   [ ] **Create Request Models**: Add Pydantic models in `models/[domain].py`
-   [ ] **Create Response Models**: Add response models with proper typing
-   [ ] **Add Validation Logic**: Include `@root_validator` for complex validation
-   [ ] **Follow Gold Standard**: Use `assetsV3.py` patterns for validation
-   [ ] **Import in Models**: Add new models to appropriate `__init__.py`

#### **Step 2: Backend Handler Implementation**

-   [ ] **Create Handler File**: Add handler in `handlers/[domain]/[handler].py`
-   [ ] **Follow Gold Standard**: Use `assetService.py` patterns for structure
-   [ ] **Implement Error Handling**: Use comprehensive try/catch with proper exceptions
-   [ ] **Add Authorization**: Include Casbin enforcement with object-type checking
-   [ ] **Add Logging**: Use `safeLogger` for structured logging
-   [ ] **Add Environment Variables**: Load required environment variables with error handling
-   [ ] **Add AWS Clients**: Configure AWS clients with retry configuration
-   [ ] **Implement Business Logic**: Separate business logic from request handling
-   [ ] **Add Response Enhancement**: Include version info and bucket details where applicable

#### **Step 3: CDK Infrastructure**

-   [ ] **Update Storage Resources**: Add new DynamoDB tables/S3 buckets in `storageBuilder-nestedStack.ts`
-   [ ] **Create Lambda Builder**: Add lambda function builder in `lambdaBuilder/[domain]Functions.ts`
-   [ ] **Configure Environment Variables**: Pass storage resources to lambda environment
-   [ ] **Configure Permissions**: Grant appropriate DynamoDB/S3/SNS permissions
-   [ ] **Configure VPC**: Add VPC/subnet configuration based on config flags
-   [ ] **Add KMS Permissions**: Include KMS key permissions for encryption
-   [ ] **Add API Routes**: Register routes in `apiBuilder-nestedStack.ts`
-   [ ] **Follow Naming Conventions**: Use consistent naming patterns

#### **Step 4: API Gateway Integration**

-   [ ] **Add Route Definitions**: Use `attachFunctionToApi` for route registration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awslabs/visual-asset-management-system](https://github.com/awslabs/visual-asset-management-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
