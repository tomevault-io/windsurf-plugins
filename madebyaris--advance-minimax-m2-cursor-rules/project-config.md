---
trigger: always_on
description: Flutter development: widget patterns, state management, Dart best practices, and platform channel integration
---


# Flutter Development Patterns

Modern Flutter patterns for cross-platform mobile, web, and desktop development.

## Flutter Workflow

Before changing Flutter code:

```text
1. Read `pubspec.yaml` and the current app structure first
2. Check the repo's Flutter and Dart constraints before using newer language or framework features
3. For new apps, use `flutter create`; do not hand-create `pubspec.yaml`
4. For new dependencies or version-sensitive work, verify current versions with the actual current date
```

### CLI-First Flutter Development

Prefer Flutter CLI workflows:
```bash
# Project creation (NEVER manually create pubspec.yaml)
flutter create my_app
flutter create --org com.example my_app
flutter create --template package my_package

# Add dependencies (NEVER manually edit pubspec.yaml for adding)
flutter pub add provider
flutter pub add go_router
flutter pub add flutter_bloc
flutter pub add dio
flutter pub add freezed --dev
flutter pub add build_runner --dev

# Get dependencies after any pubspec change
flutter pub get

# Code generation (for freezed, json_serializable)
dart run build_runner build --delete-conflicting-outputs

# Verify project health
flutter analyze
flutter test
```

### Post-Edit Verification

After meaningful Flutter changes, run the smallest useful check for the task:

```bash
flutter analyze
flutter test
dart format --set-exit-if-changed .
```

Run `flutter pub get` when `pubspec.yaml` changed rather than after every code edit.

### Common Dart/Flutter Syntax Traps (Avoid These!)

```dart
// WRONG: Missing const for immutable widgets
Widget build(BuildContext context) {
  return Container(  // Should be const Container()
    child: Text('Hello'),
  );
}

// CORRECT: Use const where possible
Widget build(BuildContext context) {
  return const Container(
    child: Text('Hello'),
  );
}

// WRONG: Not disposing controllers
class _MyWidgetState extends State<MyWidget> {
  final controller = TextEditingController();
  // Missing dispose!
}

// CORRECT: Always dispose controllers
class _MyWidgetState extends State<MyWidget> {
  final controller = TextEditingController();
  
  @override
  void dispose() {
    controller.dispose();
    super.dispose();
  }
}

// WRONG: Using setState after dispose
void _onDataLoaded(data) async {
  await fetchMore();
  setState(() {  // Might be called after dispose!
    this.data = data;
  });
}

// CORRECT: Check mounted before setState
void _onDataLoaded(data) async {
  await fetchMore();
  if (mounted) {
    setState(() {
      this.data = data;
    });
  }
}

// WRONG: Missing required in named parameters (Dart 3+)
void greet({String name}) { }  // Error in null-safe Dart

// CORRECT: Use required for non-nullable required params
void greet({required String name}) { }
```

---

## Widget Fundamentals

### StatelessWidget
```dart
class UserCard extends StatelessWidget {
  const UserCard({
    super.key,
    required this.user,
    this.onTap,
  });

  final User user;
  final VoidCallback? onTap;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        leading: CircleAvatar(
          backgroundImage: NetworkImage(user.avatarUrl),
        ),
        title: Text(user.name),
        subtitle: Text(user.email),
        onTap: onTap,
      ),
    );
  }
}
```

### StatefulWidget
```dart
class Counter extends StatefulWidget {
  const Counter({super.key, this.initialValue = 0});

  final int initialValue;

  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  late int _count;

  @override
  void initState() {
    super.initState();
    _count = widget.initialValue;
  }

  void _increment() {
    setState(() {
      _count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: _increment,
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

### Widget Composition
```dart
// Prefer composition over inheritance
class ProfilePage extends StatelessWidget {
  const ProfilePage({super.key, required this.user});

  final User user;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(user.name)),
      body: SingleChildScrollView(
        child: Column(
          children: [
            ProfileHeader(user: user),
            ProfileStats(user: user),
            ProfileActions(user: user),
          ],
        ),
      ),
    );
  }
}
```

---

## State Management

### Provider Pattern
```dart
// Model
class CartModel extends ChangeNotifier {
  final List<Item> _items = [];

  List<Item> get items => List.unmodifiable(_items);
  
  int get totalItems => _items.length;
  
  double get totalPrice => _items.fold(0, (sum, item) => sum + item.price);

  void add(Item item) {
    _items.add(item);
    notifyListeners();
  }

  void remove(Item item) {
    _items.remove(item);
    notifyListeners();
  }

  void clear() {
    _items.clear();
    notifyListeners();
  }
}

// Provider setup
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CartModel(),
      child: const MyApp(),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m2-cursor-rules](https://github.com/madebyaris/advance-minimax-m2-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
