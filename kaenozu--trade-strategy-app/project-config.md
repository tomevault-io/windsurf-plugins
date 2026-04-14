---
trigger: always_on
description: Streamlitベースの日本株トレード戦略アプリ。95銘柄のテクニカル分析・バックテスト・ウォークフォワード検証を行う。
---

# 日本株トレード戦略アプリ (trade-strategy-app)

Streamlitベースの日本株トレード戦略アプリ。95銘柄のテクニカル分析・バックテスト・ウォークフォワード検証を行う。

## プロジェクト構造

```
├── app.py                    # Streamlitダッシュボード（メインUI）
├── components/               # タブ別コンポーネント
│   ├── sidebar.py            # サイドバー設定UI
│   ├── dashboard.py          # ダッシュボードタブ
│   ├── dashboard_portfolio.py # ポートフォリオ表示
│   ├── charts.py             # チャート表示
│   ├── signals.py            # 戦略シグナル表示
│   ├── backtest_tab.py       # バックテストタブ
│   ├── risk_tab.py           # リスク管理タブ
│   ├── performance_tab.py    # パフォーマンスタブ
│   ├── constants.py          # UI定数（シグナル絵文字等）
│   └── __init__.py           # コンポーネントエクスポート
├── strategies/               # 個別戦略モジュール
│   ├── bb_reversal.py        # BB反転戦略
│   ├── trend_follow.py       # トレンドフォロー戦略
│   ├── breakout.py           # ブレイクアウト戦略
│   ├── mean_reversion.py     # 平均回帰戦略
│   ├── composite.py          # 総合判定（composite_score ラッパー）
│   ├── sector_rotation.py    # セクターローテーション戦略
│   └── __init__.py           # 戦略エクスポート
├── strategy_engine.py        # 戦略レジストリ・実行・シグナル取得
├── signal_utils.py           # シグナルユーティリティ（LatestSignal等）
├── backtest.py               # バックテスト（ATR損切り・利確・トレーリングストップ）
├── exit_logic.py             # 退出ロジック（損切り・利確・保有日数）
├── risk_manager.py           # ポジションサイジング（2%ルール・かぶミニ対応）
├── indicators.py             # テクニカル指標（SMA/RSI/MACD/BB/ATR/StochRSI/ADX）
├── config/                   # 全パラメータ・95銘柄リスト（パッケージ化）
│   ├── __init__.py           # 定数のエクスポート
│   ├── stocks.py             # 銘柄リスト・ユニバース・プレイブック
│   ├── trade_params.py       # トレードパラメータ
│   ├── indicators.py         # 指標パラメータ
│   └── data.py               # データ取得設定
├── data_fetcher.py           # yfinance経由OHLCV取得（parquetキャッシュ）
├── stock_screener.py         # 予算フィルタリング・銘柄スコアリング
├── portfolio_db.py           # SQLiteポートフォリオ管理
├── walk_forward.py           # ウォークフォワードテスト
├── market_regime.py          # マーケットレジーム検出
├── strategy_rotation.py      # 動的戦略選択
├── composite_scoring.py      # 総合スコアリング（Zスコア正規化）
├── portfolio_backtest.py     # ポートフォリオレベルバックテスト
├── portfolio_helpers.py      # ポートフォリオ内部ヘルパー
├── performance_reporting.py  # パフォーマンスレポート・実行ゲート
├── trade_utils.py            # トレードユーティリティ
├── daily_signal.py           # デイリーシグナル生成
├── operations.py             # 自動運用オペレーション
├── cli_report.py             # CLIレポート
├── tests/                    # テスト（805テスト・全パス）
│   ├── conftest.py           # テスト用フィクスチャ
│   ├── helpers.py            # テストヘルパー
│   ├── test_strategy_engine.py
│   ├── test_risk_manager.py
│   ├── test_stock_screener.py
│   ├── test_indicators.py
│   ├── test_data_fetcher.py
│   ├── test_walk_forward.py
│   ├── test_market_regime.py
│   ├── test_strategy_rotation.py
│   ├── test_backtest.py
│   ├── test_composite_scoring.py
│   ├── test_portfolio_backtest.py
│   ├── test_performance_reporting.py
│   ├── test_e2e_workflow.py
│   ├── test_benchmarks.py
│   ├── test_daily_signal.py
│   ├── test_performance_tab.py
│   ├── test_public_api.py
│   ├── test_sector_rotation.py
│   └── ...（他テストファイル）
├── scripts/                  # 分析・評価スクリプト（検証用）
│   ├── optimize_*.py         # 最適化スクリプト
│   ├── eval_*.py             # 評価スクリプト
│   ├── test_wf_*.py          # WF検証スクリプト
│   └── analyze_*.py          # 分析スクリプト
└── reports/                  # 最適化レポート・結果CSV
    ├── *_REPORT.md           # 分析レポート
    └── *_results.csv         # 結果データ
```

## 開発ルール

### 必須
- **mainへの直接プッシュ禁止**。全てPR経由でマージ。
- **変更前に必ずテスト実行**: `python -m pytest tests/ -v` → 全テスト通過必須
- **ウォークフォワード検証が基準**: 戦略変更は必ずOOS PnLで評価。IS成績だけで判断しない。
- 変更はブランチ→PR→レビュー→マージ

### コードスタイル
- コメントは不要（コードは自明に書く）
- 日本語のdocstringは既存パターンに従う
- 型ヒントは既存コードに合わせる
- importは `config/` の定数を活用（マジックナンバー禁止）

### テスト
- テストフレームワーク: pytest（850テスト・全パス）
- カバレッジ: 100%
- 全テストファイルは `tests/` に配置
- テスト実行: `python -m pytest tests/ -v`
- 高速化: `python -m pytest tests/ -q -n auto --dist=loadfile`

### ウォークフォワード検証
- `walk_forward.py` は `window_type` パラメータで `"expanding"`（デフォルト）と `"sliding"` をサポート
- ポートフォリオ制約検証: `test_wf_portfolio_constraints.py`
- ベースライン: BB反転 OOS PnL +49,915円（95銘柄・5年・5分割・sliding）
- 戦略ローテーション: OOS PnL +71,824円（+43.9%改善）
- 検証スクリプト: `scripts/verify_sliding_window_wf.py`（expanding vs sliding 比較）
- 改善案は個別にテストし、ベースラインを上回るもののみ採用

### レジーム連動戦略選択
- `strategy_rotation.py` でマーケットレジームに基づく戦略選択
- `portfolio_backtest.simulate_portfolio(strategy_key="strategy_rotation")` で銘柄ごとに最適戦略を自動選択
- レジーム判定: `market_regime.detect_regime()`（指数: ^N225、yfinance使用）
- バックテストタブ: `_get_rotation_result()` でローテーション結果を自動追加表示
- レジーム別推奨戦略:
  - `bull`: weekly_breakout, momentum, trend_follow, breakout, composite
  - `bear`: bb_reversal, weekly_mean_reversion, vix_timing
  - `range`: bb_reversal, composite, weekly_mean_reversion
  - `volatile`: composite, weekly_mean_reversion, mean_reversion, vix_timing
- スコアブースト: `REGIME_BOOST_FACTOR = 1.2`（推奨戦略のスコアを1.2倍）

## 戦略の現状

### 個別戦略 OOS成績（95銘柄・sliding・5y・5split）

| 戦略 | 個別OOS成績（95銘柄） | トレード数 | プラス銘柄率 | 状態 |
|---|---|---|---|---|
| BB反転（デフォルト） | +49,915円 | - | - | 推奨（ベースライン） |
| 平均回帰 | **+42,817円**（30銘柄V2） | - | - | 改善済み（V2） |
| ブレイクアウト | **+564,440円** | 662 | 63% | 改善済み |
| 総合判定 | **+399,075円** | 343 | 66% | 改善済み |
| トレンドフォロー | **+302,017円** | 450 | 65% | 改善済み |
| 週足ブレイクアウト | **+113,873円**（WF検証） | 982 | 62%（805/805） | 検証済み |
| セクターローテーション | **+138,915円**（95銘柄V2） | 1,224 | 64%（805/805） | 改善済み（V2・BB反転+178.3%） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaenozu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
