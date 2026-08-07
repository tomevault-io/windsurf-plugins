---
trigger: always_on
description: > 読み「きゅーびー」(PC-98 = きゅうはち → Q + Bee)。内部コードネーム/旧称は **QB**。
---

# QuuBee - PC-98 フリーソフト文化のブラウザプレイヤー

> 読み「きゅーびー」(PC-98 = きゅうはち → Q + Bee)。内部コードネーム/旧称は **QB**。
> コード識別子 (`qb_*` / `QB_*` / `qbDebug`) と `.qb` フォーマット拡張子は QB のまま据え置き
> (巨大リファクタ回避・継続性のため)。プロダクト表記のみ QuuBee。

**ミッション**: PC-98 のフリーソフト文化を、罪悪感なく継承・再体験できるプレイヤー。NEC BIOS も
MS-DOS も使わず（HLE-DOS + 合成 BIOS + MIT の NP2kai）、書庫ドロップで即プレイ。

## 仕様書（必読）
- コンセプト（魂・現行）: [docs/concept.md](docs/concept.md)
- HLE-DOS の実 DOS との差異・未対応: [docs/dos_hle_gaps.md](docs/dos_hle_gaps.md)
- 仕様書本体 (Notion): https://www.notion.so/msonrm/QB-v2-Wasm-PC-98-36740929a47081878a5fd6740a97ada5

## 現在のフェーズ
**Phase 1 完了 ✓** — NP2kai を Emscripten で Wasm ビルドし、FreeDOS(98) がブラウザで起動することを確認

**Phase 2 進行中** — PC-98 ゲームディスクのロードと動作確認
- ✓ CPU を i386c (NP21) へ拡張、FPU 有効化（2026-06-26 にライセンス整合のため DOSBox2(GPL)→SoftFloat3(BSD) へ切替）
- ✓ 標準キーボード入力（英数, 記号, 矢印, F1-F10, テンキー）
- ✓ 表示パイプラインのピクセルパーフェクト化
- ✓ 自己起動最小ディスクが画面表示まで動作 (`tools/boot_hello/`)
- ✓ ディスクの D&D / ファイル選択 UI、A:/B: 2 ドライブ対応 (B: はリセットなし)
- ✓ マウス入力 (Pointer Lock + 相対移動 + 左右ボタン)
- ✓ 実 PC-98 のゲームディスク (.d88) で CPU/FDD・CG 経路を検証（Phase 2 のブリングアップ。市販ソフトの
  動作は射程外なので、テストスイートは同人/フリーソフトに統一 — 末尾参照）
- ✓ サウンド対応 (FM 音源、AudioWorklet + postMessage、メインスレッドジャンク耐性あり)
- ✓ HDD スロット (C:/D:、SASI/IDE) と `np2kai_insert_hdd` ブリッジ — UI 配線 OK だが
  DOS 系 HDD イメージは BIOS ホールで起動できない (FreeDOS と同じ壁)
- 次: 追加タイトル検証、PC-98 固有キー、GitHub Actions CI

**Phase 3 進行中** — ミニマル DOS ローダ
（日次の詳細経緯は [CHANGELOG.md](CHANGELOG.md) / 確立した知見は memory/MEMORY.md を参照。以下は到達点サマリ）
- ✓ **DOS ローダ確立**: INT 21h 多数 (file/mem/date/vector/IOCTL/find/exec/stdin) / INT 23h Ctrl-C ハンドラ発火 (既定=中断・IRET/far RET 復帰規律・PSP+0Eh 保存復元・ブラウザ Ctrl+C 透過) / AH=4Bh EXEC / TSR (AH=31h・INT 27h) / MCB チェーン / .bat インタプリタ (errorlevel 分岐・cd・set) / XMS Tier1 (既定 ON) / INT 33h マウスドライバ (MS/NEC 二流派・既定 MS・実測正典 tools/mousetest/) / VRAM 直書きは `memp_write8` 経由 / tty (PC-98 ANSI・INT 29h・INT DCh CL=10h・SGR・グラフ文字モード・カーソル座標ワーク 0x710/0x71C 同期)
- ✓ **テキストエディタ互換クラス**: VZ / JED / MUAP (INT DCh setkey) + ホスト IME 日本語入力 (FEP 非常駐・SJIS 注入)
- ✓ **HLE FEP (日本語入力の第 2 経路、2026-07-07〜08)**: 実 FEP と同じ「キー横取り→よみ/文節をゲスト画面へインライン描画 (VRAM 直書き・セル所有権検証復元)→確定 SJIS 注入」をホスト変換ループで再現。変換 = **hechima-wasm** (BSD-3・powered by Mozc、wasm 2.7MB + 辞書 19MB を FEP 初回 ON で遅延 fetch、専用 Worker + watchdog 自己回復)。複数文節 (←→移動・候補選択)・句読点即確定・トグル =「あ」ボタン/Ctrl+Space/Ctrl+J (ChromeOS は Ctrl+Space 不可)・設定 FEP Style (WX/ATOK)。**ビルドと正典は logical-layout-labo の `hechima-wasm/` へ移管 (2026-07-13)**、成果物は同リポの GitHub Release から pin して web/assets へ vendoring (JS グローバル `HechimaModule` / C シンボル `hechima_init`・`hechima_convert`・`hechima_resize` (v0.2.0〜) / **ラッパーは必ず -DNDEBUG** の罠あり)。回帰 = fep_test / fep_mozc_test / fep_resize_test。詳細 = [[project_hechima_stack]] / docs/hechima_handoff.md
- ✓ **音楽**: PMD `.M` (自前ビルド KAJA・常駐演奏) / FMP・FMDSP (ちびおと = 86+ADPCM) / MIDI (RS-MIDI・MPU-PC98 + TinySoundFont/SF2) / OPNA 内蔵リズム (クリーン代替 WAV) / BEEP ブースト
- ✓ **Mate-X PCM (CS4231) 検出対応 (2026-07-09)**: DOS/4GW 近代エンジン (Suika3 移植版等、FM を見ず SB16/Mate-X PCM だけ検出) の「No supported sound card found.」を根治。既定 `SOUND_SW` を段階選択化 (設定 Sound Board: 86 / 86+ADPCM / **86+ADPCM+Mate-X PCM=0x64 既定**)。0x64 は 0x14 の上位互換で FM/ADPCM 発音同一 (回帰なし実証)。**残=PCM 途切れはエンジン側の構造** (デコード律速ではない — 下記)。正典=[[reference_matex_pcm_wss]]
- ✓ **XMS の 15〜16MB ホール対応 (2026-07-09)**: DOS エクステンダ (DOS/4GW) の `Out of memory` を根治。PC-98 の物理 15〜16MB は RAM ではなく PEGC VRAM/未接続/先頭 1MB エイリアス (`CPU_EXTLIMIT16 = MIN(extsize+0x100000, 0xf00000)`)。`dos_xms.c` のプールが一枚板だったため、Lock して線形アドレスを直接触る DOS エクステンダに「1MB が RAM でない連続ブロック」を渡していた。ホールを使用中区間として除外 (`xms_occupied`)。EXTMEM=32MB で連続 EMB は 17.00MB = 実機 32MB 機と同値。診断 `qbDebug.extmem(MB)` (連続上限 = MB - 15)。ブラウザ実機で OOM 解消確認。正典=[[reference_pc98_15_16mb_hole]]
- ✓ **Suika3 の音の途切れ = デコードではなくメインループ周期 (2026-07-09、計測で確定)**: `98main.c` は毎フレーム全画面クリア+ソフト合成+GDC プレーン変換をし、バッファ補充 (`sound_poll`) はループ 1 周に 1 回だけ。実測 1 周 4.43 エミュ秒 (multiple=20) に対し音声 half は 1.024 秒 → 古い half が鳴り直される。**音源ボードを外しても 1 周は 4.43 秒のまま** (デコードは 8% 未満)。CPU プロファイルでも SoftFloat3 は 12.5% で FPU 説も棄却。→ ホスト Vorbis デコード内蔵は無意味。直すならエンジン側で `sound_poll()` を描画ループ中からも呼ぶ。我々側の本丸はエミュ本体の高速化 → 下へ
- ✓ **エミュ高速化 第 1+2 弾 (2026-07-10/11)**: patch 07 (統合)。第 1 弾 = メモリ/フェッチ fast path インライン化 (conventional + 拡張 2 窓。**DOS/4GW はコードを 16MB 以上に置く**のが肝) → Suika3 **1.39 倍** (11.2→8.1ms)。第 2 弾 = 16bit 実モード対応 (vmemory/load_segreg 逐語インライン + 16bit 直接ディスパッチ + USE_CPU_INLINEINST/EIPMASK) → Ray **1.43 倍** (14.0→9.8ms)。挙動不変・回帰全 PASS。ブラウザ実機 (ユーザー): 「Ray が一番体感できる。multiple 26 まで上げられる (前は 20 超で即ノイズ)」。**ベンチは `tools/bench_game.js` (32bit) + `tools/bench_ray.js` (16bit) の両方で** (bench_frame.js は BOUND 例外連発で longjmp を測ってしまう罠)。溢れ診断 = `np2kai_debug_memprobe(100+i/200+i)`。既定 multiple=20 据え置き (27 は Ray 級の実機確認後に判断)。正典=[[reference_cpu_mem_fastpath]]
- ✓ **画像・文書ビューア**: MAG・PI デコーダ / readme (NEC 罫線→Unicode・VZ %タグリンク) / 仮想 30 行 BIOS (`qbDebug.lines30`)
- ✓ **ホスト連携 QoL**: ゲームパッド / ファイル単体 Save・＋Add / 閲覧専用形式 (画像/音楽) の非破壊オープン / サブディレクトリ起動の CWD 代行
- 動作確認: さめがめ / ザルバール / Super Depth / Ray IV / うさちゃん列車 / 東方旧作 4 作 (TH02-05 体験版・ブラウザ実機確認) / bio100 純ゲーム 31 本 (ALIVE21・CRASH0・描画到達 25・動作確認 27) / MIMPI v3.8 (MIDI プレイヤー、I/F=MPU 演奏 + LIO ミキサー画面・ブラウザ実機確認 2026-07-03)
- ターゲット: フロッピーベース・2D・〜1998 年の同人/フリーソフト (期待カバー率 80〜90%)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [msonrm/quubee](https://github.com/msonrm/quubee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
