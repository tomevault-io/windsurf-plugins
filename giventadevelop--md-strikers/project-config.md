---
trigger: always_on
description: Standard pattern for form validation styling, error display, and scroll-to-error functionality across all forms (ProfileForm, EventForm, etc.)
---


# Form Validation Styling Pattern

## **Overview**
This rule defines the standard pattern for form validation styling, error display, and scroll-to-error functionality used across all forms in the application (ProfileForm, EventForm, etc.). This ensures consistent validation UX, error styling, and user feedback patterns.

## **Problem Solved**
- **Consistent Validation UX**: Ensures all forms use the same validation styling and error display patterns
- **Error Visibility**: Red borders, inline error messages, and error summary boxes provide clear feedback
- **Scroll-to-Error**: Automatically navigates users to the first error field on validation failure
- **Real-time Error Clearing**: Errors clear as users type, providing immediate feedback
- **Immediate Field Validation**: Required fields validate on blur (when user clicks outside), showing errors immediately without waiting for form submission
- **Professional Presentation**: Consistent error styling (red borders, red text, error icons) across all forms

## **Core Pattern**

### **1. State Management**

```tsx
// ✅ DO: Add validation state and field refs
import { useState, useRef } from "react";
import { flushSync } from "react-dom";

export default function FormComponent() {
  // Error state management
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [showErrors, setShowErrors] = useState(false);

  // Refs for form fields to enable scroll-to-error functionality
  const fieldRefs = useRef<Record<string, HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>>({});

  // ... rest of component
}
```

### **2. Scroll-to-Error Function**

```tsx
// ✅ DO: Add scroll-to-first-error function
const scrollToFirstError = (errorObj?: Record<string, string>) => {
  // Use provided errors or fall back to state
  const errorsToUse = errorObj || errors;
  const firstErrorField = Object.keys(errorsToUse)[0];
  if (firstErrorField && fieldRefs.current[firstErrorField]) {
    const field = fieldRefs.current[firstErrorField];
    // Scroll to field but DON'T focus it immediately
    // This allows all fields to show red borders before focusing
    field.scrollIntoView({
      behavior: 'smooth',
      block: 'center',
      inline: 'nearest'
    });
    // Delay focus slightly to ensure all fields have rendered with red borders
    setTimeout(() => {
      if (fieldRefs.current[firstErrorField]) {
        fieldRefs.current[firstErrorField]?.focus();
      }
    }, 100);
  }
};
```

### **3. Error Count Helper**

```tsx
// ✅ DO: Add helper function to get error count
const getErrorCount = () => Object.keys(errors).length;
```

### **4. Validation Function**

```tsx
// ✅ DO: Add validate() function with flushSync for immediate state updates
function validate(): boolean {
  const errs: Record<string, string> = {};

  // Required field validations
  if (!formData.firstName || formData.firstName.trim() === '') {
    errs.firstName = 'First name is required';
  }
  if (!formData.lastName || formData.lastName.trim() === '') {
    errs.lastName = 'Last name is required';
  }
  if (!formData.email || formData.email.trim() === '') {
    errs.email = 'Email is required';
  } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email.trim())) {
    errs.email = 'Please enter a valid email address';
  }

  // Additional validations (length, format, etc.)
  if (formData.title && formData.title.length > 250) {
    errs.title = 'Title must not exceed 250 characters';
  }

  // CRITICAL: Use flushSync to force immediate state update so red borders appear instantly
  const hasErrors = Object.keys(errs).length > 0;

  if (hasErrors) {
    // Force synchronous state updates so fields show red borders immediately
    flushSync(() => {
      setErrors(errs);
      setShowErrors(true);
    });

    // Scroll to first error field
    scrollToFirstError(errs);
  } else {
    setErrors({});
    setShowErrors(false);
  }

  return !hasErrors;
}
```

### **5. HandleChange Pattern (Error Clearing)**

```tsx
// ✅ DO: Clear errors when user starts typing
const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => {
  const { name, value, type } = e.target;
  const checked = (e.target as HTMLInputElement).checked;

  // Clear error for this field when user starts typing
  if (errors[name]) {
    setErrors(prev => {
      const newErrors = { ...prev };
      delete newErrors[name];
      return newErrors;
    });
  }

  // Update form data
  setFormData((prev) => ({
    ...prev,
    [name]: type === 'checkbox' ? checked : (value || ''),
  }));
};
```

### **6. Individual Field Validation (onBlur Pattern)**

```tsx
// ✅ DO: Create validateField function for individual field validation on blur
const validateField = (fieldName: keyof ValidationErrors) => {
  const newErrors: ValidationErrors = { ...errors };

  switch (fieldName) {
    case 'fieldName': {
      if (!formData.fieldName?.trim()) {
        newErrors.fieldName = 'Field name is required.';
      } else {
        delete newErrors.fieldName;
      }
      break;
    }

    case 'description': {
      if (!formData.description?.trim()) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
