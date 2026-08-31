---
trigger: always_on
description: 本ファイルはエージェントが移行作業を実施する際に**必ず最初に読み込むドキュメント**である。
---

# AGENTS.md — SkiShop 移行エージェント指示書
## Java 5 / Struts 1.3 → Java 21 / Spring Boot 3.2.x

本ファイルはエージェントが移行作業を実施する際に**必ず最初に読み込むドキュメント**である。
全セクションを熟読した上で作業を開始すること。

---

## 1. 本プロジェクトの概要

| 項目 | 内容 |
|------|------|
| プロジェクト名 | SkiShop（スキー用品 EC サイト） |
| 移行元 | Java 5 / Struts 1.3.10 / Tomcat 6 / WAR パッケージ |
| 移行先 | Java 21 / Spring Boot 3.2.x / 組み込み Tomcat / JAR パッケージ |
| 移行先ディレクトリ | `appmod-migrated-java21-spring-boot-3rd/` |
| ベースパッケージ | `com.skishop` |
| DB | PostgreSQL（スキーマは変更しない／追加のみ可） |
| 詳細設計書 | `docs/migration/DESIGN.md` |
| 移行計画書 | `docs/migration/PLAN.md` |

**作業前に必ず参照するファイル一覧**:
1. `docs/migration/DESIGN.md` — アーキテクチャ・変換仕様・コード例
2. `docs/migration/PLAN.md` — フェーズ計画・チェックリスト・完了条件
3. `.github/instructions/java-coding-standards.instructions.md` — Java コーディング規約
4. `.github/instructions/security-coding.instructions.md` — セキュリティ規約
5. `.github/instructions/api-design.instructions.md` — Controller/API 設計規約
6. `.github/instructions/spring-config.instructions.md` — Spring 設定規約
7. `.github/instructions/pom-dependency.instructions.md` — 依存関係管理規約
8. `.github/instructions/test-standards.instructions.md` — テスト規約
9. `.github/instructions/dockerfile-infra.instructions.md` — Dockerfile / コンテナ設定規約
10. `.github/instructions/sql-schema-review.instructions.md` — SQL スキーマ・Flyway 規約

---

## 2. アーキテクチャの基本理解

### 2.1 レイヤー依存方向（絶対に逆転させない）

```
Controller → Service → Repository
                         ↓
                     JPA Entity (model/)
```

- **Controller は Repository を直接呼び出さない**
- **Service は Controller に依存しない**
- **Repository は Service/Controller に依存しない**

### 2.2 パッケージマッピング

| 移行元（Struts） | 移行先（Spring Boot） |
|----------------|---------------------|
| `web/action/*.java` | `controller/*.java` |
| `web/form/*.java` | `dto/request/*.java` (record クラス) |
| `service/**/*.java` | `service/*.java` |
| `dao/**/*.java` | `repository/*.java` (Spring Data JPA interface) |
| `domain/**/*.java` | `model/*.java` (JPA @Entity) |
| `common/util/PasswordHasher.java` | `util/PasswordHasher.java` (継続使用) |

### 2.3 移行数サマリー

| レイヤー | 数 | 移行先 |
|---------|-----|--------|
| Struts Action | 29 | Spring MVC Controller（8 クラスに集約） |
| ActionForm | 12 | Bean Validation 付き record DTO |
| DAO | 20 | Spring Data JPA Repository（interface） |
| Service | 13 | @Service クラス（CheckoutService 追加で 14） |
| Domain POJO | 22 | JPA @Entity |
| JSP テンプレート | 30+ | Thymeleaf テンプレート |

---

## 3. コーディング規約（必須遵守）

以下は `.github/instructions/java-coding-standards.instructions.md` の要点。
**詳細はインストラクションファイル本体を参照すること。**

### 3.1 命名規則

| 対象 | 規約 | 正しい例 | 誤った例 |
|------|------|---------|---------|
| クラス名 | PascalCase | `UserService`, `OrderController` | `user_service` |
| インターフェース | PascalCase（`I` プレフィックス不要） | `UserRepository` | `IUserRepository` |
| メソッド | camelCase + 動詞始まり | `findByEmail`, `createOrder` | `userSearch`, `data` |
| 変数 | camelCase + 意味のある名前 | `userName`, `orderItems` | `x`, `tmp`, `data` |
| 定数 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` | `maxRetryCount` |
| boolean | `is/has/can/should` プレフィックス | `isActive`, `hasPermission` | `active`, `checkPerm` |
| コレクション変数 | 複数形 | `users`, `orderItems` | `userList`, `items_arr` |

### 3.2 禁止事項チェックリスト（Critical / High）

| 優先度 | 禁止事項 | 代替手段 |
|--------|---------|---------|
| **Critical** | `System.out.println` / `System.err.println` | SLF4J `@Slf4j` + `log.info()` |
| **Critical** | `catch (Exception e) {}` 例外の握りつぶし | 必ずログ出力または再スロー |
| **Critical** | 秘密情報のハードコード（`password = "..."` 等） | 環境変数 `${DB_PASSWORD}` |
| **Critical** | 文字列結合による SQL 構築 | Spring Data JPA / `@Query` パラメータバインド |
| **Critical** | `new` による Service/Repository の生成 | Spring DI（コンストラクタインジェクション） |
| **High** | `@Autowired` フィールドインジェクション | コンストラクタインジェクション |
| **High** | `Optional.get()` の使用 | `orElseThrow()` / `orElse()` / `map()` |
| **High** | ログへの個人情報出力 | マスキングまたは出力しない |
| **High** | Controller が Repository を直接参照 | Service 経由 |
| **High** | `@Transactional` を Controller に付与 | Service 層のみに付与 |

### 3.3 Spring Boot DI 規約

```java
// ✅ 正しい: コンストラクタインジェクション (@RequiredArgsConstructor が推奨)
@Service
@RequiredArgsConstructor
public class AuthService {
    private final UserRepository userRepository;
    private final SecurityLogRepository securityLogRepository;
}

// ❌ 禁止: フィールドインジェクション
@Service
public class AuthService {
    @Autowired
    private UserRepository userRepository;  // 禁止
}
```

### 3.4 Java 21 の積極的活用

| 機能 | 適用場面 | コード例 |
|------|---------|---------|
| **record クラス** | DTO（リクエスト/レスポンス）の不変型定義 | `public record LoginRequest(String email, String password) {}` |
| **sealed クラス** | 決済結果などの限定型階層 | `sealed interface PaymentResult permits Success, Failure` |
| **switch 式** | status の文字列 → Enum 変換等 | `var label = switch (s) { case "A" -> ...; }` |
| **パターンマッチング** | instanceof チェック | `if (obj instanceof String s) { process(s); }` |
| **テキストブロック** | JPQL / HTML テンプレート文字列 | `""" SELECT u FROM User u WHERE ... """` |

### 3.5 @Transactional の使い方

| ケース | アノテーション |
|--------|-------------|
| 読み取り専用クエリ | `@Transactional(readOnly = true)` |
| 単一テーブル更新 | `@Transactional` |
| 複数テーブル更新（注文確定等） | `@Transactional`（Service メソッドで一括） |
| バッチ処理（新規トランザクション） | `@Transactional(propagation = Propagation.REQUIRES_NEW)` |

### 3.6 例外処理規約

```java
// ✅ 正しい例外クラス階層
ResourceNotFoundException  → HTTP 404
BusinessException          → HTTP 422（ビジネスルール違反）
AuthenticationException    → HTTP 401


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yoshioterada/Monolith-Struts-Sample](https://github.com/yoshioterada/Monolith-Struts-Sample) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
