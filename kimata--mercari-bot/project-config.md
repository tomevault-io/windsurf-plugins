---
trigger: always_on
description: - 変更前に意図と影響範囲を説明し、ユーザー確認を取る
---

# CLAUDE.md
### 共通運用ルール

- 変更前に意図と影響範囲を説明し、ユーザー確認を取る
- `pyproject.toml` 等の共通設定は `../py-project` で管理し、各リポジトリで直接編集しない
- `my_lib` の変更は `../my-py-lib` で実施し、各リポジトリのハッシュ更新後に `uv lock && uv sync` を実行
- 依存関係管理は `uv` を標準とし、他の手段はフォールバック扱い
- 構造化データは `@dataclass` を優先し、辞書からの生成は `parse()` 命名で統一
- Union 型が 3 箇所以上で出現する場合は `TypeAlias` を定義
- `except Exception` は避け、具体的な例外型を指定する
- ミラー運用がある場合は primary リポジトリにのみ push する


このファイルは Claude Code がこのリポジトリで作業する際のガイダンスを提供します。

## プロジェクト概要

mercari-bot は、メルカリに出品中のアイテムの価格を自動的に値下げするボットです。Selenium WebDriver を使用してメルカリにログインし、お気に入り数やアイテムの価格に応じて戦略的な価格調整を行います。

### 主な機能

- お気に入り数に応じた値下げ戦略
- 複数プロファイル（アカウント）対応
- LINE 経由のログイン認証
- CAPTCHA 対応（音声認識）
- Slack/メール通知

## 開発コマンド

```bash
# 依存関係のインストール
uv sync

# ヘルプ表示
uv run python src/app.py -h

# デバッグモード（価格変更は行わない）
uv run python src/app.py -D

# 通常実行（ログ通知付き）
uv run python src/app.py -l

# 設定ファイル指定
uv run python src/app.py -c custom-config.yaml

# Docker で実行（推奨）
docker compose run --build --rm mercari-bot

# テスト実行
uv run pytest

# 型チェック
uv run mypy src/
uv run pyright src/
```

## アーキテクチャ

```
src/
├── app.py                          # エントリポイント（docopt CLI）
└── mercari_bot/
    ├── config.py                   # 設定読み込み・型定義（dataclass）
    └── mercari_price_down.py       # 値下げ処理ロジック
```

### 処理フロー

1. `app.py`: 設定読み込み → 各プロファイルに対して `mercari_price_down.execute()` を実行
2. `mercari_price_down.py`:
    - `my_lib.store.mercari.login.execute()` でログイン
    - `my_lib.store.mercari.scrape.iter_items_on_display()` で出品中アイテムを取得
    - `_execute_item()` で各アイテムの値下げ処理

### 値下げロジック

- 更新から一定時間経過したアイテムのみ対象（`interval.hour` で設定）
- お気に入り数に応じて値下げ幅を決定（`discount` で設定）
- 最低価格（`threshold`）を下回る場合はスキップ
- 価格は 10 円単位に丸める

## コーディング規約

### インポートスタイル

`from xxx import yyy` は基本的に使用せず、`import xxx` としてモジュールをインポートし、参照時は `xxx.yyy` と完全修飾名で記述する：

```python
# 推奨
import my_lib.selenium_util
import my_lib.store.mercari.login

driver = my_lib.selenium_util.create_driver(...)

# 非推奨
from my_lib.selenium_util import create_driver

driver = create_driver(...)
```

これにより、関数やクラスがどのモジュールに属しているかが明確になり、コードの可読性と保守性が向上する。

例外: dataclass や型定義は直接インポート可

```python
from mercari_bot.config import AppConfig, ProfileConfig
```

### 型アノテーションと型情報のないライブラリ

型情報を持たないライブラリを使用する場合、大量の `# type: ignore[union-attr]` を記載する代わりに、変数に `Any` 型を明示的に指定する：

```python
from typing import Any

# 推奨: Any 型を明示して type: ignore を不要にする
result: Any = some_untyped_lib.call()
result.method1()
result.method2()

# 非推奨: 大量の type: ignore コメント
result = some_untyped_lib.call()  # type: ignore[union-attr]
result.method1()  # type: ignore[union-attr]
result.method2()  # type: ignore[union-attr]
```

これにより、コードの可読性を維持しつつ型チェッカーのエラーを抑制できる。

### pyright エラーへの対処方針

pyright のエラー対策として、各行に `# type: ignore` コメントを記載して回避するのは**最後の手段**とする。

**優先順位：**

1. **型推論できるようにコードを修正する** - 変数の初期化時に型が明確になるようにする
2. **型アノテーションを追加する** - 関数の引数や戻り値、変数に適切な型を指定する
3. **Any 型を使用する** - 型情報のないライブラリの場合（上記セクション参照）
4. **`# type: ignore` コメント** - 上記で解決できない場合の最終手段

```python
# 推奨: 型推論可能なコード
items: list[str] = []
items.append("value")

# 非推奨: type: ignore の多用
items = []  # type: ignore[var-annotated]
items.append("value")  # type: ignore[union-attr]
```

**例外：** テストコードでは、モックやフィクスチャの都合上 `# type: ignore` の使用を許容する。

### CLI

- docopt を使用
- エントリポイントは `src/app.py`
- `if __name__ == "__main__":` で引数解析

### 設定

- YAML 形式（`config.yaml`）
- JSON Schema で検証（`schema/config.schema`）
- `mercari_bot/config.py` で `dataclass` として型定義

### パス設定の扱い

設定ファイルで相対パスを指定する場合、`my_lib.config.resolve_path()` を使用して設定ファイルの場所を基準とした絶対パスに解決する：

```python
import my_lib.config

# config.yaml の場所を基準にパスを解決
selenium_path = my_lib.config.resolve_path(raw_config, data["selenium"])
```

DataConfig には str ではなく pathlib.Path を格納し、使用側での変換を不要にする。

### my_lib との型整合性

my_lib が特定の型（dataclass 等）を使用する場合、mercari-bot 側でもその型を使用する。これにより IDE の補完が効き、型安全性が向上する。

```python
# 推奨: my_lib の型を使用
from my_lib.store.mercari.config import MercariItem

def on_item_start(self, index: int, total: int, item: MercariItem) -> None:
    name = item.name  # 属性アクセス

# 非推奨: dict[str, Any] で受け取る
def on_item_start(self, index: int, total: int, item: dict[str, Any]) -> None:
    name = item.get("name", "不明")  # キーアクセス
```

TYPE_CHECKING ブロックで条件付きインポートすることで、実行時のインポートを避けつつ型チェックを有効にできる：

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from my_lib.store.mercari.config import MercariItem

def process_item(item: MercariItem) -> None:
    ...
```

## 設定ファイル

`config.example.yaml` を参考に `config.yaml` を作成：

```yaml
profile:
    - name: Profile 1
      line:
          user: LINE ユーザ ID
          pass: LINE パスワード
      discount:
          - favorite_count: 10 # お気に入り数が10以上
            step: 200 # 200円値下げ
            threshold: 3000 # 3000円が下限
          - favorite_count: 0 # デフォルト
            step: 100
            threshold: 3000
      interval:
          hour: 20 # 20時間以内に更新済みならスキップ

# オプション: Slack 通知
slack:
    bot_token: xoxp-...
    from: Mercari Bot
    info:
        channel:
            name: "#mercari"
    captcha:
        channel:
            name: "#captcha"
            id: XXXXXXXXXXX
    error:
        channel:
            name: "#error"
            id: XXXXXXXXXXX
        interval_min: 180
```

## 依存関係

### 主要ライブラリ

- `my-lib`: 共通ライブラリ（Selenium 操作、メルカリログイン、通知）
- `selenium`: ブラウザ自動化
- `undetected-chromedriver`: 検出回避付き Chrome ドライバ
- `pydub` / `speechrecognition`: CAPTCHA 音声認識

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kimata) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
