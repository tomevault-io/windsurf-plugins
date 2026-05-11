---
trigger: always_on
description: Guidelines for developing cloud provider integrations in Armature.
---


# Cloud Provider Integrations

Guidelines for developing cloud provider integrations in Armature.

## Cloud Crates Overview

| Crate | Provider | Services |
|-------|----------|----------|
| `armature-aws` | AWS | S3, DynamoDB, SQS, SNS, SES, Lambda, KMS, Cognito |
| `armature-gcp` | GCP | Cloud Storage, Pub/Sub, Firestore, Spanner, BigQuery |
| `armature-azure` | Azure | Blob Storage, Cosmos DB, Service Bus, Key Vault |
| `armature-redis` | Redis | Connection pooling, Pub/Sub, Cluster |
| `armature-lambda` | AWS Lambda | Lambda runtime integration |
| `armature-cloudrun` | Cloud Run | Cloud Run runtime integration |
| `armature-azure-functions` | Azure Functions | Azure Functions runtime |

## Architecture Pattern

### Feature-Gated Services

Each cloud crate uses feature flags to enable only needed services:

```toml
# armature-aws/Cargo.toml
[features]
default = []
full = ["s3", "dynamodb", "sqs", "sns", "ses", "lambda", "kms", "cognito"]
s3 = ["aws-sdk-s3"]
dynamodb = ["aws-sdk-dynamodb"]
sqs = ["aws-sdk-sqs"]
sns = ["aws-sdk-sns"]
ses = ["aws-sdk-ses"]
lambda = ["aws-sdk-lambda"]
kms = ["aws-sdk-kms"]
cognito = ["aws-sdk-cognitoidentityprovider"]
```

### Service Factory Pattern

```rust
// armature-aws/src/lib.rs
pub struct AwsServices {
    config: AwsConfig,
    #[cfg(feature = "s3")]
    s3: OnceCell<S3Client>,
    #[cfg(feature = "dynamodb")]
    dynamodb: OnceCell<DynamoDbClient>,
    // ... other services
}

impl AwsServices {
    pub async fn new(config: AwsConfig) -> Result<Self, AwsError> {
        Ok(Self {
            config,
            #[cfg(feature = "s3")]
            s3: OnceCell::new(),
            #[cfg(feature = "dynamodb")]
            dynamodb: OnceCell::new(),
        })
    }

    #[cfg(feature = "s3")]
    pub fn s3(&self) -> Result<&S3Client, AwsError> {
        self.s3.get_or_try_init(|| {
            S3Client::new(&self.config.sdk_config)
        })
    }

    #[cfg(feature = "dynamodb")]
    pub fn dynamodb(&self) -> Result<&DynamoDbClient, AwsError> {
        self.dynamodb.get_or_try_init(|| {
            DynamoDbClient::new(&self.config.sdk_config)
        })
    }
}
```

### Configuration from Environment

```rust
// armature-aws/src/config.rs
#[derive(Debug, Clone)]
pub struct AwsConfig {
    pub region: String,
    pub sdk_config: SdkConfig,
    enabled_services: HashSet<String>,
}

impl AwsConfig {
    pub fn from_env() -> AwsConfigBuilder {
        AwsConfigBuilder {
            region: std::env::var("AWS_REGION").ok(),
            profile: std::env::var("AWS_PROFILE").ok(),
            endpoint_url: std::env::var("AWS_ENDPOINT_URL").ok(),
            enabled_services: HashSet::new(),
        }
    }

    pub fn builder() -> AwsConfigBuilder {
        AwsConfigBuilder::default()
    }
}

#[derive(Default)]
pub struct AwsConfigBuilder {
    region: Option<String>,
    profile: Option<String>,
    endpoint_url: Option<String>,
    enabled_services: HashSet<String>,
}

impl AwsConfigBuilder {
    pub fn region(mut self, region: impl Into<String>) -> Self {
        self.region = Some(region.into());
        self
    }

    pub fn enable_s3(mut self) -> Self {
        self.enabled_services.insert("s3".to_string());
        self
    }

    pub fn enable_dynamodb(mut self) -> Self {
        self.enabled_services.insert("dynamodb".to_string());
        self
    }

    pub async fn build(self) -> Result<AwsConfig, AwsError> {
        let mut config_loader = aws_config::defaults(BehaviorVersion::latest());

        if let Some(region) = &self.region {
            config_loader = config_loader.region(Region::new(region.clone()));
        }

        if let Some(endpoint) = &self.endpoint_url {
            config_loader = config_loader.endpoint_url(endpoint);
        }

        let sdk_config = config_loader.load().await;

        Ok(AwsConfig {
            region: self.region.unwrap_or_else(|| "us-east-1".to_string()),
            sdk_config,
            enabled_services: self.enabled_services,
        })
    }
}
```

## DI Integration Pattern

```rust
// In user's module
use armature::prelude::*;
use armature_aws::*;

#[module(
    providers: [AwsServicesProvider],
    controllers: [FileController],
)]
struct CloudModule;

// Provider for DI
#[injectable]
pub struct AwsServicesProvider {
    services: Arc<AwsServices>,
}

impl AwsServicesProvider {
    pub async fn new() -> Result<Self, AwsError> {
        let config = AwsConfig::from_env()
            .enable_s3()
            .enable_sqs()
            .build()
            .await?;

        let services = AwsServices::new(config).await?;

        Ok(Self {
            services: Arc::new(services),
        })
    }
}

// Usage in controller
#[controller("/files")]
struct FileController {
    aws: AwsServicesProvider,
}

impl FileController {
    #[post("/upload")]
    async fn upload(&self, body: Bytes) -> Result<Json<UploadResponse>, Error> {
        let s3 = self.aws.services.s3()?;

        s3.put_object()
            .bucket("my-bucket")
            .key("file.txt")
            .body(body.into())
            .send()
            .await?;

        Ok(Json(UploadResponse { success: true }))
    }
}
```

## Error Handling

### Unified Error Type

```rust
// armature-aws/src/error.rs
use thiserror::Error;


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/armature](https://github.com/quinnjr/armature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
