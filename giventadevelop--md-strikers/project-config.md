---
trigger: always_on
description: Standards for displaying numerical and currency values with consistent decimal places
---


# Numerical & Currency Value Display Standards

## **Overview**
This rule defines the standard pattern for displaying numerical and currency values across the application, ensuring consistent decimal place formatting (e.g., `$0.80` instead of `$0.8`, `$10.00` instead of `$10`).

## **Problem Solved**
- **Consistent Currency Display**: Ensures all currency values display with exactly 2 decimal places
- **Professional Presentation**: Prevents confusion from inconsistent decimal formatting
- **User Experience**: Users see consistent, professional monetary values throughout the application

## **Core Rule**

### **Currency Display Standard**
- **Rule:** Always display currency values with exactly 2 decimal places.
- **Purpose:** Ensures consistent and professional presentation of monetary values.
- **Implementation:** Use `Intl.NumberFormat` with `minimumFractionDigits: 2` and `maximumFractionDigits: 2`.

## **Currency Formatting Function**

### **Standard Implementation**
```typescript
// ✅ DO: Format currency to 2 decimal places
const formatCurrency = (amount: number, currency: string = 'USD') => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
    minimumFractionDigits: 2,  // Always show 2 decimal places
    maximumFractionDigits: 2,  // Never show more than 2
  }).format(amount);
};

// Examples:
formatCurrency(0.8, 'USD');   // Returns: "$0.80"
formatCurrency(10, 'USD');     // Returns: "$10.00"
formatCurrency(99.9, 'USD');   // Returns: "$99.90"
formatCurrency(100.5, 'USD');  // Returns: "$100.50"
```

### **Common Usage Patterns**

#### **Display Price in Cards/Tables**
```tsx
// ✅ DO: Use formatCurrency for all price displays
<span className="text-4xl font-bold">
  {formatCurrency(plan.price, plan.currency)}
</span>

// ❌ DON'T: Display raw price value
<span className="text-4xl font-bold">
  ${plan.price}  // May show "$0.8" instead of "$0.80"
</span>
```

#### **Price Input Field Formatting**
```tsx
// ✅ DO: Format price input display value
const [displayPrice, setDisplayPrice] = useState<string>('');

useEffect(() => {
  if (formData.price !== undefined && formData.price !== null) {
    setDisplayPrice(formData.price.toFixed(2));
  }
}, [formData.price]);

<input
  type="number"
  name="price"
  value={displayPrice}
  onChange={(e) => {
    const numValue = parseFloat(e.target.value) || 0;
    handleChange({ target: { name: 'price', value: numValue } });
  }}
  onBlur={(e) => {
    const numValue = parseFloat(e.target.value) || 0;
    setDisplayPrice(numValue.toFixed(2)); // Format to 2 decimal places on blur
  }}
  step="0.01"
  placeholder="0.00"
/>
```

#### **Display Price Values in Tables**
```tsx
// ✅ DO: Format price for display in tables
<td>
  {new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: plan.currency,
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  }).format(plan.price)}
</td>

// ❌ DON'T: Display raw price value
<td>${plan.price}</td>  // May show "$0.8" instead of "$0.80"
```

## **Non-Currency Numerical Values**

### **Decimal Numbers (Percentages, Rates, etc.)**
For non-currency numerical values that require decimal precision:

```typescript
// ✅ DO: Use appropriate decimal places for context
const formatPercentage = (value: number) => {
  return new Intl.NumberFormat('en-US', {
    style: 'percent',
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  }).format(value / 100);
};

// Examples:
formatPercentage(85.5);  // Returns: "85.50%"
formatPercentage(100);   // Returns: "100.00%"
```

### **Whole Numbers (Counts, Quantities)**
For whole numbers that don't require decimal places:

```typescript
// ✅ DO: Use no decimal places for counts
const formatCount = (count: number) => {
  return new Intl.NumberFormat('en-US', {
    minimumFractionDigits: 0,
    maximumFractionDigits: 0,
  }).format(count);
};

// Examples:
formatCount(10);   // Returns: "10"
formatCount(1000); // Returns: "1,000"
```

## **Input Field Best Practices**

### **Price Input Fields**
- **Display Value**: Use string state to allow free typing (e.g., `displayPrice: "0.80"`)
- **Numeric Value**: Store actual numeric value separately (e.g., `formData.price: 0.8`)
- **On Blur Formatting**: Format to 2 decimal places when user leaves the field
- **Step Attribute**: Use `step="0.01"` for price inputs
- **Zero Prefix Validation**: Automatically prefix decimal values starting with `.` with `0` (e.g., `.70` becomes `0.70`)

```tsx
// ✅ DO: Format price input with 2 decimal places and zero prefix validation
const [displayPrice, setDisplayPrice] = useState<string>('');

useEffect(() => {
  if (editingItem) {
    setDisplayPrice(editingItem.price ? editingItem.price.toFixed(2) : '0.00');
  } else {
    setDisplayPrice('');
  }
}, [editingItem]);

<input
  type="text"
  name="price"
  inputMode="decimal"
  value={displayPrice}
  onChange={(e) => {
    const inputValue = e.target.value;

    // Update display value immediately to allow free typing
    setDisplayPrice(inputValue);

    // CRITICAL: If value starts with a decimal point (e.g., ".70"), prefix with "0"
    let processedValue = inputValue;
    if (inputValue.startsWith('.')) {
      processedValue = '0' + inputValue;
      setDisplayPrice(processedValue);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
