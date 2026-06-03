---
trigger: always_on
description: - **Don't assume, always verify** - Check existing code patterns, imports, and dependencies before making changes
---

# Cursor Rules for Pera Wallet Android

## Critical Rules - Always Follow

### Verification Before Changes
- **Don't assume, always verify** - Check existing code patterns, imports, and dependencies before making changes
- **Verify class types before extending** - Check if classes are `final`, `open`, or `abstract`
- **Verify constructor signatures before mocking** - Check actual parameters and types
- **Verify access modifiers** - Check `public`, `internal`, `private` before using functions/classes
- **Check for breaking changes** - Search for usages before modifying shared code
- **Check existing patterns** - Look for similar files to understand conventions

### Compilation & Quality
- **Code must compile** - Always verify code compiles before submitting
- **Never use @Suppress without approval** - Always ask user before adding any `@Suppress` annotation
- **Run linting before compilation**:
  1. `./gradlew :app:detektProdDebug --no-daemon 2>&1 | grep -E "(violation|error)"`
  2. `./gradlew :app:ktlintCheck --no-daemon 2>&1 | grep -E "(error|warning|FAILED|BUILD SUCCESS)"`
  3. `./gradlew :app:compileProdDebugKotlin --no-daemon 2>&1 | grep -E "(error:|FAILED|BUILD SUCCESS)"`
- **Optimize imports** - Remove duplicates, organize by groups before compiling
- **Run tests after changes**: `./gradlew :app:testProdDebugUnitTest`

### When Adding Sealed Type Variants
1. Search ALL usages first (`when` expressions, pattern matches)
2. Update all occurrences at once
3. Don't fix errors one by one

### Common ktlint Violations
- No blank lines before closing braces `}`
- No multiple consecutive blank lines
- Space after `//` in comments
- No trailing whitespace
- File name must match top-level class name
- Files must end with exactly one newline

## Refactoring Restrictions - Do NOT Change Without Request

- **Do NOT change data class to sealed interface**
- **Do NOT remove wrapper/delegation functions**
- **Do NOT change filter+map to mapNotNull**
- **Do NOT remove TODO placeholders** without implementing
- **Do NOT move data classes or companion objects**
- **Do NOT inject use cases directly when helper exists**
- **Do NOT change existing class dependencies**
- **Do NOT rename identifiers without request**
- **Do NOT remove string resources**
- **Do NOT simplify when expressions**

## Architecture Rules

### Module Boundaries (app ↔ common-sdk)
- **common-sdk data layer is internal** - API services, mappers, request/response models
- **common-sdk domain layer is public** - Repository interfaces and use cases
- **App module uses only common-sdk domain layer** - Never import data layer directly

### Layer Rules
- **Data models (Request/Response) must be internal**
- **Domain models do NOT use DTO suffix** - Use `JointAccount`, not `JointAccountDTO`
- **Use Input suffix for input models** - `CreateJointAccountInput`
- **Split interface and implementation into separate files**

### Fragment/ViewModel State Management
- **Never create state flags in Fragments** - State is lost on recreation. Move to ViewModel
- **Logic depending on state goes in ViewModel** - Including deep link handling, initialization flags
- **ViewModel calls stay in ViewModel** - Don't route events through Fragment back to ViewModel
  ```kotlin
  // BAD: Fragment routes event back to ViewModel
  override fun onEvent(event: Event) {
      viewModel.handleEvent(event)
  }

  // GOOD: Handle in Composable directly or use sealed ViewEvent
  LaunchedEffect(event) { when(event) { ... } }
  ```

### ViewState/ViewEvent Pattern
- **Use sealed interface ViewState** - Not data class with boolean flags
- **Use sealed interface ViewEvent** - For one-time navigation/UI events
- **Collect events in Fragment** - ViewEvents for navigation that requires Fragment context
- **Collect state in Composable** - ViewState for UI rendering
  ```kotlin
  sealed interface MyViewState {
      data object Loading : MyViewState
      data object Empty : MyViewState
      data class Content(val items: List<Item>) : MyViewState
      data class Error(val message: String) : MyViewState
  }

  sealed interface MyViewEvent {
      data class NavigateToDetail(val id: String) : MyViewEvent
      data class ShowError(val message: String) : MyViewEvent
  }
  ```

### UseCase Pattern
- Interface (fun interface) + `{UseCaseName}UseCase` implementation
- Use `operator fun invoke(...)` for functional interfaces
- **Avoid unnecessary implementations** - If just delegating to repository, provide via DI lambda:
  ```kotlin
  @Provides
  fun provideAddSignature(repo: Repository): AddSignature = AddSignature { id, input ->
      repo.addSignature(id, input)
  }
  ```

### Mapper Pattern
- Internal mappers: `internal interface FooMapper` + `internal class FooMapperImpl`
- **No DTO suffix on mappers** - Use `InboxSearchMapper`, not `InboxSearchDTOMapper`
- Use `with` scope function for cleaner code

### Cache vs Repository Naming
- **Use PersistentCacheProvider** for persistent storage, not SharedPreferences local sources
  ```kotlin
  // BAD: Custom SharedPreferences class
  class LastOpenedTimeLocalSource @Inject constructor(sharedPref: SharedPreferences)

  // GOOD: Use PersistentCacheProvider
  @Provides

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [perawallet/pera-android](https://github.com/perawallet/pera-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
