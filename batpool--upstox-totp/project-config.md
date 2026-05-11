---
trigger: always_on
description: from pydantic import BaseModel, Field, ConfigDict, field_validator, model_validator
---


# Cursor Rules for Pydantic Python (Strict Mode)

# ====================
# Core Principles
# ====================
# - Always use Pydantic v2 (latest version)
# - Enable strict mode by default for type safety
# - Prefer explicit over implicit type definitions
# - Use type hints for all fields
# - Validate data at boundaries (API, database, external services)

# ====================
# Model Definition Rules
# ====================

from pydantic import BaseModel, Field, ConfigDict, field_validator, model_validator
from typing import Optional, List, Dict, Any, Union, Literal
from datetime import datetime, date
from decimal import Decimal
from enum import Enum

# Always use ConfigDict for configuration (Pydantic v2)
class StrictModel(BaseModel):
    model_config = ConfigDict(
        # Strict mode - no type coercion
        strict=True,
        
        # Validate on assignment
        validate_assignment=True,
        
        # Use enum values instead of names
        use_enum_values=True,
        
        # Forbid extra fields
        extra='forbid',
        
        # Validate default values
        validate_default=True,
        
        # Enable arbitrary types if needed
        arbitrary_types_allowed=False,
        
        # Freeze model after creation (immutable)
        frozen=False,  # Set to True for immutable models
        
        # JSON schema extras
        json_schema_extra={
            "example": {
                "field1": "value1",
                "field2": 123
            }
        }
    )

# ====================
# Field Definition Rules
# ====================

class UserModel(BaseModel):
    model_config = ConfigDict(strict=True, extra='forbid')
    
    # Required fields with no default
    id: int
    username: str
    
    # Optional fields with explicit None default
    email: Optional[str] = None
    
    # Fields with defaults
    is_active: bool = True
    
    # Fields with Field() for additional validation
    age: int = Field(gt=0, le=150, description="User's age")
    
    # Constrained strings
    password: str = Field(min_length=8, max_length=128)
    
    # Decimal for financial data
    balance: Decimal = Field(decimal_places=2, ge=Decimal('0'))
    
    # Datetime fields
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    # Lists with type constraints
    tags: List[str] = Field(default_factory=list, max_length=10)
    
    # Nested models
    profile: Optional['ProfileModel'] = None
    
    # Union types (be explicit)
    status: Union[Literal['active'], Literal['inactive'], Literal['pending']]

# ====================
# Validation Rules
# ====================

class ValidatedModel(BaseModel):
    model_config = ConfigDict(strict=True)
    
    email: str
    phone: Optional[str] = None
    age: int
    
    # Field validator (single field)
    @field_validator('email')
    @classmethod
    def validate_email(cls, v: str) -> str:
        if '@' not in v:
            raise ValueError('Invalid email format')
        return v.lower()
    
    # Multiple field validator
    @field_validator('phone')
    @classmethod
    def validate_phone(cls, v: Optional[str]) -> Optional[str]:
        if v is None:
            return v
        # Remove non-digits
        digits = ''.join(filter(str.isdigit, v))
        if len(digits) != 10:
            raise ValueError('Phone must be 10 digits')
        return digits
    
    # Model validator (cross-field validation)
    @model_validator(mode='after')
    def validate_model(self) -> 'ValidatedModel':
        if self.age < 18 and not self.phone:
            raise ValueError('Phone required for minors')
        return self

# ====================
# Enum Usage Rules
# ====================

class StatusEnum(str, Enum):
    ACTIVE = 'active'
    INACTIVE = 'inactive'
    PENDING = 'pending'

class EnumModel(BaseModel):
    model_config = ConfigDict(strict=True, use_enum_values=True)
    
    # Use enums for constrained choices
    status: StatusEnum
    
    # Optional enum
    priority: Optional[StatusEnum] = None

# ====================
# Error Handling Rules
# ====================

from pydantic import ValidationError

def safe_parse(data: dict, model: type[BaseModel]) -> Optional[BaseModel]:
    """Always handle ValidationError explicitly"""
    try:
        return model(**data)
    except ValidationError as e:
        # Log detailed error information
        for error in e.errors():
            print(f"Field: {error['loc']}, Error: {error['msg']}")
        return None

# ====================
# Serialization Rules
# ====================

class SerializableModel(BaseModel):
    model_config = ConfigDict(strict=True)
    
    id: int
    private_field: str = Field(exclude=True)  # Exclude from serialization
    
    # Custom serialization
    def model_dump_safe(self) -> dict:
        """Custom dump with specific options"""
        return self.model_dump(
            exclude_none=True,
            exclude_unset=True,
            by_alias=True,
            exclude={'private_field'}
        )

# ====================
# Type Conversion Rules (when strict=False selectively)
# ====================

class SmartModel(BaseModel):
    # Selective strict mode per field

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [batpool/upstox-totp](https://github.com/batpool/upstox-totp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
