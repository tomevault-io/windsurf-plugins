---
trigger: always_on
description: >
---


# Serverpod 3.4.x + Flutter Expert Skill

You are an expert in **Serverpod 3.4.x** (Dart backend framework) and **Flutter** (frontend). Always use Serverpod 3.4.x APIs, package versions `^3.4.0`, and modern Dart null-safety syntax.

> **Critical rule:** Always run `dart run serverpod_cli generate` after any `.spy.yaml` or endpoint change.

---

## 1. Project Structure

```
my_project/
├── my_project_server/          # Serverpod backend
│   ├── bin/
│   │   └── main.dart           # Entry point
│   ├── lib/
│   │   ├── src/
│   │   │   ├── endpoints/      # Endpoint classes (extend Endpoint)
│   │   │   ├── models/         # .spy.yaml model definitions
│   │   │   └── generated/      # Auto-generated — NEVER edit manually
│   │   └── server.dart
│   ├── migrations/             # Schema migration SQL files
│   ├── config/
│   │   ├── development.yaml
│   │   ├── staging.yaml
│   │   ├── production.yaml
│   │   └── passwords.yaml      # Secrets — NEVER commit to version control
│   └── pubspec.yaml
├── my_project_client/          # Generated Dart client — do NOT edit
└── my_project_flutter/         # Flutter frontend
    ├── lib/
    │   ├── src/
    │   │   ├── screens/
    │   │   ├── widgets/
    │   │   ├── providers/
    │   │   └── services/
    │   └── main.dart
    └── pubspec.yaml
```

### Project Creation (3.4.x)

```bash
# Install CLI
dart pub global activate serverpod_cli

# Create new project (full stack)
serverpod create my_project

# Create with specific template
serverpod create my_project --template server-only

# After creation, start Docker services then run
cd my_project/my_project_server
docker-compose up --build --detach
dart run bin/main.dart --apply-migrations
```

---

## 2. Models (.spy.yaml)

Models are defined in `lib/src/models/`. Run `dart run serverpod_cli generate` after every change.

### Basic Model

```yaml
# lib/src/models/user_profile.spy.yaml
class: UserProfile
table: user_profiles
fields:
  userId: int
  displayName: String
  bio: String?
  avatarUrl: String?
  metadata: Map<String, dynamic>?   # JSON field (3.4.x native support)
  createdAt: DateTime
  updatedAt: DateTime
indexes:
  user_profiles_user_id_idx:
    fields: userId
    unique: true
  user_profiles_display_name_idx:
    fields: displayName
```

### Enum Model

```yaml
# lib/src/models/order_status.spy.yaml
enum: OrderStatus
serialized: byName              # 3.4.x: serialize as string name, not index
values:
  - pending
  - paid
  - shipped
  - delivered
  - cancelled
```

### Model with Enum, JSON, and Composite Index

```yaml
# lib/src/models/order.spy.yaml
class: Order
table: orders
fields:
  userId: int
  status: OrderStatus
  totalCents: int
  shippingAddress: Map<String, dynamic>?  # JSON field
  tags: List<String>?                      # JSON array field
  createdAt: DateTime
  updatedAt: DateTime
indexes:
  orders_user_status_idx:
    fields: userId, status       # Composite index
  orders_created_idx:
    fields: createdAt
```

### One-to-Many Relationship

```yaml
# lib/src/models/post.spy.yaml
class: Post
table: posts
fields:
  authorId: int
  title: String
  content: String
  isPublished: bool, default=false
  publishedAt: DateTime?
  viewCount: int, default=0
indexes:
  posts_author_idx:
    fields: authorId
  posts_published_idx:
    fields: isPublished, publishedAt
```

```yaml
# lib/src/models/comment.spy.yaml
class: Comment
table: comments
fields:
  postId: int
  authorId: int
  body: String
  createdAt: DateTime
indexes:
  comments_post_idx:
    fields: postId
```

### Many-to-Many (Junction Table)

```yaml
# lib/src/models/post_tag.spy.yaml
class: PostTag
table: post_tags
fields:
  postId: int
  tagId: int
indexes:
  post_tags_unique_idx:
    fields: postId, tagId
    unique: true
```

---

## 3. Endpoints

Endpoint classes live in `lib/src/endpoints/` and extend `Endpoint`. Public methods are automatically exposed as RPC calls.

### Standard Endpoint Pattern

```dart
// lib/src/endpoints/post_endpoint.dart
import 'package:serverpod/serverpod.dart';
import '../generated/protocol.dart';

class PostEndpoint extends Endpoint {
  /// Public: no authentication required
  Future<List<Post>> getPublishedPosts(
    Session session, {
    int limit = 20,
    int offset = 0,
  }) async {
    return await Post.db.find(
      session,
      where: (t) => t.isPublished.equals(true),
      orderBy: (t) => t.publishedAt,
      orderDescending: true,
      limit: limit,
      offset: offset,
    );
  }

  /// Requires login — override requireLogin for the entire endpoint
  @override
  bool get requireLogin => true;

  Future<Post> createPost(
    Session session,
    String title,
    String content,
  ) async {
    if (title.trim().isEmpty) throw ArgumentError('Title cannot be empty');
    if (content.trim().isEmpty) throw ArgumentError('Content cannot be empty');

    final userId = await session.auth.authenticatedUserId;
    if (userId == null) throw ServerpodUnauthenticatedException();

    final now = DateTime.now().toUtc();
    final post = Post(
      authorId: userId,
      title: title.trim(),
      content: content.trim(),
      isPublished: false,
      createdAt: now,
      updatedAt: now,
    );
    return await Post.db.insertRow(session, post);
  }

  Future<Post?> getPost(Session session, int postId) async {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TOMOKI977/serverpod-flutter-expert](https://github.com/TOMOKI977/serverpod-flutter-expert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
