---
trigger: always_on
description: A Flutter news aggregator with AI summarization. Currently uses **mock repositories** simulating all backend operations (web scraping, LLM API calls, database). Real implementations are planned but not yet built.
---

# Smart Reader - AI Coding Instructions

## Project Overview
A Flutter news aggregator with AI summarization. Currently uses **mock repositories** simulating all backend operations (web scraping, LLM API calls, database). Real implementations are planned but not yet built.

## Architecture Pattern: Simplified Clean Architecture

```
lib/
├── core/                    # Shared resources across features
│   ├── constants/          # App-wide constants
│   ├── services/           # Cross-cutting services (PromptService, StorageService)
│   ├── theme/              # AppTheme, AppColors, AppTextStyles
│   └── widgets/            # Reusable widgets
└── features/
    ├── news/               # News aggregation & summarization
    │   ├── data/
    │   │   ├── models/     # Plain Dart classes with copyWith
    │   │   └── repositories/ # Abstract interfaces + Mock implementations
    │   └── presentation/
    │       ├── providers/  # Riverpod state management
    │       ├── screens/    # Full-page widgets
    │       └── widgets/    # Feature-specific components
    └── history/            # Reading history tracking
```

### Key Architectural Decisions

**Repository Pattern**: All data access goes through abstract repository interfaces (`NewsSourceRepository`, `NewsScraperRepository`, `LlmRepository`, `HistoryRepository`). Current implementations are prefixed `Mock*` and return hardcoded data with delays to simulate network calls.

**Data Models**: Immutable classes with `copyWith` for state updates. Key models:
- `NewsSource` - Sources user can toggle (TechCrunch, BBC, etc.)
- `NewsArticle` - Individual scraped articles with `briefSummary` and `summary`
- `GroupedNews` - Similar articles grouped together from multiple sources
- `SummaryHistory` - Saved fetch sessions with timestamp

**No Domain Layer**: Business logic lives directly in Riverpod Notifiers to keep the architecture simple for this app's scope.

## State Management: Riverpod 3.0

### Provider Patterns Used

**1. AsyncNotifier for Async Data** (see `news_source_provider.dart`, `history_provider.dart`)
```dart
class NewsSourceNotifier extends AsyncNotifier<List<NewsSource>> {
  @override
  Future<List<NewsSource>> build() async {
    // Initial async load
  }
  
  Future<void> toggleSource(String id) {
    // Mutate state with ref.read(repositoryProvider)
    state = AsyncValue.data(updatedList);
  }
}
```

**2. Notifier for Sync State** (see `news_feed_provider.dart`)
```dart
class NewsFeedNotifier extends Notifier<NewsFeedState> {
  @override
  NewsFeedState build() => const NewsFeedState();
  
  Future<void> fetchNews(List<NewsSource> sources) {
    state = state.copyWith(isLoading: true);
    // ... async work ...
    state = NewsFeedState(...);
  }
}
```

**3. Provider for Dependency Injection**
```dart
final newsScraperRepositoryProvider = Provider<NewsScraperRepository>((ref) {
  return MockNewsScraperRepository(); // Swap for real implementation later
});
```

### State Access in Widgets
- Use `ConsumerWidget` or `ConsumerStatefulWidget`
- Read state: `ref.watch(newsSourceNotifierProvider)`
- Call methods: `ref.read(newsSourceNotifierProvider.notifier).toggleSource(id)`

## Critical Workflows

### Running the App
```bash
flutter pub get
flutter run
```

### Testing (Not Yet Implemented)
Testability is built-in via repository abstractions. To add tests:
1. Mock repositories are already separated - use them in tests
2. Widget tests: Wrap with `ProviderScope` and override providers
3. Unit tests: Test Notifier logic by providing mock repositories

### Adding a New Feature
1. Create feature folder under `lib/features/`
2. Add `data/models/` and `data/repositories/` (abstract + mock)
3. Add `presentation/providers/` with Riverpod notifiers
4. Add `presentation/screens/` and `presentation/widgets/`
5. Register providers in the feature's provider file

## UI/UX Conventions

### Theme System
All colors come from `core/theme/app_colors.dart`. Text styles from `core/theme/app_text_styles.dart`. Never hardcode colors or text styles - always reference these constants.

**Primary Colors**: Deep Blue (`#2563EB`), Purple accent (`#7C3AED`)

### Loading States
Use `NewsLoadingShimmer` widget for skeleton loading. All async operations must show loading UI via `isLoading` state flags.

### Navigation
Currently uses direct `Navigator.push`. No named routes or routing package. Screens are instantiated with required parameters:
```dart
Navigator.push(context, MaterialPageRoute(
  builder: (context) => ArticleDetailScreen(article: article),
));
```

### Screen Structure
`HomeScreen` uses bottom navigation with 3 tabs. FAB triggers news fetch. See `home_screen.dart:_autoFetchNews()` for initial load pattern - uses `WidgetsBinding.instance.addPostFrameCallback` to fetch after first frame.

## Integration Points

### Future LLM Integration
`LlmRepository` defines 3 methods:
- `generateBriefSummary(content)` - 1-2 sentences for cards
- `generateDetailedSummary(content)` - Full summary for detail view
- `generateCombinedSummary(articles)` - Multi-article synthesis


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/senaogut) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
