---
trigger: always_on
description: This rule defines the validation patterns and error handling for the `FromEmailSelect` component used in event forms and other admin pages. The component provides a searchable dropdown for selecting tenant email addresses with comprehensive validation.
---

# From Email Select Component - Validation Rules

## **Overview**
This rule defines the validation patterns and error handling for the `FromEmailSelect` component used in event forms and other admin pages. The component provides a searchable dropdown for selecting tenant email addresses with comprehensive validation.

## **Problem Solved**
- **Database Value Validation**: Validates that `fromEmail` from database matches an email in the tenant email addresses list, clearing invalid values
- **Empty List Validation**: Detects when no email addresses are configured and shows appropriate error message
- **Empty Field Validation**: Validates that user has selected an email address before form submission
- **Custom Error Messages**: Provides user-friendly error messages instead of browser default validation
- **Parent Component Integration**: Notifies parent components about validation state for form-level validation

## **Component Location**
- **Component**: `src/components/FromEmailSelect.tsx`
- **Usage**: `src/components/EventForm.tsx` and other admin forms

## **Validation Rules**

### **1. Database Value Validation (Event Loading)**

**CRITICAL**: When loading event data from the database, the `fromEmail` value must be validated against the tenant email addresses list. If the database value doesn't exist in the list, it must be cleared so validation will catch it.

**Problem**: The database may contain an email address (e.g., `'events@example.com'`) that doesn't exist in the tenant email addresses list. This creates a conflict where:
- The form field appears empty (because `FromEmailSelect` can't display a value not in its list)
- But `form.fromEmail` still has the database value
- Validation passes incorrectly because the field has a value (even though it's invalid)

**Solution**: Validate the database value on event load and clear it if it doesn't exist in the email addresses list.

**Implementation Pattern:**
```tsx
// In EventForm useEffect when loading event data
useEffect(() => {
  if (event) {
    // CRITICAL: Validate fromEmail against the email addresses list
    // If the database value doesn't exist in the list, clear it so validation will catch it
    const validateAndSetFromEmail = async () => {
      let validFromEmail = event.fromEmail || '';

      // Only validate if fromEmail has a value
      if (validFromEmail && validFromEmail.trim() !== '') {
        try {
          // Fetch all email addresses (use a large page size to get all)
          const emailAddresses = await fetchTenantEmailAddressesServer(0, 1000);

          // Check if the fromEmail exists in the list and is active
          const emailExists = emailAddresses.some(
            email => email.emailAddress === validFromEmail && email.isActive === true
          );

          if (!emailExists) {
            // Email doesn't exist in the list - clear it so validation will catch it
            console.warn('[EventForm] fromEmail from database does not exist in email addresses list:', {
              fromEmail: validFromEmail,
              availableEmails: emailAddresses.map(e => e.emailAddress),
            });
            validFromEmail = '';
          }
        } catch (error) {
          // If fetching email addresses fails, log error but still clear the field
          // This ensures validation will catch it
          console.error('[EventForm] Failed to validate fromEmail against email addresses list:', error);
          validFromEmail = '';
        }
      }

      // Set form with validated fromEmail
      const formData = { ...defaultEvent, ...event, fromEmail: validFromEmail };
      setForm(formData);

      // ... rest of event loading logic (metadata, etc.)
    };

    // Call the async function to validate and set fromEmail
    void validateAndSetFromEmail();
  }
}, [event]);
```

**Key Requirements:**
- ✅ **Always validate** database `fromEmail` value against the email addresses list when loading event data
- ✅ **Clear invalid values** - if database value doesn't exist in list, set `form.fromEmail = ''`
- ✅ **Check active status** - only consider emails where `isActive === true`
- ✅ **Handle errors gracefully** - if fetching email addresses fails, clear the field to ensure validation catches it
- ✅ **Log warnings** - log when database value doesn't match list for debugging
- ✅ **Run before form initialization** - validation must complete before setting form state

**Why This Is Critical:**
- Prevents silent validation failures where form appears empty but has invalid value
- Ensures users must select a valid email from the dropdown
- Maintains data integrity by only allowing emails from the tenant email addresses list
- Provides clear validation feedback when database contains invalid data

### **2. Empty Email List Validation**

When the email address list is empty (length === 0), the component must:

- **Display Error Message**: Show a prominent error message below the input field
- **Error Message Text**: "The from email list is empty. Please contact Admin to add the list of from email addresses."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
