---
trigger: always_on
description: This guide defines the workflow for managing the ACS (Automated Communication System) CDK infrastructure codebase. The infrastructure is designed to be **dev-friendly**, **dynamic**, and **automated** while maintaining **simplicity** and **efficiency** in development workflows.
---

# ACS CDK Infrastructure Workflow Guide

## 🎯 Overview

This guide defines the workflow for managing the ACS (Automated Communication System) CDK infrastructure codebase. The infrastructure is designed to be **dev-friendly**, **dynamic**, and **automated** while maintaining **simplicity** and **efficiency** in development workflows.

## 🏗️ Architecture Principles

### Core Design Philosophy
- **Compartmentalization**: Each component is isolated for easy maintenance
- **Automation**: Minimal manual intervention required
- **Dynamism**: Environment variables and settings are automatically managed
- **Content Preservation**: Critical data is retained across deployments
- **Multi-Region**: Single account, multiple regions with shared resources

### Key Features
- **Auto-Detection**: Lambda runtime and handlers are automatically detected
- **Environment Management**: Seamless switching between dev/prod environments
- **Resource Retention**: S3, DynamoDB, and other data stores preserve content
- **Shared Resources**: Cognito and other global resources are shared across regions

## 📁 Project Structure

```
acsd2p/
├── bin/acsd2p.ts                 # CDK App entry point
├── lib/acsd2p-stack.ts           # Main infrastructure stack
├── lambdas/                      # Lambda functions (auto-detected)
│   ├── [FunctionName]/           # Each function in its own directory
│   │   ├── lambda_function.py    # Python handler (auto-detected)
│   │   ├── index.js/mjs          # Node.js handler (auto-detected)
│   │   ├── requirements.txt      # Python dependencies
│   │   └── package.json          # Node.js dependencies
├── cdk.json                      # CDK configuration with environment contexts
├── config.env                    # Environment configuration
└── WORKFLOW_GUIDE.md            # This guide
```

## 🔧 Development Workflow

### 1. Environment Setup

#### Prerequisites
```bash
# Install dependencies
npm install

# Install AWS CDK globally (if not already installed)
npm install -g aws-cdk
```

#### Environment Configuration
1. **Copy configuration template**:
   ```bash
   cp config.env .env.local
   ```

2. **Configure environment variables** in `.env.local`:
   ```env
   # For existing Cognito resources (recommended for production)
   EXISTING_USER_POOL_ID=us-west-1_xxxxxxxxx
   EXISTING_USER_POOL_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxx
   EXISTING_USER_POOL_CLIENT_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxx
   
   # Environment selection
   ENVIRONMENT=dev
   ```

### 2. Lambda Development

#### Adding New Lambda Functions
1. **Create function directory** in `lambdas/`:
   ```bash
   mkdir lambdas/MyNewFunction
   ```

2. **Add function code** (supports both Python and Node.js):
   ```python
   # lambdas/MyNewFunction/lambda_function.py
   def handler(event, context):
       return {
           'statusCode': 200,
           'body': 'Hello from MyNewFunction!'
       }
   ```

3. **Add dependencies** (if needed):
   ```bash
   # For Python
   echo "requests==2.31.0" > lambdas/MyNewFunction/requirements.txt
   
   # For Node.js
   echo '{"dependencies": {"axios": "^1.6.0"}}' > lambdas/MyNewFunction/package.json
   ```

#### Auto-Detection Features
- **Runtime Detection**: Automatically detects Python vs Node.js based on file presence
- **Handler Detection**: Finds the appropriate entry point (`lambda_function.py`, `index.js`, etc.)
- **Environment Variables**: All functions automatically receive shared environment variables
- **Permissions**: Automatic IAM permissions for DynamoDB, S3, SQS, and Cognito

### 3. Database Management

#### DynamoDB Tables
All tables are created with `RemovalPolicy.RETAIN` to preserve data during deployments:

```typescript
// Tables are automatically created with retention policies
const table = new dynamodb.Table(this, 'TableName', {
  removalPolicy: cdk.RemovalPolicy.RETAIN, // Data preserved during deployments
  // ... other configuration
});
```

#### Adding New Tables
1. **Add table definition** in `lib/acsd2p-stack.ts`
2. **Grant permissions** to lambda functions (automatic)
3. **Add to shared environment** variables if needed

### 4. API Gateway Management

#### Adding New Routes
1. **Add route definition** in the `routeMap` array:
   ```typescript
   { path: ['api', 'new', 'endpoint'], method: 'POST', lambda: "MyNewFunction" }
   ```

2. **CORS is automatically configured** for all routes
3. **Lambda integration is automatic**

## 🚀 Deployment Workflow

### Development Deployment
```bash
# Deploy to development environment (us-west-1)
npx cdk deploy --context env=dev

# Or using environment variable
ENVIRONMENT=dev npx cdk deploy
```

### Production Deployment
```bash
# Deploy to production environment (us-east-2)
npx cdk deploy --context env=prod --require-approval never
```

### Deployment Safety Features
- **Production Warning**: 10-second delay with clear warnings
- **Environment Validation**: Ensures correct region deployment
- **Resource Retention**: Critical data is preserved
- **Rollback Capability**: Previous versions can be restored


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anaypant) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
