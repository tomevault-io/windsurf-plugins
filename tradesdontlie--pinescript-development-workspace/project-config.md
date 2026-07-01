---
trigger: always_on
description: *Enhanced with Pine Script v6 Extension analysis and [TradingView Pine Script v6 Reference](https://www.tradingview.com/pine-script-reference/v6/)*
---

# Enhanced Pine Script v6 Rules 

*Enhanced with Pine Script v6 Extension analysis and [TradingView Pine Script v6 Reference](https://www.tradingview.com/pine-script-reference/v6/)*

---

## 🎯 Core Type System & Forms

### **Rule 1.1: Type Keywords & Forms**

| **Keyword** | **Meaning / Usage Rule** | **v6 Examples** |
|-------------|--------------------------|-----------------|
| `int`, `float`, `bool`, `string`, `color` | Explicitly declare base types when clarity needed or when initializing with `na` | `int myVar = na` |
| `line`, `label`, `box`, `table`, `linefill`, `polyline`, `chart.point` | Declare object IDs returned by their respective `*.new()` functions. Always **series form** | `polyline pl = polyline.new()` |
| `array<type>` or `type[]` | Declare an array holding elements of `type` | `array<float> arr = array.new_float()` |
| `matrix<type>` | Declare a 2D matrix of elements | `matrix<float> mx = matrix.new<float>(3,3)` |
| `map<keyType, valueType>` | Declare key-value map structure | `map<string, float> prices = map.new<string, float>()` |
| `simple` | Use in **exported library functions** to demand a *simple* (non‑series) argument | `export emaRight(float src, simple int len) => ta.ema(src,len)` |
| `series` | Implicit for most variables; rarely needs explicit statement unless contrasting with `simple` | `series float price = close` |

**Rule 1.2:** Prefer implicit typing unless compilation or readability demands explicit keywords.

**Rule 1.3:** Do **not** mix type forms inside the same expression; both branches of a conditional must resolve to the same type & form.

```pine
// ✅ Correct - same type forms
result = condition ? 1 : 2

// ❌ Error - mixing type forms  
result = condition ? 1 : 1.0  // int vs float mismatch
```

---

## 🔢 Array, Matrix & Map Handling

### **Rule 2.1: Array Operations (Enhanced for v6)**

```pine
// ✅ Array creation with explicit typing
var array<float> buffer = array.new_float(100, na)
var array<string> symbols = array.new_string(0, "DEFAULT")

// ✅ v6 Feature: Negative indices for array access
if array.size(prices) >= 2
    lastPrice = array.get(prices, -1)      // Last element
    secondLast = array.get(prices, -2)     // Second to last

// ✅ Safe array access pattern
safeArrayGet(arr, index) =>
    if array.size(arr) > math.abs(index)
        array.get(arr, index)
    else
        na
```

**Rule 2.2:** Arrays are **zero‑based** (`array.get(a,0)` is the first element).

**Rule 2.3:** Arrays are reference objects; changes through any reference affect the original.

**Rule 2.4:** **ALWAYS** check `array.size()` before reading to avoid runtime errors.

### **Rule 2.5: Matrix Operations (v6)**

```pine
// Matrix creation and operations
var matrix<float> priceMatrix = matrix.new<float>(rows=3, cols=3, initial_value=0.0)

// Matrix access and modification
matrix.set(priceMatrix, row=0, col=0, value=close)
currentPrice = matrix.get(priceMatrix, row=0, col=0)

// Matrix utility operations
rows = matrix.rows(priceMatrix)
cols = matrix.columns(priceMatrix)
```

### **Rule 2.6: Map Operations (v6)**

```pine
// Map creation and usage
var map<string, float> symbolPrices = map.new<string, float>()

// Map operations
map.put(symbolPrices, "AAPL", 150.0)
applePrice = map.get(symbolPrices, "AAPL")
hasApple = map.contains(symbolPrices, "AAPL")
```

---

## 📝 Assignment Operators

| **Operator** | **Purpose** | **v6 Usage** |
|--------------|-------------|--------------|
| `=`  | Initial declaration & assignment | `myVar = 10` |
| `:=` | Re‑assignment to an already declared identifier | `myVar := 20` |
| `+=`, `-=`, `*=`, `/=`, `%=` | Compound arithmetic updates; equivalent to `x = x op y` | `myVar += 5` |

**Rule 3.1:** Use `:=` *only* after variable declaration; never for first assignment.

```pine
// ✅ Correct declaration pattern
myVar = 0        // Initial declaration
myVar := myVar + 1   // Reassignment

// ❌ Error pattern  
myVar := myVar + 1   // Error: 'myVar' not declared
```

---

## 🔧 Core Language Operators

| **Operator** | **Notes** | **v6 Enhancements** |
|--------------|-----------|---------------------|
| `?:` (ternary) | Forms: `test ? a : b`. Chainable for *switch‑like* logic | Short-circuit evaluation improved |
| `[]` (series subscript) | Access historical values: `close[1]` is previous bar | Consistent with array negative indexing |
| `+` `-` `*` `/` `%` | Numeric math (element‑wise when inputs are series) | `+` also concatenates strings |
| `==` `!=` `>` `<` `>=` `<=` | Comparison; returns `bool` / `series<bool>` | Boolean optimization in v6 |

**Rule 4.1:** Zero, `NaN`, ±`Infinity` evaluate as *false* in boolean contexts.

**Rule 4.2:** Use short-circuit evaluation for performance in v6:

```pine
// ✅ v6 optimized boolean evaluation
if array.size(myArray) > 0 and array.first(myArray) > 0
    // array.first() only evaluated if size > 0
    process(array.first(myArray))
```

---

## 🎨 Function Calls & Parameters

### **Rule 5.1:** Supply arguments in the exact order shown in the official syntax.

```pine
// ✅ Correct parameter order and types
line.new(x1, y1, x2, y2, color=color.blue, width=2)

// ✅ v6 Dynamic requests
symbols = array.from("AAPL", "GOOGL", "MSFT")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tradesdontlie/pinescript-development-workspace](https://github.com/tradesdontlie/pinescript-development-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
