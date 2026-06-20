---
trigger: always_on
description: |
---


## I. Composable Structure

### 1. Function Separation

Keep composable functions under ~50 lines of meaningful code (ignoring empty lines, braces-only lines).
Separate functions by responsibility: layout, input handling, dialogs, styling.
Mixing responsibilities is fine for small functions.

**Recomposition Scope**
Composable functions create a recomposition scope tied to their content. Deliberately creating small functions can isolate frequently-changing values into narrow scopes. Conversely, avoid unnecessary extraction when you only want readability (e.g. fixing detekt issues) — it creates an unneeded scope.

A single empty line can visually separate layout sections instead of comments — apply sparingly and only where readability suffers.

**Example — good separation for a large function:**
```kotlin
@Composable
private fun PostList(
    postsFeed: PostsFeed,
    favorites: Set<String>,
    showExpandedSearch: Boolean,
    onPostTapped: (postId: String) -> Unit,
    onToggleFavorite: (String) -> Unit,
    modifier: Modifier = Modifier,
    contentPadding: PaddingValues = PaddingValues(0.dp),
    state: LazyListState = rememberLazyListState(),
    searchInput: String = "",
    onSearchInputChanged: (String) -> Unit,
) {
    LazyColumn(
        modifier = modifier,
        contentPadding = contentPadding,
        state = state,
    ) {
        if (showExpandedSearch) {
            item {
                HomeSearch(
                    Modifier.padding(horizontal = 16.dp),
                    searchInput = searchInput,
                    onSearchInputChanged = onSearchInputChanged,
                )
            }
        }

        item { PostListTopSection(postsFeed.highlightedPost, onPostTapped) }

        if (postsFeed.recommendedPosts.isNotEmpty()) {
            item {
                PostListSimpleSection(postsFeed.recommendedPosts, onPostTapped, favorites, onToggleFavorite)
            }
        }

        if (postsFeed.popularPosts.isNotEmpty() && !showExpandedSearch) {
            item { PostListPopularSection(postsFeed.popularPosts, onPostTapped) }
        }

        if (postsFeed.recentPosts.isNotEmpty()) {
            item { PostListHistorySection(postsFeed.recentPosts, onPostTapped) }
        }
    }
}
```
`PostList` handles layout and delegates content rendering to child composables.

**Example — small function with mixed responsibilities (acceptable):**
```kotlin
@Composable
fun JetchatIcon(contentDescription: String?, modifier: Modifier = Modifier) {
    val semantics = if (contentDescription != null) {
        Modifier.semantics {
            this.contentDescription = contentDescription
            this.role = Role.Image
        }
    } else Modifier

    Box(modifier = modifier.then(semantics)) {
        Icon(painterResource(R.drawable.ic_jetchat_back), null, tint = MaterialTheme.colorScheme.primaryContainer)
        Icon(painterResource(R.drawable.ic_jetchat_front), null, tint = MaterialTheme.colorScheme.primary)
    }
}
```

### 2. Default Values for Arguments

1. Expose `Modifier` as the first optional argument with default `Modifier` for any composable with a single top-level composable call. Exception: screen-level composables don't need exposed `Modifier`.
2. No defaults for core data (e.g. `UserProfile` without `name` is useless).
3. No defaults for primary actions (e.g. `onClick` in `Button`).
4. No defaults in single-use private functions.
5. Optional arguments, secondary actions, and styling should have defaults in public/internal composables or in private composables already reused in the file.

```kotlin
// ✅ Correct
@Composable
fun AddTransactionScreen(
    onBack: () -> Unit = {},
    transactionId: Long? = null,
    viewModel: AddTransactionViewModel = koinViewModel(parameters = { parametersOf(transactionId) }),
) { /* Body */ }

@Composable
private fun DescriptionField(  // Used only once — no defaults
    state: TextFieldState,
    error: StringResource?,
    enabled: Boolean,
) { /* Body */ }

// ❌ Incorrect
@Composable
private fun TypeToggle(  // Single-use: `enabled` should not have a default (rule #4)
    isExpense: Boolean,
    enabled: Boolean = true,
    onToggle: (Boolean) -> Unit,
) { /* Body */ }

@Composable
internal fun UserScreen(
    viewModel: ViewModel = koinViewModel(),
    onNavigateBack: () -> Unit,  // Secondary action: should default to {} (rule #5)
) { /* Body */ }
```

### 3. Reusing Components

Check related directories for existing components before creating new ones. Components serve various purposes: common parameter configuration (e.g. `Header` = styled `Text`), layout patterns (e.g. `InfoCard`), or state/logic wrappers (e.g. animation containers).

**Component location:**
- **App-wide**: shared module or `components` package — used across the whole app.
- **Feature-wide**: `components` package in current module or a descriptively named file (e.g. `PostCards.kt`).
- **Screen-level**: same file as other screen code, or a nearby file if the screen file is large.

**Component extraction:**
- Look for recurring patterns with minor variations that can be parameterized.
- Place extracted components at the top-most common location for all usages.
- Expose `modifier: Modifier = Modifier` as the first optional parameter when the component is used in varied layout contexts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HornedHeck/compose-skill](https://github.com/HornedHeck/compose-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
