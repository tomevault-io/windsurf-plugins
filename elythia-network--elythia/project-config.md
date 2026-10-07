---
trigger: always_on
description: このファイルは、このリポジトリで作業するClaude Code(と、それを使う人)のためのルール集です。
---

# CLAUDE.md

このファイルは、このリポジトリで作業するClaude Code(と、それを使う人)のためのルール集です。

## このプロジェクトについて

**Elythia(旧称mk-go)は、Misskeyとの互換性を保ったまま、独自の機能を育てているGo製のMisskey系サーバーです。** 改名の作業は#3180で進めています。関数名などコード上の識別子、`/api/meta`の`mkGoVersion` / `mkGoCommit`、環境変数の接頭辞`MK_`は、改名の後も据え置きます(例外はnodeinfoの宣言で、`mkGoPlugins`から`elythiaPlugins`に改めた。#3400)。

出発点は、Misskey(TypeScript/NestJS)のバックエンドをGoで書き換えるリライトでした。本家との互換を一通り満たした今は、次の方針で開発しています。

- **ActivityPubの連合の互換性は必ず守る。** 他のサーバーから見て、Misskeyと同じように振る舞う
- **REST APIは、本家のクライアントがそのまま動く互換性を保つ。** 独自の拡張は、フィールドやエンドポイントの**追加だけ**で行う。既存のものの意味を変えない
- **frontendはMisskeyのforkで、独自に手を入れてよい**
- **TS版Misskeyからの移行は保証する。** TS版のDBをそのまま引き継いで起動できるようにする。TS版へ戻せること(復路)は保証しない(#3191)。今どこまで戻れるかは`dropin-e2e`で測っており、戻らなくなったものは[docs/migration-from-ts.md](docs/migration-from-ts.md#戻らなくなったもの)に記録する
- 本家と意図的に違える挙動は、理由と一緒に[docs/divergence.md](docs/divergence.md)に記録する

読み方:

- **Section 0**の約10項目は必ず守ってください。以降の節は、その詳細と、踏むと壊れる罠です
- 各ルールには理由を1文だけ付けています。経緯や実測値は、リンク先の`docs/`にあります
- 関係するファイルを読むと、`.claude/rules/`から「先にこの`docs/`を読む」という指示が出ます(例: `*_test.go`を読むと`docs/testing.md`)。指示が出たら従ってください。指示はファイルを読んだときにだけ出るので、**新しいファイルを一から作るときは出ないことがあります**。そのときは、Section 4 / 7 / 8 / 9のリンク先を自分で開いてください。それ以外の`docs/`も、必要なときに自分で開いてください
- セクション番号(1〜10)は、コードやdocから「CLAUDE.md Section N」の形で参照されています。**番号を変えないでください**
- 運営者の手元の運用(本番環境など)に固有の話は、gitignore済みの`CLAUDE.local.md`に分けてあります。無くても作業できます

---

## 0. 最初に守ること

1. **コミット・push・PRのマージは、頼まれたときだけ行う。** 作業が終わったら、確認を求めて止まる
2. **作業はissueから始める。** issueの本文は、作成する前に文面を提示して確認を取る(Section 7)
3. **コミットの前に`make check`を通す。** fmt → vet → actionlint → golangci-lint → テスト(`-race`付き)の順で回る
4. **新しいテストは、変異させて落ちることを確かめる。** 直したコードを1行戻しても通るテストは、何も守っていない
5. **DBをモックしない。** DBを使うテストは`testutil.OpenTestDB`を通して、実PostgreSQLの専用schemaで動かす(Section 4)
6. **生成物を手で直さない。** 例: `docs/api-compat.md`、`go.work`
7. **`go mod tidy`を使わない。** 依存の追加は`go get`で行い、`GOWORK=off go build ./...`で充足を確かめる
8. **`name:`の無いcompose(`docker-compose.yml`など)は、ディレクトリ名がproject名になることに注意する。** 同じ名前のprojectが既に動いているホストで起動すると、そのコンテナに合流して作り直してしまう。検証用のcomposeには、必ず`name:`で専用の名前を付ける
9. **公開される場所に、要らない情報を書かない。** セッションURL、第三者のホスト、未修正の脆弱性の詳細が該当する(Section 7)
10. **数値・識別子・パスは、実行かgrepで確かめてから書く。** 推論で書いた数字は、高い確率で間違っている

---

## 1. 技術スタック

| 用途 | ライブラリ |
|---|---|
| 言語 | Go 1.27(`go.mod`で管理) |
| Web | Echo v4 |
| ORM | GORM + pgx/v5(PostgreSQL 18) |
| Migration | golang-migrate(SQLファイル) |
| 設定 | Viper(Misskey互換YAML + `MK_`環境変数) |
| ログ | slog |
| Redis | go-redis v9 |
| ジョブキュー | mkq(BullMQとwire互換) |
| 検索 | meilisearch-go |
| ストレージ | aws-sdk-go-v2/s3 |
| HTTP署名 / LD署名 | 自前実装(`internal/activitypub/`)+ `piprate/json-gold` |
| 認証 | bcrypt / pquerna/otp / go-webauthn |
| テスト | testing + testify + testcontainers-go(Redis用) |

## 2. 構成

```
cmd/elythia/    実行バイナリ`elythia`。`serve` / `migrate` / `backfill <名前>`などのサブコマンドで呼び分ける
internal/       本体。依存の向きは api → core → repository → model
  api/          APIハンドラ(エンドポイント単位のサブディレクトリ)
  core/         ビジネスロジック
  repository/   データアクセス
  model/        DBモデル
  entity/       レスポンス用DTO(ドメインロジックを入れない)
  activitypub/  連合(Inbox / Deliver / Renderer / Resolver / 署名)
  cli/          `elythia`のサブコマンドの処理(`cmd/elythia`はこれを呼ぶだけ)
  queue/ stream/ server/ config/ testutil/ ほか
plugin/         プラグインがimportする公開パッケージ
plugins/        プラグイン本体(gitignore済み。同梱するものだけ例外)
migration/      NNNNNN_name.up.sql / .down.sql
tests/          Goのe2e(`tests/e2e` / `tests/e2e-federation`)と、Go以外の検証基盤
frontend/       本家Misskeyから取り込んだfrontend(pnpm workspace、#3379)
tools/          parityゲートとコード生成のCLI
docs/           ドキュメント
```

- **依存は`api → core → repository → model`の向きだけにする。** 逆向きのimportは入れない
- `activitypub`は`core`から呼ぶ
- 全ディレクトリの説明は[docs/architecture.md](docs/architecture.md)にある

## 3. よく使うコマンド

```bash
make build / make dev        # ビルド / go runで起動
make fmt                     # gofmt -s -w(go env GOROOTのgofmtを使う)
make check                   # コミット前に必須
make test                    # CIと同じ条件(-race -count=1 -shuffle=3)
make test-fast               # -race抜き。反復用で、コミット前の検査ではない
make gates                   # 静的なparityゲートを一括(サーバー・Docker不要)
make plugin-test             # 同梱プラグインのテスト(別moduleなので./...に入らない)
make migrate-up / migrate-down   # downは1段だけ戻す
make frontend-check          # frontend/の型チェック + frontendを読むゲート + 絵文字の正規表現 + eslint
make frontend-lint / frontend-test   # frontend/のeslint / vitest
```

- 全targetは`make help`で見られる。説明は[docs/development.md](docs/development.md)にある
- **`make tidy`は使わない。** `plugins/`にプラグインを置いた環境では、生成される`cmd/elythia/plugins_generated.go`がプラグインのmoduleをimportするので失敗する。置いていない環境でも、CIと結果がずれる
- **Goの版を上げたら、`make plugins`で`go.work`を作り直す。** `go.work`は生成物で、作り直さないと古い版のtoolchainが選ばれ、ビルドが`requires go >= ...`で落ちる
- **`make uds-*`と`make docker-*`は運営者の環境向け。** 手元の検証には使わない

## 4. テスト

### ルール

- **新しい機能にはテストを付ける。** CIはパッケージごとにカバレッジ90%を要求する(下回るとマージできない)。通常のPRでは95%を、新しいパッケージや小さなパッケージでは100%を目指す
- 例外は`internal/api/admin`(80%)と、`internal/testutil` / `internal/server` / e2e(対象外)。理由は[docs/development.md](docs/development.md)にある
- テストは、対象と同じパッケージの`_test.go`に置く
- **単体テストは`internal/testutil`のモックを使う。統合テストは実DBを使う**

### DBを使うテスト

`testutil.OpenTestDB` / `MustOpenTestDB`は、**呼び出し元のパッケージ専用のPostgreSQL schema**に接続します。`go test`はパッケージを並行に実行し、CIのshardはDBを1つしか持たないためです。

- **DBを読み書きするテストで`OpenSharedTestDB`を使わない。** 接続処理そのものを試すテスト専用
- **`public.`のように、`search_path`をまたぐ生SQLを書かない**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Elythia-Network/elythia](https://github.com/Elythia-Network/elythia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
