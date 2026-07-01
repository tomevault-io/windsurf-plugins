---
trigger: always_on
description: Japanese furigana / TTS-prep engine。 Lindera + IPADIC + TOML データ駆動。
---

# ja-furigana lib (Rust)

Japanese furigana / TTS-prep engine。 Lindera + IPADIC + TOML データ駆動。

- **GitHub**: <https://github.com/RyuuNeko1107/ja-furigana>
- **crates.io**: `ja-furigana` (lib) + `ja-furigana-cli` (bin: `furigana`)
- **License**: MIT or Apache-2.0
- **MSRV**: Rust 1.89+

## 現 version + 進捗

- **LIVE**: `0.1.16` (2026-06-14、 0.1.0 cut は 2026-05-12)。 0.1.x patch は master HEAD から cut
  しており 0.2.0 開発分 (ScoringContext / ADR-0004 / bracket-accent parser / scoring pub(crate) 化) を
  含む実質 0.2.0-preview (semver は user 判断で 0.1.x 据え置き)。 直近 patch: 0.1.10 = **NameBoundaryPass**
  (人名+敬称の token 衝突補正、 ADR-0005 第 3 adapter)、 0.1.11 = **match_hits condition weighting**
  (match block の hit condition 数を literal=2/broad=1 で `Score::match_hits` に累積)、
  0.1.12 = **テスト品質刷新 + 実バグ修正一括** (CJK 拡張B〜H 漢字判定 [𠮷/𩸽 に furigana]、 IVS/異体字
  セレクタ除去、 漢数字 ≥100+助数詞 [三百回目→さんびゃくかいめ、 kansuji_to_arabic を百千万億 additive +
  〇 positional 一般化]、 romaji 無音脱落、 ruby 出力の区切り記号エスケープ、 matcher O(N²) 線形化、
  Mutex poison 回復、 reload preload、 enumeration 決定化。 mutation testing で corpus 非依存
  モジュールを網羅検証。 lib 491 + cli 48 test green、 corpus 1085/1085。 CHANGELOG 参照)。
  0.1.13 = **入力正規化を「lookup のみ正規化・表示 surface 原文保持」で配線** (compat/IVS/NFKC が
  alpha.15 以降 production 未配線だった regression。 `kana::normalize_text_aligned` で char 単位
  alignment を保持し解析後に surface/range を原文へ remap = `髙田→{髙田|たかだ}` / `３本→{３本|さんぼん}`、
  通常入力は fast path で挙動不変)。 0.1.14 = **compat を core_dict_dir からも読み込む**
  (`FuriganaBuilder` が rules_dir に加え core/user dict の role="compat" も補完、 defense-in-depth)。
  ※production の真の compat 修正は **dict 側で `compat.toml` を core/ → rules/ へ移動** (dict v2026.06.14、
  `load_rules_dir` が rules_dir 走査で拾うのでどの lib version でも有効)。
  0.1.15 = **複数文字 canonical の compat 完全展開** (旧字漢数字 `廿→二十` / `卅→三十` 等 5件、
  従来は先頭1文字で `廿→二`)、 0.1.16 = **多文字展開が token 分割された際の空 surface ruby 修正**
  (`卅→{卅|さん}{|じゅう}` を、 空 surface token の読みを直前 token に結合して `{卅|さんじゅう}` に。
  0.1.13 surface 保持機構の latent bug)。 lib 499 + cli 48 test green。
  既知の限界: `卅日` 等 旧字漢数字+助数詞は誤読しうる (展開後 十日=とおか と誤分割、 稀ケース)。
  本番 furigana-api は wrapper `2.0.13` で lib 0.1.16 稼働 (2026-06-14 deploy、 health gate ✓、
  dict v2026.06.14 swap 済で 髙田→たかだ / 廿→にじゅう / 卅→さんじゅう / ３本→さんぼん live 確認)
- **`0.2.0` (master 実装中・未 release)**: accent **core は実装済** =
  bracket notation parse + `AccentResult`/`AccentToken` + CLI `--mode=accent`
  (`api.rs::to_accent` / `scoring/bracket.rs`、 ADR-0003)。
  **残件**: UniDic pitch accent (aType) 統合のみ (loanwords は alpha.21、
  numeric_phrases は 2026-06-11 に再統合済)。 cut 判断は user 主導 (PROPOSALS/intonation.md)

## alpha.10 task 進捗 (2026-05-11)

### ✅ completed (16)

- A 系 (foundation): A1 schema_version validator + A1b caller wire-up (load_rules_dir / Dict / Loanwords / SingleOverrides の file load 経路に必須化、 lib fixture も `[meta] schema_version = "2"` 化、 dict 側の coordinate stamp は E1 / alpha.11 dict release 側責任) / A2 dict format 拡張 / A3 matcher
- B 系 (Smart engine core): B1 Viterbi DP / B2 band lexicographic / B3 (b)(c) penalty / B4 Engine 切替
- C 系 (cross-cutting):
  - C1 保護トークン抽出 (`scoring/special.rs` の ProtectTokenProvider、 band 2000)
  - C2 アルファベット passthrough (`scoring/special.rs` の AlphabetPassthroughProvider、 hit band 1000 / miss band 100)
  - C3 数字 + 助数詞 / 大数スケール / SI 単位 / 日付 / 時刻 / 記号 / 素の数字 (`scoring/numbers.rs` の NumberCandidateProvider、 band 950) + Furigana::analyze 5 provider 構成に統合
  - C4 踊り字 (`scoring/odoriji.rs` の OdorijiProvider + apply_rendaku post-pass、 既存 Strict 連濁 logic は kana::voice_first_kana に共通化) + postprocess 独立性 doc
- D: bracket forward compat (lib strip)
- F1: scoring/analyze.rs standalone API + Furigana::analyze() / CLI --mode analyze / HTTP mode=analyze (alpha.10 段階で ProtectToken / AlphabetPassthrough / DictBridge / NumberCandidate / Odoriji の 5 provider 構成、 loanwords / numeric_phrases 統合は今後)
- F2: `furigana-diff-engines` CLI tool

### ⏳ pending (2)

- E1: migration script Python (**dict repo 側 work**)
- ~~H1: alpha.10 release prep~~ (撤回、 alpha.10 release 自体 skip 方針、 alpha.11+ 一括 release で対応)

## 主要 module 構造

```
crates/furigana/src/
├── api.rs                 — Furigana / FuriganaBuilder (公開 entry、 解析は scoring/pipeline 経由の薄い層)
├── analyzer.rs            — Lindera + IPADIC ラッパー
├── char_class.rs          — 文字種 (CharType) 分類 + Unicode range 表の single home (kana/matcher/special が参照)
├── dict.rs                — jukugo / unihan / rich entry / [[kanji]] block 多重保持 (先頭 char prefix index 付)
├── kana.rs                — ひら⇄カタ変換 + 連濁 (voice_first_kana)。 判定 3 関数は char_class への公開 delegate
├── loader.rs              — TOML loader (schema_version validate)
├── numbers/               — kansuji / 助数詞 logic (scoring/numbers.rs から呼ばれる)
├── reading/               — 出力 layer (ReadingToken + tokens_to_hiragana / tokens_to_ruby)
├── romaji.rs              — ひらがな → ローマ字 (Hepburn / Kunrei)
├── rules/                 — counters / context / days / scales / postprocess 等 TOML data
├── scoring/               — Smart engine module (詳細 別記)
└── tts.rs                 — TTS pre-processing (pause 整形 等)

(旧 chunks/ / loanwords.rs / single_overrides.rs / reading/pipeline.rs は alpha.15 で削除済。
 loanwords は alpha.21 で AlphabetPassthroughProvider に再統合。)

crates/furigana-cli/src/
├── main.rs                — `furigana` バイナリ (CLI + HTTP server)
├── commands/              — lookup / repl / serve / dict subcommands

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RyuuNeko1107/ja-furigana](https://github.com/RyuuNeko1107/ja-furigana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
