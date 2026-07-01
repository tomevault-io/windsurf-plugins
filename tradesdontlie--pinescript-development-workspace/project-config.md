---
trigger: always_on
description: *Derived from Pine Script v6 Extension analysis and [TradingView Pine Script v6 Reference](mdc:https:/www.tradingview.com/pine-script-reference/v6)*
---

# Comprehensive Pine Script v6 Rules & Best Practices

*Derived from Pine Script v6 Extension analysis and [TradingView Pine Script v6 Reference](mdc:https:/www.tradingview.com/pine-script-reference/v6)*

---

## 📋 Table of Contents

1. [Version & Script Declaration](mdc:#version--script-declaration)
2. [Type System & Keywords](mdc:#type-system--keywords)
3. [Variable Declaration & Assignment](mdc:#variable-declaration--assignment)
4. [Function Definition & Usage](mdc:#function-definition--usage)
5. [Object Lifecycle Management](mdc:#object-lifecycle-management)
6. [Array, Matrix & Map Operations](mdc:#array-matrix--map-operations)
7. [Pine Script v6 Specific Features](mdc:#pine-script-v6-specific-features)
8. [Common Error Patterns & Solutions](mdc:#common-error-patterns--solutions)
9. [Performance & Best Practices](mdc:#performance--best-practices)
10. [Syntax Rules & Line Continuation](mdc:#syntax-rules--line-continuation)

---

## 🔖 Version & Script Declaration

### **Rule V1.1: Version Declaration (MANDATORY)**
```pine
//@version=6
indicator("My Indicator", overlay=true)
```

**✅ Supported Versions:** `4`, `5`, `6`  
**🎯 Recommended:** Always use v6 for new scripts  
**📌 Pattern:** `^(\/\/)(@)(version)(=)([456])`

### **Rule V1.2: Script Type Declaration**
```pine
// Choose ONE of these script types:
indicator("Title", overlay=true)    // Technical analysis indicator
strategy("Title", overlay=true)     // Trading strategy with orders
library("Title")                    // Reusable functions for export
```

**🚨 Critical:** Script declaration must appear after version and before any code

---

## 🏷️ Type System & Keywords

### **Rule T1.1: Type Keywords & Forms**

| **Type** | **Usage** | **Example** |
|----------|-----------|-------------|
| `int`, `float`, `bool`, `string`, `color` | Basic types | `int myVar = 10` |
| `line`, `label`, `box`, `table`, `linefill` | Object IDs (always series) | `line myLine = line.new(...)` |
| `polyline`, `chart.point` | v6 Objects | `polyline pl = polyline.new()` |
| `array<type>` or `type[]` | Arrays | `array<float> arr = array.new_float()` |
| `matrix<type>` | Matrices | `matrix<float> mx = matrix.new_float()` |
| `map<keyType, valueType>` | Key-value maps | `map<string, float> m = map.new<string, float>()` |

### **Rule T1.2: Type Forms (Simple vs Series)**
```pine
// In exported library functions:
export emaRight(float source, simple int length) =>  // ✅ Correct
    ta.ema(source, length)

export emaWrong(float source, int length) =>         // ❌ Error
    ta.ema(source, length)  // length becomes 'series int', not supported
```

**🎯 Key Point:** Use `simple` for library function parameters that need compile-time constants

### **Rule T1.3: Type Consistency**
```pine
// ✅ Correct - same type in both branches
result = condition ? 1 : 2

// ❌ Error - type mismatch
result = condition ? 1 : 1.0  // int vs float
```

---

## 📝 Variable Declaration & Assignment

### **Rule V2.1: Assignment Operators**

| **Operator** | **Purpose** | **Usage** |
|--------------|-------------|-----------|
| `=` | Initial declaration | `myVar = 10` |
| `:=` | Reassignment | `myVar := 20` |
| `+=`, `-=`, `*=`, `/=`, `%=` | Compound operations | `myVar += 5` |

### **Rule V2.2: Variable Scope & Declaration**
```pine
// ✅ Correct declaration order
var globalVar = 0       // Global scope with 'var'
myVar = 0              // Declare first
myVar := myVar + 1     // Then reassign

// ❌ Error - undeclared identifier
myVar := myVar + 1     // Error: 'myVar' not declared
```

### **Rule V2.3: Explicit Typing Requirements**
```pine
// Required when initializing with 'na'
int myInt = na         // ✅ Explicit type needed
float myFloat = na     // ✅ Explicit type needed

// Optional for normal initialization
myInt = 10             // ✅ Type inferred
myFloat = 3.14         // ✅ Type inferred
```

---

## 🔧 Function Definition & Usage

### **Rule F1.1: Function Definition Syntax**
```pine
// Standard function definition
myFunction(param1, param2) =>
    result = param1 + param2
    result  // Return value (last expression)

// Function with default parameters
myFunction(param1, param2 = 10) =>
    param1 + param2

// Multi-line function with local variables
complexFunction(x, y) =>
    local1 = x * 2
    local2 = y * 3
    final = local1 + local2
    final
```

### **Rule F1.2: Function Parameter Types**
```pine
// Library export functions - specify type forms
export customTA(series float src, simple int len) =>
    ta.sma(src, len)

// Method definitions
method toString(simple int this) =>
    str.tostring(this)
```

### **Rule F1.3: Built-in Function Usage**
```pine
// ✅ Correct parameter order and types
line.new(x1, y1, x2, y2, color=color.blue, width=2)

// ✅ Check required vs optional parameters
request.security(symbol, timeframe, expression)  // All required

// ✅ Use namespaced functions correctly
ta.sma(close, 20)      // Technical analysis
math.max(high, low)    // Math operations
array.push(arr, val)   // Array operations
```

---

## 🗂️ Object Lifecycle Management

### **Rule O1.1: Object Creation & Deletion**
```pine
// ✅ Proper object lifecycle
if barstate.isconfirmed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tradesdontlie/pinescript-development-workspace](https://github.com/tradesdontlie/pinescript-development-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
