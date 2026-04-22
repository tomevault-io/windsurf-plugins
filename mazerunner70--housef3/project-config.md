---
trigger: always_on
description: Backend Pydantic model patterns and conventions
---


# Backend Models Conventions

## Pydantic Model Architecture

### Model Structure Pattern
All domain models follow this standard structure:
```python
class ModelName(BaseModel):
    """Main domain model with full business logic"""
    # Core fields with proper types and aliases
    # Validators and business methods
    # DynamoDB serialization methods

class ModelNameCreate(BaseModel):
    """DTO for creating new instances - required fields only"""
    # Subset of fields needed for creation
    # Creation-specific validation

class ModelNameUpdate(BaseModel):
    """DTO for updates - all fields optional"""
    # All fields optional for partial updates
    # Update-specific validation
```

### Field Conventions

#### Field Naming & Aliases
- Use snake_case for Python field names
- Use camelCase aliases for API/JSON serialization
- Always provide aliases for external-facing fields:
```python
account_id: uuid.UUID = Field(default_factory=uuid.uuid4, alias="accountId")
user_id: str = Field(alias="userId")
created_at: int = Field(default_factory=timestamp_ms, alias="createdAt")
```

#### Required vs Optional Fields
- Use explicit `Optional[Type]` for nullable fields
- Set sensible defaults with `Field(default=value)`
- Use `Field(default_factory=func)` for dynamic defaults
- Required fields have no default value

#### Field Validation
- Use `field_validator` for single field validation
- Use `model_validator` for cross-field validation
- Validate at creation time, not just at runtime:
```python
@field_validator('created_at', 'updated_at')
@classmethod
def check_positive_timestamp(cls, v: int) -> int:
    if v < 0:
        raise ValueError("Timestamp must be positive")
    return v

@model_validator(mode='after')
def check_currency_if_balance_exists(self) -> Self:
    if self.balance is not None and self.currency is None:
        raise ValueError("Currency required when balance is set")
    return self
```

## Enum Handling

### String Enum Pattern
All enums inherit from `str` for JSON serialization:
```python
class Status(str, enum.Enum):
    ACTIVE = "active"
    INACTIVE = "inactive"
    PENDING = "pending"
```

### Enum Validation
Validate enums properly in field validators:
```python
@field_validator('currency', mode='after')
@classmethod
def validate_currency(cls, v, info: ValidationInfo) -> Optional[Currency]:
    if v is None:
        return None
    if isinstance(v, Currency):
        return v
    # Check database deserialization context
    if info.context and info.context.get('from_database'):
        return v
    raise ValueError(f"Currency must be Currency enum, got {type(v).__name__}")
```

### Enum Conversion Utilities
Provide helper functions for API input conversion:
```python
def convert_currency_input(currency_input: Any) -> Optional[Currency]:
    """Convert API input to Currency enum with proper error handling"""
    if currency_input is None:
        return None
    if isinstance(currency_input, Currency):
        return currency_input
    if isinstance(currency_input, str):
        try:
            return Currency(currency_input)
        except ValueError:
            valid_options = ', '.join([c.value for c in Currency])
            raise ValueError(f"Invalid currency: '{currency_input}'. Valid: {valid_options}")
    raise ValueError(f"Currency must be string or enum, got {type(currency_input).__name__}")
```

## DynamoDB Integration

### Serialization Pattern
Every model must implement DynamoDB serialization:
```python
def to_dynamodb_item(self) -> Dict[str, Any]:
    """Serialize to DynamoDB item format"""
    item = self.model_dump(mode='python', by_alias=True, exclude_none=True)
    
    # Convert UUIDs to strings
    if 'accountId' in item and isinstance(item['accountId'], uuid.UUID):
        item['accountId'] = str(item['accountId'])
    
    # Convert Decimals to strings for DynamoDB
    if 'balance' in item and isinstance(item['balance'], Decimal):
        item['balance'] = str(item['balance'])
    
    # Convert enums to string values
    if 'currency' in item and item.get('currency') is not None:
        item['currency'] = item['currency'].value if hasattr(item['currency'], 'value') else str(item['currency'])
    
    return item
```

### Deserialization Pattern
Use `model_construct()` to preserve enum objects:
```python
@classmethod
def from_dynamodb_item(cls, data: Dict[str, Any]) -> Self:
    """Deserialize from DynamoDB item"""
    # Copy to avoid modifying original
    converted_data = data.copy()
    
    # Convert Decimal fields
    if 'balance' in converted_data and converted_data['balance'] is not None:
        try:
            converted_data['balance'] = Decimal(str(converted_data['balance']))
        except decimal.InvalidOperation:
            raise ValueError(f"Invalid decimal: {converted_data['balance']}")
    
    # Convert string enums to enum objects
    if 'currency' in converted_data and isinstance(converted_data.get('currency'), str):
        try:
            converted_data['currency'] = Currency(converted_data['currency'])
        except ValueError:
            logger.warning(f"Invalid currency from DB: {converted_data['currency']}")
            converted_data['currency'] = None
    
    # Use model_construct to preserve enum objects

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mazerunner70) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
