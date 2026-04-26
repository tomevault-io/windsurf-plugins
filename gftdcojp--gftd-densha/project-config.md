---
trigger: always_on
description: Densha Framework Naming Conventions - 命名規則ガイドライン
---


# Densha Framework 命名規則

このドキュメントは、Denshaフレームワーク全体で使用する命名規則を定義します。コードの可読性、一貫性、保守性を高めるためのガイドラインです。

## 1. 基本原則

### 1.1 一貫性 (Consistency)
- **英語で統一する**: すべての識別子は英語を使用
- **統一された用語を使用**: 同じ概念には同じ名前を使用
- **省略形を避ける**: 明確な単語を使用

### 1.2 明確性 (Clarity)
- **目的が伝わる名前**: 何をするものか一目でわかる
- **文脈を考慮**: スコープ内で一意性を確保
- **一般用語を優先**: ドメイン特化しすぎない

### 1.3 簡潔性 (Brevity)
- **冗長な接頭辞/接尾辞を避ける**: `create`, `get`, `set`などの多用を避ける
- **長すぎる名前を避ける**: 3-4単語以内に収める
- **意味のある短縮形を使用**: 広く認知された略語のみ使用

## 2. 型定義の命名規則

### 2.1 基本型 (Base Types)
```typescript
// ✅ 良い例
type User = { ... }
type Order = { ... }
type Product = { ... }

// ❌ 避ける例
type UserEntity = { ... }
type OrderData = { ... }
type ProductModel = { ... }
```

### 2.2 複合型 (Compound Types)
```typescript
// ✅ 良い例
type UserWithProfile = { ... }
type OrderStatus = 'pending' | 'confirmed' | 'shipped'
type ApiResponse<T> = { data: T; error?: string }

// ❌ 避ける例
type UserWithProfileData = { ... }
type OrderStatusType = 'pending' | 'confirmed' | 'shipped'
type ApiResponseResult<T> = { data: T; error?: string }
```

### 2.3 関数型 (Function Types)
```typescript
// ✅ 良い例
type Handler<T> = (input: T) => void
type Validator<T> = (value: T) => boolean
type Transform<I, O> = (input: I) => O

// ❌ 避ける例
type HandlerFunction<T> = (input: T) => void
type ValidationFunction<T> = (value: T) => boolean
type TransformationFunction<I, O> = (input: I) => O
```

## 3. 関数・メソッドの命名規則

### 3.1 基本関数 (Core Functions)
```typescript
// ✅ 良い例
const user = (config: UserConfig) => { ... }
const order = (config: OrderConfig) => { ... }
const product = (config: ProductConfig) => { ... }

// ❌ 避ける例
const createUser = (config: UserConfig) => { ... }
const makeOrder = (config: OrderConfig) => { ... }
const newProduct = (config: ProductConfig) => { ... }
```

### 3.2 ヘルパー関数 (Helper Functions)
```typescript
// ✅ 良い例
const attr = (schema: Schema, defaultValue?) => { ... }
const action = (config: ActionConfig) => { ... }
const policy = (fn: PolicyFn) => { ... }

// ❌ 避ける例
const createAttribute = (schema: Schema, defaultValue?) => { ... }
const defineAction = (config: ActionConfig) => { ... }
const createPolicy = (fn: PolicyFn) => { ... }
```

### 3.3 クエリ・操作関数 (Query/Operation Functions)
```typescript
// ✅ 良い例
const find = (query: Query) => { ... }
const filter = (predicate: Predicate) => { ... }
const sort = (comparator: Comparator) => { ... }

// ❌ 避ける例
const findByQuery = (query: Query) => { ... }
const filterWithPredicate = (predicate: Predicate) => { ... }
const sortByComparator = (comparator: Comparator) => { ... }
```

## 4. 変数・定数の命名規則

### 4.1 変数 (Variables)
```typescript
// ✅ 良い例
const user = createUser(config)
const orders = await fetchOrders(userId)
const isValid = validateEmail(email)

// ❌ 避ける例
const userEntity = createUser(config)
const orderList = await fetchOrders(userId)
const isEmailValid = validateEmail(email)
```

### 4.2 定数 (Constants)
```typescript
// ✅ 良い例
const MAX_RETRY_COUNT = 3
const API_TIMEOUT = 5000
const CACHE_TTL = 3600000

// ❌ 避ける例
const MAXIMUM_RETRY_COUNT = 3
const API_TIMEOUT_IN_MS = 5000
const CACHE_TIME_TO_LIVE = 3600000
```

## 5. クラス・インターフェースの命名規則

### 5.1 インターフェース (Interfaces)
```typescript
// ✅ 良い例
interface Handler<T> { ... }
interface Repository<T> { ... }
interface Service { ... }

// ❌ 避ける例
interface IHandler<T> { ... }
interface IRepository<T> { ... }
interface IService { ... }
```

### 5.2 クラス (Classes)
```typescript
// ✅ 良い例
class UserService { ... }
class OrderRepository { ... }
class ProductHandler { ... }

// ❌ 避ける例
class UserServiceImpl { ... }
class OrderDataAccessObject { ... }
class ProductHandlerManager { ... }
```

## 6. ファイルの命名規則

### 6.1 ファイル名 (File Names)
```typescript
// ✅ 良い例
user.ts
order.ts
product.ts
user-service.ts
order-repository.ts

// ❌ 避ける例
user-entity.ts
order-model.ts
product-data.ts
user-service-impl.ts
order-repository-dao.ts
```

### 6.2 ディレクトリ名 (Directory Names)
```typescript
// ✅ 良い例
domain/
  user.ts
  order.ts
  product.ts

infrastructure/
  user-repository.ts
  order-repository.ts

application/
  user-service.ts
  order-service.ts

// ❌ 避ける例
domain/
  user-entity.ts
  order-model.ts

infrastructure/
  user-repository-impl.ts
  order-repository-dao.ts

application/
  user-service-manager.ts
  order-service-handler.ts
```

## 7. 命名パターン一覧

### 7.1 許可されるパターン
```typescript
// 基本パターン
PascalCase: 型定義、クラス、インターフェース
camelCase: 関数、変数、メソッド、プロパティ
SCREAMING_SNAKE_CASE: 定数
kebab-case: ファイル名、ディレクトリ名

// 具体例
type User = { ... }                    // PascalCase
const user = (config) => { ... }       // camelCase
const MAX_RETRIES = 3                  // SCREAMING_SNAKE_CASE
user-service.ts                        // kebab-case
```

### 7.2 避けるべきパターン
```typescript
// ❌ 避けるパターン
type UserEntity = { ... }              // Entityサフィックス
const createUser = () => { ... }       // createプレフィックス
const userData = { ... }               // Dataサフィックス
interface IUser { ... }                // Iプレフィックス
class UserServiceImpl { ... }          // Implサフィックス
```

## 8. Densha Framework アーキテクチャ - Ash Framework完全対応

### 8.1 Ash Framework API対応表

DenshaはAsh FrameworkのAPI構造を可能な限り忠実に再現し、Effectによる高度な型安全とWASM Component Modelを統合しています。

#### 🎯 **Resources (主要概念)**
| Ash Framework | Densha | 対応状況 | 備考 |
|---------------|--------|----------|------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gftdcojp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
