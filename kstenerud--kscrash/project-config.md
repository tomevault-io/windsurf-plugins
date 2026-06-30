---
trigger: always_on
description: When performing code reviews on this repository, follow these instructions to identify API breaking changes in the KSCrash crash reporting library.
---

# GitHub Copilot Code Review Instructions for KSCrash

When performing code reviews on this repository, follow these instructions to identify API breaking changes in the KSCrash crash reporting library.

## Scope of Review

Only review changes to public API surfaces. The public modules are: KSCrashRecording, KSCrashFilters, KSCrashSinks, KSCrashInstallations, KSCrashDiscSpaceMonitor, KSCrashBootTimeMonitor, and KSCrashDemangleFilter. Only examine files in `Sources/[ModuleName]/include/*.h` directories as these contain the public headers.

## Critical Breaking Changes - Always Flag

### Method Parameter Changes
Flag ANY parameter addition, removal, or type changes to existing Objective-C methods. Objective-C has no default parameters, so even adding a nullable parameter breaks all existing call sites.

Examples of breaking changes:
```objc
// BREAKING: Adding parameter
- (void)method:(NSString *)existing;                                     // Old
- (void)method:(NSString *)existing newParam:(nullable NSString *)param; // New - BREAKING

// BREAKING: Parameter removal  
- (void)method:(NSString *)param1 param2:(NSString *)param2;    // Old
- (void)method:(NSString *)param1;                              // New - BREAKING

// BREAKING: Parameter type changes
- (void)method:(NSString *)param;    // Old
- (void)method:(NSArray *)param;     // New - BREAKING

// BREAKING: Parameter reordering
- (void)method:(NSString *)first second:(NSString *)second;    // Old
- (void)method:(NSString *)second first:(NSString *)first;     // New - BREAKING
```

### Callback Signature Changes
Flag any changes to callback or function pointer signatures including parameter addition, removal, reordering, or return type changes.

Examples of breaking changes:
```c
// BREAKING: Parameter addition
typedef void (*SomeCallback)(Writer *writer);                   // Old
typedef void (*SomeCallback)(Policy policy, Writer *writer);    // New - BREAKING

// BREAKING: Return type changes
typedef void (*SomeCallback)(Context *ctx);      // Old
typedef Policy (*SomeCallback)(Context *ctx);    // New - BREAKING
```

### Property Changes
Flag any property type changes or nullability changes in either direction.

Examples of breaking changes:
```objc
// BREAKING: Property type changes
@property (nonatomic, strong) NSString *prop;    // Old
@property (nonatomic, strong) NSArray *prop;     // New - BREAKING

// BREAKING: Nullability changes (both directions)
@property (nullable) NSString *prop;    // Old
@property (nonnull) NSString *prop;     // New - BREAKING (breaks code passing nil)

@property (nonnull) NSString *prop;     // Old
@property (nullable) NSString *prop;    // New - BREAKING (Swift API: String → String?)

// BREAKING: Property attribute changes
@property (atomic) id prop;       // Old
@property (nonatomic) id prop;    // New - BREAKING (ABI change)
```

### Swift API Changes via NS_SWIFT_NAME
Flag any addition, modification, or removal of NS_SWIFT_NAME attributes on existing types or methods.

Examples of breaking changes:
```objc
// BREAKING: Adding NS_SWIFT_NAME to existing type
@interface ExistingClass : NSObject                              // Old
@interface ExistingClass : NSObject NS_SWIFT_NAME(SwiftName)     // New - BREAKING

// BREAKING: Changing existing NS_SWIFT_NAME
NS_SWIFT_NAME(OldName) @interface MyClass : NSObject     // Old
NS_SWIFT_NAME(NewName) @interface MyClass : NSObject     // New - BREAKING

// BREAKING: Removing NS_SWIFT_NAME
NS_SWIFT_NAME(SwiftName) @interface MyClass : NSObject    // Old
@interface MyClass : NSObject                             // New - BREAKING

// BREAKING: Changing Swift parameter names
- (void)method:(NSString *)param NS_SWIFT_NAME(method(value:));    // Old
- (void)method:(NSString *)param NS_SWIFT_NAME(method(input:));    // New - BREAKING
```

### Struct and Enum Changes
Flag any struct or enum field reordering, removal, or type changes as these break binary compatibility.

Examples of breaking changes:
```c
// BREAKING: Field reordering
typedef struct {    // Old
    int field1;
    int field2;
} PublicStruct;

typedef struct {    // New - BREAKING
    int field2;     // Reordered!
    int field1;
} PublicStruct;

// BREAKING: Field type changes
typedef struct {
    int field;      // Old
} PublicStruct;

typedef struct {
    float field;    // New - BREAKING
} PublicStruct;

// BREAKING: Enum value changes
typedef enum {      // Old
    Value1 = 0,
    Value2 = 1,
} PublicEnum;

typedef enum {      // New - BREAKING
    Value1 = 1,     // Changed value!
    Value2 = 0,
} PublicEnum;
```

### Protocol Requirement Changes
Flag changes between required and optional protocol methods.

Examples of breaking changes:
```objc
// BREAKING: Adding required methods
@protocol PublicProtocol         // Old
- (void)existingMethod;
@end

@protocol PublicProtocol         // New - BREAKING
- (void)existingMethod;
- (void)newRequiredMethod;       // Added required method
@end

// BREAKING: Making optional methods required
@protocol PublicProtocol         // Old
- (void)existingMethod;
@optional
- (void)method;
@end

@protocol PublicProtocol         // New - BREAKING
- (void)existingMethod;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kstenerud/KSCrash](https://github.com/kstenerud/KSCrash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
