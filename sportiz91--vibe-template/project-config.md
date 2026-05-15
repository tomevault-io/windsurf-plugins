---
trigger: always_on
description: handleSuccess(result)
---

Rule Name: coding-standards
Description: 
This rule defines the coding standards and formatting guidelines that must be followed for all code changes in this project.

<coding_format>

A. Syntax & Structure
- File names must be dash-case (word-cloud.service.ts) unless an existing pattern differs.
- Group imports: node/standard → npm packages → internal paths. No unused imports.
- Use arrow functions everywhere except inside class bodies, where concise method syntax is allowed.
- Prefer early returns; nested if/else blocks deeper than two levels are disallowed.
- Early returns must use block format with braces (e.g., `if (!value) { return }`) for readability.
- Extract function call results as scope variables before using in conditions (e.g., `const trimmedText = text.trim(); if (!trimmedText) {...}` instead of `if (!text.trim()) {...}`).
- Use async/await—never chain .then().
- No .forEach for side effects; use for (const x of arr) instead.
- Array combinators (map, reduce, filter) are allowed only when you return their result.
- Identifiers must be English.
- No commented code allowed.

B. Functional-Programming Rules
- Each function must:
    * Be ≤ 50 lines (preferably; extract helpers if longer).
    * Take ≤ 4 parameters (optional ones last).
    * Have a single responsibility.
    * Be pure unless it is an intentional I/O wrapper (e.g. DB write); such wrappers must be ≤ 15 lines.
    * Name functions with camelCase imperative verbs (calculateTotals, getUserById).

C. Type Safety & Error Handling
- Explicitly type all function parameters, return types, and exported constants.
- Type all local variables inside a function.
- **Special attention for async operations**: Variables from awaited functions (e.g., `const { userId } = await auth()`) must be explicitly typed, especially in Next.js components where auth results should use proper domain types.
- No any; if an external library forces it, wrap and narrow.
- Error handling in catch blocks:
  - If the error variable is not used, use `catch {}` (no parameter).
  - If the error is used, type it as `unknown` and handle it safely within the catch block.

D. React Component Standards

- Always define props with interfaces, never inline types
- Place interfaces directly above component definitions
- Use const arrow functions for component definitions
- Use implicit return syntax when components only return JSX (no logic before return)
- Export components using export default pattern (required for Next.js pages/layouts)
- Handler functions inside components must be ≤ 20 lines and have a single, clear responsibility. Extract helper functions for complex logic.

  **Wrong (~50 lines in one handler):**
  ```tsx
  const handleFormSubmit = async (): Promise<void> => {
    const trimmedName: string = formData.name.trim()
    const trimmedEmail: string = formData.email.trim()
    const trimmedMessage: string = formData.message.trim()
    
    if (!trimmedName) {
      setErrors({ ...errors, name: "Name is required" })
      toast({ title: "Error", description: "Name is required", variant: "destructive" })
      return
    }
    
    if (!trimmedEmail || !trimmedEmail.includes("@")) {
      setErrors({ ...errors, email: "Valid email is required" })
      toast({ title: "Error", description: "Valid email is required", variant: "destructive" })
      return
    }
    
    if (!trimmedMessage || trimmedMessage.length < 10) {
      setErrors({ ...errors, message: "Message must be at least 10 characters" })
      toast({ title: "Error", description: "Message too short", variant: "destructive" })
      return
    }
    
    setIsSubmitting(true)
    setErrors({})
    
    try {
      const payload: FormPayload = {
        name: trimmedName,
        email: trimmedEmail,
        message: trimmedMessage,
        timestamp: new Date().toISOString()
      }
      
      const response: Response = await fetch("/api/contact", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
      })
      
      if (!response.ok) {
        throw new Error("Failed to submit")
      }
      
      const result: SubmissionResult = await response.json()
      
      setFormData({ name: "", email: "", message: "" })
      setSubmissionCount(prev => prev + 1)
      
      toast({ title: "Success", description: "Message sent successfully!" })
      
      if (onSuccess) {
        onSuccess(result)
      }
    } catch (error: unknown) {
      const errorMessage: string = error instanceof Error ? error.message : "Unknown error"
      console.error("Submission error:", errorMessage)
      setErrors({ submit: "Failed to send message" })
      toast({ title: "Error", description: "Failed to send message", variant: "destructive" })
    } finally {
      setIsSubmitting(false)
    }
  }
  ```

  **Good (broken into focused helpers ≤ 20 lines each):**
  ```tsx
  const validateForm = (): boolean => {
    const trimmedName: string = formData.name.trim()
    const trimmedEmail: string = formData.email.trim()
    const trimmedMessage: string = formData.message.trim()
    
    if (!trimmedName) {
      setErrors({ ...errors, name: "Name is required" })
      toast({ title: "Error", description: "Name is required", variant: "destructive" })
      return false
    }
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sportiz91/vibe-template](https://github.com/sportiz91/vibe-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
