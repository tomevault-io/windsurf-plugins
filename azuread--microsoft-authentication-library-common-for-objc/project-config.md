---
trigger: always_on
description: This document provides code style guidelines that AI agents MUST follow when working with this codebase. These guidelines are adapted from industry best practices and tailored to match the existing Objective-C and Swift patterns in this repository.
---

# Objective-C and Swift Code Style Guidelines for AI Agents

## Overview

This document provides code style guidelines that AI agents MUST follow when working with this codebase. These guidelines are adapted from industry best practices and tailored to match the existing Objective-C and Swift patterns in this repository.

## Key Principles

### RFC 2119 Compliance

- **MUST**: Absolute requirement
- **MUST NOT**: Absolute prohibition
- **SHOULD**: Recommended but may have valid reasons to ignore
- **SHOULD NOT**: Not recommended but may have valid reasons to use
- **MAY**: Optional

---

## Code Style Rules

### 1. Dot Notation Syntax

**RECOMMENDED:** Use dot notation for getting and setting properties.

```objc
// Preferred
view.backgroundColor = UIColor.orangeColor;
NSString *username = account.username;

// Avoid
[view setBackgroundColor:[UIColor orangeColor]];
NSString *username = [account username];
```

### 2. Spacing and Indentation

**MUST** follow these spacing rules:

- Indentation: 4 spaces (never tabs)
- **Opening braces on NEW line** (repository convention)
- Closing braces on new line
- One blank line between methods

```objc
// Correct (as used in this repository)
- (instancetype)initWithUsername:(NSString *)username
                   homeAccountId:(MSIDAccountId *)homeAccountId
                     environment:(NSString *)environment
{
    self = [super init];
    
    if (self)
    {
        _username = username;
        _environment = environment;
        _homeAccountId = homeAccountId;
    }
    
    return self;
}

// For if/else statements
if (user.isHappy)
{
    // Do something
}
else
{
    // Do something else
}
```

### 3. Conditionals

**MUST** always use braces for conditional bodies, even for single-line statements.

```objc
// Correct
if (!error)
{
    return success;
}

// Incorrect - Never do this
if (!error)
    return success;

if (!error) return success;
```

### 4. Ternary Operator

**SHOULD** only evaluate a single condition per ternary expression.

```objc
// Acceptable
result = account.isValid ? account : nil;

// Avoid - too complex
result = account.isValid ? account.username = tenant.isValid ? tenant.id : nil : nil;
```

### 5. Error Handling

**MUST** check the return value, **MUST NOT** check the error variable directly.

```objc
// Correct
NSError *error;
if (![self trySomethingWithError:&error])
{
    // Handle Error
}

// Incorrect - Apple APIs may write garbage to error on success
NSError *error;
[self trySomethingWithError:&error];
if (error)
{
    // Handle Error
}
```

### 6. Method Signatures

**SHOULD** include space after scope symbol and between method segments.

```objc
// Correct
- (void)acquireTokenWithParameters:(MSIDSilentTokenParameters *)parameters
                   completionBlock:(MSIDCompletionBlock)completionBlock;

// For methods exceeding 80 characters, format like a form
- (MSIDResult *)resultWithTokenResult:(MSIDTokenResult *)result
                           authScheme:(id<MSIDAuthenticationSchemeProtocol>)authScheme
                           popManager:(MSIDDevicePopManager *)popManager
                                error:(NSError **)error;
```

### 7. Variables

#### Naming

**SHOULD** use descriptive variable names:

- `NSString *username` - clear and concise
- `NSString *accessToken` - describes the token type
- `MSIDAccount *currentAccount` - not just `account`
- `MSIDRequestParameters *requestParams` - abbreviated but clear
- `MSIDApplicationConfig *config` - clear context

**NOT RECOMMENDED:** Single letter variable names (except loop counters)

#### Pointer Asterisks

**MUST** attach asterisks to variable name:

```objc
// Correct
NSString *clientId

// Incorrect
NSString* clientId
NSString * clientId
```

Exception: Constants (`NSString * const MSIDErrorDomain`)

#### Properties vs Instance Variables

**SHOULD** use properties instead of naked instance variables.

```objc
// Preferred
@interface MSIDAccount : NSObject
@property (nonatomic) NSString *username;
@property (nonatomic) NSString *environment;
@end

// Avoid
@interface MSIDAccount : NSObject
{
    NSString *username;
    NSString *environment;
}
@end
```

**SHOULD** avoid direct instance variable access except in:

- Initializer methods (`init`, `initWithCoder:`)
- `dealloc` methods
- Custom setters and getters

#### Variable Qualifiers

**SHOULD** place ARC qualifiers between asterisks and variable name:

```objc
NSString * __weak weakReference;
MSIDAccount * __autoreleasing autoreleasedAccount;
```

### 8. Naming Conventions

#### Class Names and Constants

**MUST** use `MSID` prefix for classes and constants

```objc
// Correct
static const NSTimeInterval MSIDDefaultTokenRefreshInterval = 300.0;
static NSString * const MSIDInvalidTokenResultKey = @"MSIDInvalidTokenResultKey";

// Incorrect
static const NSTimeInterval refreshInterval = 300.0;
```

#### Properties and Local Variables

**MUST** be camelCase with lowercase leading word.

```objc
NSString *accessToken;
MSIDALAccount *currentAccount;
MSIDRequestParameters *requestParams;
```

#### Instance Variables


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AzureAD/microsoft-authentication-library-common-for-objc](https://github.com/AzureAD/microsoft-authentication-library-common-for-objc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
