---
trigger: always_on
description: このドキュメントは、Robert C. Martin（Uncle Bob）の**4層オニオンアーキテクチャ**とVladimir Khorikivの**単体テストの原則、実践、パターン**に従って構築されたTypeScript製testcliアプリケーションのアーキテクチャとテストガイドラインを説明しています。**関数モジュール**、**Command Query Separation (CQS)**、**Dependency Inversion Principle (DIP)** に重点を置いた、堅牢でテスト可能なEthereumトランザクション構築・署名・送信CLIアプリの作成を目標としています。
---

# testcli: アーキテクチャとテストガイドライン

このドキュメントは、Robert C. Martin（Uncle Bob）の**4層オニオンアーキテクチャ**とVladimir Khorikivの**単体テストの原則、実践、パターン**に従って構築されたTypeScript製testcliアプリケーションのアーキテクチャとテストガイドラインを説明しています。**関数モジュール**、**Command Query Separation (CQS)**、**Dependency Inversion Principle (DIP)** に重点を置いた、堅牢でテスト可能なEthereumトランザクション構築・署名・送信CLIアプリの作成を目標としています。

## プロジェクト概要

- **目的**: CLIアプリ（`testcli`）で、入力（例：`--to`, `--amount`, `--privateKey`）を受け取り、Ethereumトランザクションを構築・署名し、（最終的に）テストネットに送信する。
- **技術スタック**: TypeScript, Node.js, ethers.js, Vitest, commander.js。

## アーキテクチャ: 4層オニオンアーキテクチャ

このプロジェクトはUncle BobのClean Architectureに従い、`enterprise-business-rules`、`application-business-rules`、`interface-adapter`、`frameworks`の4層で構成されています。各層には特定の役割があり、依存関係は内側に向かって流れます（外側の層が内側の層に依存）。

interface-adapaters以上では、Nodeには依存しないが、TypeScript(JavaScript)の提供するオブジェクト等には依存してよい。

### 1. enterprise-business-rules（エンタープライズビジネスルール層）

- **役割**: 外部システムに依存しない純粋なビジネスロジック。ビジネスルール（例：トランザクション検証）を持つコアエンティティを含む。
- **例**: `Transaction`エンティティはアドレス、金額、ガス制限を検証する。
- **構成**: エンティティクラスとそのデータ構造インターフェースを含む。
- **重要な原則**: 外部依存なし（例：ethers.jsなし）。純粋なTypeScriptロジック。
- **テスト**: Vitestによる単体テスト、全てのエッジケース（例：無効入力）をカバー。

### 2. application-business-rules（アプリケーションビジネスルール層）

- **役割**: アプリケーション固有のロジック、ユースケースとアプリケーションモデルを含む。ビジネスルールをワークフローに編成。
- **例**: `buildTransaction`（クエリ）や`signTransaction`（コマンド）などのユースケース。
- **構成**: ユースケース関数、エンティティクラス、インターフェース定義を含む。
- **重要な原則**: シンプルさとテスタビリティのため**関数モジュール**（クラスではない）を使用。**CQS**に従う:
  - クエリ（例：`buildTransaction`）: データを返し、副作用なし。
  - コマンド（例：`signTransaction`）: 副作用を起こし、最小限のデータを返す。
- **テスト**: Vitestによる単体テスト、`application-business-rules/interfaces`の依存関係（例：`Provider`、`Wallet`）をモック。

### 3. interface-adapters（インターフェースアダプター層）

- **役割**: 内部ロジックと外部システムを橋渡しする。外部依存（例：ethers.js）の型をドメインモデルに変換するアダプターを含む。
- **例**: `adaptExternalTransactionReceiptToDomain`は外部ライブラリの`TransactionReceipt`を内部の`TransactionReceipt`エンティティに変換。
- **構成**: 型変換アダプター関数と外部システムのインターフェース定義を含む。
- **重要な原則**: 外部ライブラリに直接依存せず、独自のインターフェースを定義して変換処理を行う。

### 4. frameworks（フレームワーク層）

- **役割**: 外部システム（例：ethers.js、Infura、Node.js API）と相互作用。`application-business-rules/interfaces`のインターフェースを実装し、CLIエントリーポイントをホスト。
- **例**: `EthersInfuraProvider`は`Provider`を実装；`EthersWallet`は`Wallet`を実装；CLIは入力を解析。
- **構成**: CLIエントリーポイント、外部ライブラリの実装クラス、ユーティリティ関数を含む。
- **重要な原則**: 管理されていない依存関係（ethers.js、Infura）の薄いラッパー。最小限のロジック、ビジネスルールなし。
- **テスト**: 最小限の単体テスト（Khorikivの「管理されていない依存関係」）。モックされたインターフェース経由で統合テストでカバー。

## ディレクトリ構造

```plaintext
src/
├── enterprise-business-rules/
│   ├── entities/
│   ├── interfaces/
│   ├── errors/
│   └── __tests__/
├── application-business-rules/
│   ├── entities/
│   ├── usecases/
│   ├── interfaces/
│   ├── errors/
│   └── __tests__/
├── interface-adapters/
│   ├── adapters/
│   ├── repositories/
│   ├── errors/
│   └── __tests__/
└── frameworks/
    ├── commanders/
    ├── ethers/
    └── node/
```

## テストガイドライン（Khorikivの原則）

テストはKhorikivの『単体テストの原則、実践、パターン』に従い、堅牢で保守性の高いテストを確保します。

### 単体テスト

- **スコープ**: `enterprise-business-rules`（エンティティ）、`application-business-rules`（ユースケース）、`interface-adapters`（アダプター）。
- **原則**:
  - **ビジネス価値をテスト**: 高価値ロジック（例：無効トランザクションの防止）に焦点。
  - **管理されていない依存関係をモック**: Vitestの`vi.mock`を使用して`application-business-rules`インターフェース（例：`Provider`、`Wallet`）をモック。
  - **スタブとの相互作用は検証しない**: モック呼び出しではなく、出力（クエリ）または副作用（コマンド）を検証。
  - **CQS**: クエリは戻り値をテスト；コマンドは副作用をテスト。
- **例**（`src/application-business-rules/__tests__/build-transaction.test.ts`より）:

  ```typescript
  import { vi, describe, it, expect } from 'vitest'
  import { buildTransaction } from '../usecases/build-transaction'
  import { Provider } from '../interfaces/provider'

  describe('buildTransaction', () => {
    const mockProvider: Provider = {
      getBalance: vi.fn().mockResolvedValue(BigInt('1000000000000000000')),
      getFeeData: vi
        .fn()
        .mockResolvedValue({ maxFeePerGas: BigInt('1000'), maxPriorityFeePerGas: BigInt('100') }),
      getTransactionCount: vi.fn().mockResolvedValue(0),
      broadcastTransaction: vi.fn().mockResolvedValue({ hash: '0x123' }),
      getTransactionReceipt: vi.fn().mockResolvedValue(null),
    }
    it('throws for invalid address', async () => {
      await expect(buildTransaction('invalid', '1.5', mockProvider, '0x123')).rejects.toThrow(
        'Invalid address'
      )
    })
  })
  ```

- **カバレッジ目標**: `enterprise-business-rules`、`application-business-rules`、`interface-adapters`で70%（`npm run coverage`で実行）。

### 統合テスト

- **スコープ**: `enterprise-business-rules`、`application-business-rules`、`interface-adapters`の統合を検証（`frameworks`を除く）。
- **原則**:
  - **実装ではなくインターフェースをモック**: `frameworks`の`EthersInfuraProvider`や`EthersWallet`ではなく、`application-business-rules/interfaces`の`Provider`と`Wallet`をモック。
  - **管理されていない依存関係**: `frameworks`層（ethers.js、Infura）は信頼できないものとして扱い、常にモック。
  - **統合フローをテスト**: ユースケースとエンティティが連携して動作することを確認（例：トランザクションの構築と署名）。
- **例**（`tests/integration/transaction.integration.test.ts`より）:

  ```typescript
  import { vi, describe, it, expect } from 'vitest'
  import { buildTransaction } from '../../src/application-business-rules/usecases/build-transaction'

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oogatta) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
