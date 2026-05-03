---
trigger: always_on
description: > **注意**: 本规范适用于 `aumate-app/` 和 `crates/` 下的 Tauri 应用开发。
---

# Aumate App 开发规范

> **注意**: 本规范适用于 `aumate-app/` 和 `crates/` 下的 Tauri 应用开发。
> 对于 Node.js automation library (`packages/`)，请参考 `CLAUDE.md`。

## 🏗️ 架构原则

### DDD 分层架构

本项目采用 Domain-Driven Design (DDD) 分层架构，严格遵循依赖倒置原则：

```
API Layer (Tauri Commands)
    ↓ 调用
Application Layer (Use Cases)
    ↓ 依赖接口
Domain Layer (Ports - Trait接口)
    ↑ 实现接口
Infrastructure Layer (Adapters)
```

### 核心规则

1. **依赖方向**
   - Domain Layer 定义接口（Ports），不依赖任何其他层
   - Infrastructure Layer 实现接口（Adapters）
   - Application Layer 通过接口调用，不直接依赖具体实现
   - API Layer 仅做参数验证和调用 Use Cases

2. **单一职责**
   - API Layer: 参数验证、错误转换
   - Application Layer: 业务流程编排
   - Domain Layer: 领域模型和业务规则
   - Infrastructure Layer: 技术实现细节

3. **命名约定**
   - Port接口: `xxxPort` (如 `ScreenCapturePort`)
   - Adapter实现: `xxxAdapter` (如 `ScreenCaptureAdapter`)
   - Use Case: `xxxUseCase` (如 `CaptureScreenUseCase`)
   - DTO: `xxxDto` / `xxxRequest` / `xxxResponse`

## 📦 代码组织

### Crate 结构

```
crates/
├── core/
│   ├── shared/       # 共享类型、错误定义
│   ├── domain/       # 领域模型
│   └── traits/       # Port 接口定义
├── application/      # Use Cases + DTOs
└── infrastructure/   # Adapters + Services

aumate-app/src-tauri/ # API Layer (应用特定)
└── src/
    ├── commands/     # Tauri Commands
    ├── state.rs      # AppState
    └── setup.rs      # 依赖注入
```

### 新功能实现流程

当添加新功能时，按照以下顺序实现：

#### 1. Domain Layer - 定义接口

**位置**: `crates/core/traits/src/`

```rust
// crates/core/traits/src/my_feature.rs
use async_trait::async_trait;
use aumate_core_shared::InfrastructureError;

#[async_trait]
pub trait MyFeaturePort: Send + Sync {
    async fn do_something(&self) -> Result<MyResult, InfrastructureError>;
}
```

**导出**: 在 `crates/core/traits/src/lib.rs` 中添加：
```rust
pub mod my_feature;
pub use my_feature::MyFeaturePort;
```

#### 2. Infrastructure Layer - 实现 Adapter

**位置**: `crates/infrastructure/src/adapters/`

```rust
// crates/infrastructure/src/adapters/my_feature.rs
use async_trait::async_trait;
use aumate_core_traits::MyFeaturePort;

pub struct MyFeatureAdapter {
    // 字段
}

impl MyFeatureAdapter {
    pub fn new() -> Self {
        Self {}
    }
}

#[async_trait]
impl MyFeaturePort for MyFeatureAdapter {
    async fn do_something(&self) -> Result<MyResult, InfrastructureError> {
        // 实现逻辑
        // 如果需要平台特定代码，使用：
        #[cfg(target_os = "macos")]
        {
            // macOS 实现
        }
        
        #[cfg(target_os = "windows")]
        {
            // Windows 实现
        }
    }
}
```

**平台特定代码** (如需要):
- 位置: `crates/infrastructure/src/platform/macos/` (或 `windows/`, `linux/`)
- 在 Adapter 中调用平台特定函数

**导出**: 在 `crates/infrastructure/src/adapters/mod.rs` 中添加：
```rust
pub mod my_feature;
pub use my_feature::MyFeatureAdapter;
```

#### 3. Application Layer - Use Case + DTO

**Use Case** (`crates/application/src/use_cases/my_feature.rs`):
```rust
use aumate_core_shared::UseCaseError;
use aumate_core_traits::MyFeaturePort;
use std::sync::Arc;

pub struct MyFeatureUseCase {
    feature: Arc<dyn MyFeaturePort>,
}

impl MyFeatureUseCase {
    pub fn new(feature: Arc<dyn MyFeaturePort>) -> Self {
        Self { feature }
    }

    pub async fn execute(&self) -> Result<MyResultDto, UseCaseError> {
        log::info!("[MyFeatureUseCase] Executing");

        let result = self.feature
            .do_something()
            .await
            .map_err(|e| e.into())?;

        // 转换为 DTO
        Ok(result.into())
    }
}
```

**DTO** (`crates/application/src/dto/my_feature.rs`):
```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct MyResultDto {
    // 字段
}

// 实现 From trait 进行转换
impl From<DomainType> for MyResultDto {
    fn from(value: DomainType) -> Self {
        Self {
            // 转换逻辑
        }
    }
}
```

**导出**: 
- `crates/application/src/use_cases/mod.rs` 添加 `pub mod my_feature;` 和 `pub use my_feature::*;`
- `crates/application/src/dto/mod.rs` 添加 `pub mod my_feature;` 和 `pub use my_feature::*;`

#### 4. API Layer - Tauri Command (仅在应用层)

**注意**: API commands 只在 `aumate-app/src-tauri/src/commands/` 中，不在 crates 里

**位置**: `aumate-app/src-tauri/src/commands/my_feature.rs`

```rust
use crate::state::AppState;
use aumate_application::dto::MyResultDto;
use aumate_core_shared::ApiError;
use tauri::State;

#[tauri::command]
pub async fn do_my_feature(
    state: State<'_, AppState>,
) -> Result<MyResultDto, String> {
    log::info!("API: do_my_feature called");

    state.my_feature_use_case
        .execute()
        .await
        .map_err(|e| {
            let api_error: ApiError = e.into();
            api_error.to_string()
        })
}
```

**注册命令**: 在 `aumate-app/src-tauri/src/lib.rs` 的 `invoke_handler` 中添加

#### 5. 依赖注入 - AppState

**State** (`aumate-app/src-tauri/src/state.rs`):
```rust
pub struct AppState {
    // ... existing fields ...
    pub my_feature_adapter: Arc<MyFeatureAdapter>,
    pub my_feature_use_case: Arc<MyFeatureUseCase>,
}
```

**Setup** (`aumate-app/src-tauri/src/setup.rs`):
```rust
pub fn setup_application() -> AppState {
    // 1. 创建 Adapter
    let my_feature_adapter = Arc::new(MyFeatureAdapter::new());
    
    // 2. 创建 Use Case
    let my_feature_use_case = Arc::new(MyFeatureUseCase::new(
        my_feature_adapter.clone()
    ));
    
    // 3. 返回 AppState
    AppState {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tegojs/bot](https://github.com/tegojs/bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
