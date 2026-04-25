---
trigger: always_on
description: Project architecture - Design principles and structure
---


# Project Architecture - MyProject2

## Fundamental Principles

### 1. Data-Driven

**CRITICAL RULE:** All rules come from Data Tables or Data Assets, NEVER hardcoded in code.

**Implementation:**

- Use `UDataTable` for tabular data (races, classes, items)
- Use `UDataAsset` for complex configurations (character sheets)
- Validate data in editor with `UDataAsset::PostEditChangeProperty()`
- Use `FTableRowBase` for data structures

**Example:**

```cpp
// ✅ CORRECT - Data-Driven
UPROPERTY(EditDefaultsOnly, Category = "Data")
UDataTable* RaceDataTable;

// ❌ WRONG - Hardcoded
const float DwarfConstitutionBonus = 2.0f;
```

### 2. Modularity

**CRITICAL RULE:** Code organized by domain, each part with single and clear responsibility.

**Directory Structure:**

```
Source/MyProject2/
├── Characters/          # Characters and races
├── Components/          # Reusable components
│   ├── Data/           # Data components
│   ├── Features/        # Feature components
│   └── UI/             # UI components
├── Data/               # Data Assets and Data Tables
├── Gameplay/           # Game mechanics
└── Utils/              # Utilities
```

**Rules:**

- Each module must be independent
- Use interfaces for communication between modules
- Avoid circular dependencies
- One .h and .cpp file per class

### 3. Editor-Friendly

**CRITICAL RULE:** System must work perfectly in editor, allowing configuration without running the game.

**Implementation:**

- Use `WITH_EDITOR` for editor-specific code
- Validate data in `PostEditChangeProperty()`
- Use `UPROPERTY(EditAnywhere)` for editable properties
- Provide visual feedback in editor when possible

**Example:**

```cpp
#if WITH_EDITOR
void UCharacterSheetDataAsset::PostEditChangeProperty(FPropertyChangedEvent& PropertyChangedEvent)
{
    Super::PostEditChangeProperty(PropertyChangedEvent);
    ValidateData();
}
#endif
```

### 4. Multiplayer Preparation

**CRITICAL RULE:** System prepared for multiplayer from the start, don't refactor later.

**Implementation:**

- **ALWAYS** use `DOREPLIFETIME` for replicable properties
- **ALWAYS** validate RPCs with `WithValidation`
- **ALWAYS** execute authoritative logic on server
- **ALWAYS** use `GetLocalRole()` to verify authority
- **NEVER** trust client data without validation

**Example:**

```cpp
// ✅ CORRECT - Replication
void AMyCharacter::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);
    DOREPLIFETIME(AMyCharacter, CharacterData);
}

// ✅ CORRECT - RPC with validation
UFUNCTION(Server, Reliable, WithValidation)
void ServerTakeDamage(float DamageAmount);

bool ServerTakeDamage_Validate(float DamageAmount) { return DamageAmount >= 0.0f; }
void ServerTakeDamage_Implementation(float DamageAmount) { /* authoritative logic */ }
```

### 5. Separation of Responsibilities (Component-Based Architecture)

**CRITICAL RULE:** Each layer has single and well-defined responsibility.

#### Layer 1: Data Assets (Editor/Configuration)

**Responsibility:** Store static configuration, contains no logic.

**Characteristics:**

- Inherits from `UDataAsset`
- `UPROPERTY(EditDefaultsOnly)` - editable only in defaults
- Not replicable (not needed at runtime)
- Works only in editor for designers
- Example: `UCharacterSheetDataAsset`

**Pattern:**

```cpp
UCLASS(BlueprintType)
class MYPROJECT2_API UCharacterSheetDataAsset : public UDataAsset
{
    GENERATED_BODY()

public:
    UPROPERTY(EditDefaultsOnly, Category = "Character")
    ERace Race;

    UPROPERTY(EditDefaultsOnly, Category = "Character")
    EClass Class;

    // NEVER logic here, only data
};
```

#### Layer 2: Bridge Components (Rule Application)

**Responsibility:** Bridge between Data Asset and Runtime Component, apply rules.

**Characteristics:**

- Inherits from `UActorComponent`
- Executes only on server/local (not replicable)
- Applies race and class rules
- Loads data from Data Asset to Runtime Component
- Example: `UCharacterSheetComponent`

**Pattern:**

```cpp
UCLASS(ClassGroup=(Custom), meta=(BlueprintSpawnableComponent))
class MYPROJECT2_API UCharacterSheetComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintCallable, Category = "Character")
    void InitializeFromDataAsset(UCharacterSheetDataAsset* DataAsset);

    UFUNCTION(BlueprintCallable, Category = "Character")
    void ApplyRaceBonuses();

    UFUNCTION(BlueprintCallable, Category = "Character")
    void ApplyClassFeatures();

private:
    UPROPERTY()
    UCharacterSheetDataAsset* SourceDataAsset;
};
```

#### Layer 3: Runtime Data Components (Replicable Data)

**Responsibility:** Store character data at runtime, all properties replicable.

**Characteristics:**

- Inherits from `UActorComponent`
- All properties are replicable (`DOREPLIFETIME`)
- Calculates final attributes, HP, proficiency
- Prepared for future migration to GAS Attributes
- Example: `UCharacterDataComponent`

**Pattern:**

```cpp
UCLASS(ClassGroup=(Custom), meta=(BlueprintSpawnableComponent))
class MYPROJECT2_API UCharacterDataComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Stramp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
