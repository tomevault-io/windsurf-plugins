---
trigger: always_on
description: > This rules system implements PID (Proportional-Integral-Derivative) control theory for software development,
---

<!--
# PID Control Theory in Software Development

> This rules system implements PID (Proportional-Integral-Derivative) control theory for software development,
> providing a scientific approach to process control and quality assurance.

## Core Concepts
1. P (Project Guidelines): Direct control through standards
2. I (Workflow): Continuous execution and experience accumulation  
3. D (Thinking Protocol): Predictive analysis and dynamic adjustment

## Control System Design

### 1. Project Guidelines (P)
@type: proportional_control
- Acts as proportional control
- Provides immediate response to deviations
- Directly controls current behavior through standards
- Establishes baseline quality requirements

### 2. Workflow (I) 
@type: integral_control
- Functions as integral control
- Accumulates development experience
- Eliminates systematic errors over time
- Builds consistent execution patterns

### 3. Thinking Protocol (D)
@type: derivative_control  
- Serves as derivative control
- Predicts potential issues
- Provides dynamic adjustment
- Prevents quality fluctuations

## Control Loop Process

### 1. Standards Application (P)
- Apply project guidelines
- Check against quality metrics
- Provide immediate feedback

### 2. Experience Integration (I)
- Execute development workflow
- Accumulate best practices
- Build systematic improvements

### 3. Predictive Optimization (D)
- Apply thinking protocol
- Anticipate potential issues
- Make proactive adjustments
-->
# Project Guidelines
@type: proportional_control
@description: Provides immediate quality control through standards and metrics

## Directory Structure
- api/ @description: API documentation and Swagger files
- controller/ @description: HTTP handlers
- logic/ @description: Business logic
- repository/
  - dto/ @description: Request objects
  - vo/ @description: Response objects
  - model/ @description: Database models
- services/
  - xxx_service/ @description: Complex business services
  - xxx/ @description: Utility services
- middleware/ @description: Global middleware
- third_party/ @description: Third-party integrations
- constant/
  - errcode/ @description: Error codes
- tests/ @description: Test files
- util/ @description: Utility functions
- scripts/ @description: Database migration scripts

## Naming Conventions
- Files: snake_case (webhook_controller.go)
- Controller: webhook_controller.go
- Logic: webhook_logic.go
- DTO: create_webhook_dto.go
- VO: webhook_vo.go
- Model: webhook.go
- Test: {file_name}_test.go
- Service: paypal.go, mailchimp.go

## Layer Standards

### Controller Standards
@scope: controller
- Use application.TraceCtx(ctx)
- Return echo.Success/Error
- Add swagger annotations
- Use gin.Context as parameter
- Check authentication with jwt_token
- Validate input with ShouldBind

#### Swagger Annotations Required
- @Summary: Brief description
- @Description: Detailed description
- @Tags: API grouping
- @Produce: Response content type
- @Param: Request parameters
- @Success: Success response
- @Router: API path and method

### Logic Standards
@scope: logic
- Accept context as first param
- Return domain errors
- Handle transactions

#### Business Validation
- Input parameter validation
- Business rule validation
- State validation

#### Transaction Handling
- Use model.BeginCtxTx for transactions
- Proper error handling in transactions
- Rollback on errors

#### Error Handling
- Use errcode for business errors
- Wrap errors with context
- Log errors appropriately

### Repository Standards

#### DTO Standards
@scope: repository/dto
- All request parameters in /repository/dto
- Validation tags required
- Conversion methods to models
- Default value handling
- Use binding tags for validation

#### VO Standards
@scope: repository/vo
- All response structures in /repository/vo
- Conversion methods from models
- Proper JSON tags
- Documentation for fields

#### Model Standards
@scope: repository/model
- Define proper indexes
- Use appropriate field types
- Include field comments
- Database operations in /repository/model
- Transaction handling
- Error wrapping
- Query optimization

### Service Standards

#### Structure
@scope: service
- Include Ctx context.Context field
- Use New function for initialization
- Define interfaces for dependencies
- Use dependency injection

#### Methods
- Accept context as first param
- Return domain errors from errcode
- Handle retries for external calls
- Set appropriate timeouts

#### Database
- Use model.GetDB(ctx) for access
- Handle transactions properly
- Support transaction propagation
- Use batch operations for performance

## Database Standards

### SQL Database
@scope: sql
- Use model.GetDB(ctx) for operations
- Never directly inject *gorm.DB
- Use model.BeginCtxTx for transactions
- Follow GORM conventions

### Redis Database
@scope: redis
- Use go_redis.GetSession() for operations
- Never directly inject redis.Client
- Use go_redis.Lock/Unlock for distributed locks
- Handle Redis errors properly

### Connection Management
- Use context for timeouts
- Handle connection errors
- Proper resource cleanup
- Connection pooling

## Logging Standards

### Usage
@scope: logging
- Use cdslog.W(ctx) for all logging
- Include trace context

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vjj1999/Lazy-Cursor](https://github.com/vjj1999/Lazy-Cursor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
