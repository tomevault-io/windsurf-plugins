---
trigger: always_on
description: このディレクトリは伺かのSHIORI "YAYA" のコードです。
---

このディレクトリは伺かのSHIORI "YAYA" のコードです。
C++98/旧いVC++でコンパイルできるように製作されており、文字コードShift JISなので、Sjis_ではじまる編集・探索ツール群を使う必要があります。
コード管理ツールはgitです。
仕様は C:\D_DRIVE\MyDocuments\GitHub\yaya-docs にあります。

## ビルド

### VC++6

VC++6 (`msdev.exe`) が PATH に通っているため、`aya5.dsw` をコマンドラインからビルドできる。

```powershell
msdev aya5.dsw /MAKE "aya5 - Win32 Debug" /OUT "$env:TEMP\claude\yaya_build.log"
Get-Content "$env:TEMP\claude\yaya_build.log" -Encoding oem
```

- 構成は `aya5 - Win32 Release` / `Debug` / `Release EXE` / `Debug EXE` / `ReleaseLangSep`（全部は `aya5 - ALL`）。`/MAKE` は差分ビルドで、全再ビルドは `/REBUILD`
- `msdev.exe` はGUIアプリのためコンソールに何も出ない。結果は `/OUT` のログで見る（Shift JIS なので `-Encoding oem`）。成功時の末尾は `yaya.dll - ｴﾗｰ 0、警告 N`。VC6 の STL 由来の警告 C4786 は無視してよい
- Git Bash から実行すると `/MAKE` がパス変換されるので `MSYS_NO_PATHCONV=1` を前置する
- `messagetxt/*.txt` は `aya5.rc` からリソースとして埋め込まれるが、`/MAKE` は変更を検知しない。変えたら `/REBUILD` するか `Release/aya5.res` などを消す

### サブモジュール

- parson / tinyxml2 / deelx / sqlite / gumbo は git サブモジュール。クローン直後は `git submodule update --init` が必要
- 本体側では編集しない。tinyxml2 / deelx / SQLite の修正は `../tinyxml2` / `../deelx` / `../sqlite-amalgamation-new` のフォーク側で行ってから参照を更新する。いずれも UTF-8 なので Sjis_ 系ツールで触らない
- サブモジュールを使う処理（JSON/XML、SQL* 関数、HTML）に手を入れる前に `git submodule update --init --remote parson tinyxml2 deelx sqlite gumbo` で参照を最新にする
- SQLite のフォークは upstream の amalgamation に VC6 / 古い SDK 向けの修正を1コミット載せたもの。64bit のリテラルは `INT64_C()` / `UINT64_C()`（`LL` は VC6 が、`i64` は gcc が読めない）。修正したら VC6 と gcc（Strawberry Perl 同梱）の両方で `sqlite3.c` 単体がコンパイルできることを確かめる

### VC6 以外（makefile / GitHub Actions）

- `makefile.linux` が基準で、`freebsd` / `posix`（macOS、`.bundle`）/ `emscripten` はオブジェクト一覧と規則を linux と同じにしてある。ソースを増やしたら全部の一覧に足す（`makefile.mingw32` は `posix_utils.o` に加えて `aya5_res.o` も持つ）
- ソースは Shift JIS（CP932）で統一する。POSIX 系の makefile は `.cpp` を iconv で UTF-8 にしてからコンパイルし（変換できない文字があれば止まる。`#line` で元のファイルを指す）、MinGW は `-finput-charset=CP932` なので、UTF-8 のファイルが混ざると通らない。POSIX ではヘッダは変換しないので、ヘッダの非 ASCII はコメントだけにする
- `makefile.mingw32` は Windows 用の yaya.dll を MinGW-w64 で作る（文字列を VC++ と同じく CP932 のまま扱う）。Strawberry Perl 同梱の gcc（x86_64）で確かめられる。リポジトリを汚さないよう `git ls-files --recurse-submodules` のファイルを作業用ディレクトリに写してから `mingw32-make -f makefile.mingw32`（動作確認用の EXE は `exe` ターゲット）
  - MinGW は `_WINDOWS` だが `_MSC_VER` ではない。SEH（`__try`）など MSVC 専用の書き方は `_MSC_VER` で分け、64bit の整数リテラルは `LL_DEF()` / `ULL_DEF()` で書く
- Linux / macOS は手動実行の `.github/workflows/posix-build.yml`（`gh workflow run posix-build.yml --ref 600`）でビルドし、`tests/posix_smoke.c` で dlopen → load → request → unload → dlclose まで通す。スモークテストは空ディレクトリを load してシェルモードにし、式を EVAL した結果を比べる（期待値を足すときは `yaya.exe` で先に確かめる）
  - gcc で通って clang（macOS）で落ちる典型: `yaya::int_t`（`std::int64_t`）は Linux が `long`、macOS が `long long` なので、`ptrdiff_t` / `size_t` から `CValue(...)` を作ると曖昧になる（`static_cast<yaya::int_t>` を付ける）。他の翻訳単位から呼ぶ関数は1ファイルだけで `inline` 定義しない（gcc はたまたまリンクできるが clang は実体を出さない）。macOS の `<fcntl.h>` は `FREAD` / `FWRITE` をマクロにする（`sysfunc.h` で `#undef`）。`basename` には `<libgen.h>` が要る
  - POSIX 版 `CBasis::ExecuteRequest` は成功時に入力バッファを `free` しない（エラー時は `free` する）。テストも成功時は解放しない
  - POSIX のパスは UTF-8。内部の文字列からは `posix_path()`（UTF-8 化と `fix_filepath`）で作り、OS から受け取った名前は `Ccct::MbcsToUcs2Buf(..., CHARSET_UTF8)` で戻す（旧 `narrow` / `widen` は廃止）。文字コード変換は iconv（macOS は `-liconv`）で、OS デフォルトは UTF-8 扱い
  - 公開するのは `DLLEXPORT` の関数だけ（`-fvisibility=hidden`）。スモークテストの共通ケースは Windows の `yaya.exe` で期待値を確かめてから足す。非 ASCII は UTF-8 の `\x` エスケープで書き、`\x` の直後に16進の文字が続くときは文字列を区切る
  - Emscripten は CI の対象外（最新の emsdk では `-shared` が `-fPIC` 必須の SIDE_MODULE になりリンクが通らない）。MinGW / VC6 も CI には載せていない

## リリース手順

1. バージョンを上げる（例: `Tc600-2`）。どちらも Shift JIS かつ CRLF なので Sjis_ 系ツールで編集する（Git Bash の `sed -i` は CRLF を LF に変えてしまう）
   - `manifest.cpp` の `aya_version`
   - `aya5.rc` の `FILEVERSION 6,00,2,0` と `VALUE "FileVersion", "6, 00, 2, 0\0"`（`TcXYY-N` → `X,YY,N,0`）
2. サブモジュール（`parson` / `tinyxml2` / `deelx` / `sqlite` / `gumbo`）の参照を最新に更新する
   ```powershell
   git submodule update --init --remote parson tinyxml2 deelx sqlite gumbo
   git submodule status
   ```
   - 参照が変わったら `git diff --submodule` で取り込まれるコミットを確認し、リリースのコミットに含める
   - ライセンス文の年や著作権者が変わっていたら `sysfunc.cpp` の `LICENSE` 関数の該当部分も合わせる（SQLite はパブリックドメイン）
3. Release と ReleaseLangSep をリビルドし、ログ末尾が `ｴﾗｰ 0` であることを確認する
   ```powershell
   msdev aya5.dsw /MAKE "aya5 - Win32 Release" /REBUILD /OUT "$env:TEMP\claude\yaya_build_Release.log"
   msdev aya5.dsw /MAKE "aya5 - Win32 ReleaseLangSep" /REBUILD /OUT "$env:TEMP\claude\yaya_build_ReleaseLangSep.log"
   ```
4. `make_aya.bat` を実行して `tmp/yaya.zip` と `tmp/yaya_lang_sep.zip` を作る
   - PowerShell から `cmd /c .\make_aya.bat` で実行する（Git Bash の `cmd //c make_aya.bat` はバッチが見つからず失敗する）
   - zip の更新日時が新しくなったこと、中の `yaya.dll` が `Release/` `ReleaseLangSep/` のものと一致することを確認する
5. コミットして push する。コミットメッセージの1行目は `変更内容 / TcXYY-N`、本文に変更点の箇条書き
6. GitHub のリリースを作る。タグ名とタイトルはバージョン、対象は現在のブランチ、2つの zip を添付、本文は変更点の日本語の箇条書き
   ```powershell
   gh release create Tc600-2 tmp/yaya.zip tmp/yaya_lang_sep.zip --target 600 --title Tc600-2 --notes-file <本文ファイル>
   ```

## 動作確認とシェルモード

- 通常モードで読む辞書が1つも無い（設定ファイルが無い、dic/dicif/dicdir の結果が0個）と、`CParser0::ParseShellDictionary` が `load{}unload{}request{EVAL(_argv[0])}` だけの組み込み辞書（`_SHELL_DIC_`）を作り、request の入力を EVAL した結果を返す。ログには注記 N0002 が出る
- 設定ファイルが無いときは変数の自動保存・復元をしない。緊急モード（`yaya_emerg.txt`）はシェルモードにならず、従来どおり設定ファイルが無ければ抑止する

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YAYA-shiori/yaya-shiori](https://github.com/YAYA-shiori/yaya-shiori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
