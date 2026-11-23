# EmuZ-80K Multi-block Load Patch (.MZT text / .MZM)

EmuZ-80K（武田敏也氏の Common Source Code Project）に、マルチブロックの
テープデータを読み込めるようにするパッチです。FORMや、シャープの
システムプログラム（アセンブラ SP-2101／テキストエディタ SP-2201）の
ように、本体をブロック単位で読み込むプログラムが対象です。
通常は `.MZT` ファイルをそのまま読み込めます。拡張子 `.MZM` は、
自動判別が効かない場合のための予備手段です。

## 内容

- **属性による自動判別（2026年移植版で追加）が主な仕組みです**：
  `.MZT` ファイルでも、テープヘッダ先頭の属性（ファイルモード）が
  `0x04` のファイルは、マルチブロックとして読み込みます。
  属性 `0x04` は、MZ-80K のテキストソースファイルが持つ値です。
  FORM のソースも、システムプログラムのテキストエディタ SP-2201 が
  作るアセンブラソースも、この属性を持ちます（両者は同じ
  テキストファイル形式を共有しています）。
  これにより、これらのテキストは拡張子を `.MZM` に変えなくても、
  `.MZT` のまま読み込めます
  - 対象は MZ-80K 系（MZ-80K / MZ-80A / MZ-1200）のビルドのみです。
    他の機種では属性 `0x04` が別の種類のファイル
    （BASIC のシーケンシャルデータ）を意味し、ブロック構造も異なるため、
    そちらの従来動作は変えていません
  - 属性 `0x01` のファイル（機械語プログラム、およびメモリイメージ形式で
    記録される GAME-MZ のソースファイル）は、従来どおり単一ブロックで
    読み込みます
- **予備手段としての拡張子 `.MZM`**：
  属性が `0x04` でないファイルなど、自動判別が効かない場合のために、
  拡張子でマルチブロック読み込みを指定する `.MZM` も用意しています。
  MZT形式ファイル（128バイトヘッダ＋本体）を読み、本体を256バイトずつの
  テープブロックに分割して（各ブロックにリーダーを付けて）出力する
  `load_mzm_image()` で処理します
  - `.MZM` は単一オブジェクト専用です（先頭ヘッダ1つだけを処理し、
    連結された2つ目以降は読みません。2012年版の仕様どおり）

## 沿革

- **2012-04-14** — オリジナル版。madogiwaha氏と[@Koji_Morishita](https://x.com/Koji_Morishita)による改造。
  対象は Common Source Code Project 2012-03-20版。
  当時の説明は `original/README_2012.txt`、差分そのものは
  `patches/original_2012_mzm_patch.diff` を参照してください。
- **2026年移植** — 2012年版の変更を Common Source Code Project
  2026-01-01版へ移植。変更は3ファイルのみです
  （`src/vm/datarec.cpp`、`src/vm/datarec.h`、`src/win32/winmain.cpp`）。
  あわせて上記の属性による自動判別を追加しました。

## 2026-01-01版への適用方法

1. 本家から 2026-01-01版のソースを取得します
   - Common Source Code Project: https://takeda-toshiya.my.coocan.jp/common/
   - 旧版アーカイブ: https://takeda-toshiya.my.coocan.jp/common/history/2026-01-01.7z
2. ソースを展開したフォルダ（`src` フォルダがある場所）で、
   `patches/emuz80k_mzm_patch_for_2026-01-01.patch` を適用します
   （例: `git apply` または `patch -p1`）
3. Visual Studio で mz80k プロジェクトをビルドします

## ビルド済みバイナリ

ビルド済みの実行ファイルを、このリポジトリの Releases から
ダウンロードできます（ファイル名 `emuz80k_mzm_binary_2026-01-01.zip`）。

- 中身は `mz80k_mb.exe` です。**ファイル名を `mz80k.exe` に変えて**
  お使いください（本家の実行ファイルと区別できるよう、配布時は
  別の名前を付けています）
- Common Source Code Project 2026-01-01版に本パッチを適用し、
  Visual Studio 2008（vc++2008 プロジェクト、Release Win32）で
  ビルドしたものです
- `mz80k_mb.exe` の SHA-256:
  `f42b8cbb9bb94ab9e02c43761c1cb54c3e35c887cb238b7084aa2ddbbc88e2e1`
- このバイナリに対応するソースコードは、本家の 2026-01-01版ソースに
  本リポジトリのパッチを適用したものです。バイナリのZIPにも
  パッチと GPL の本文を同梱しています

## 状態（2026-11-23時点）

- 移植パッチは、まっさらな2026-01-01版へ再適用して成果物が
  バイト単位で一致することまで確認済みです
- Windows でのビルドを確認済みです
  （Visual Studio 2008、vc++2008 プロジェクト、Release Win32）
- 実ロードも確認済みです。FORM のソースファイル（GENOCIDE）を
  `.MZM` として、および拡張子を `.MZT` に変えたもの（属性 `0x04` のまま）
  の両方で読み込み、FORM のコンパイルまで成功しました
- システムプログラムでも通しで実証済みです。システムプログラム
  マニュアルの例題アセンブラソース（APPROACHING SQUARE）を
  SP-2201 で読み書きしたファイル（属性 `0x04`）を読み込み、
  SP-2101 でアセンブル、SP-2301 でリンク・セーブし、モニタから
  実行できることまで確認しました（2026-11-23）。
  属性による自動判別は FORM 専用ではなく、MZ-80K の
  テキストファイル全般に効きます
- 2012年版から意図的に変えた点が1つあります。ヘッダと本体の間に、
  2026年版本家の `load_mzt_image()` と同じ1秒の無音を入れました。
  実ロードで問題が出る場合は、この1行を外すと2012年版のタイミングに戻ります

## ライセンス

本パッチは、対象である Common Source Code Project のライセンス
（GNU General Public License Version 2 or Later）に従います。
GPL 2.0の本文を `LICENSE-GPL-2.0.txt` に同梱しています。

---

## English summary

Patch for EmuZ-80K (part of Toshiya Takeda's Common Source Code Project)
that adds support for multi-block tape images, needed by programs such as
FORM and Sharp's SYSTEM PROGRAM tools (the SP-2101 assembler and the
SP-2201 text editor) that load their text data block by block.

The main mechanism, added in the 2026 port, is auto-detection by file
attribute: a `.MZT` file whose tape-header attribute is `0x04` — the
attribute of MZ-80K text source files, shared by FORM sources and by
assembler sources written with the SP-2201 text editor — is played back
split into 256-byte tape blocks, so text files load as they are, with
no renaming. This auto-detection applies to the MZ-80K family builds
only (MZ-80K / MZ-80A / MZ-1200); attribute `0x01` files (machine
code, and GAME-MZ sources, which are memory images) are loaded as a
single block as before. The `.MZM` extension remains available as a
fallback for files the auto-detection does not cover: renaming a file
to `.MZM` forces multi-block playback regardless of its attribute.

Originally written in April 2012 by madogiwaha and [@Koji_Morishita](https://x.com/Koji_Morishita) for the
2012-03-20 source release, and ported in 2026 to the 2026-01-01 release
(3 files changed). The Windows build (Visual Studio 2008) and real load
tests have been verified: a FORM source file (GENOCIDE) loads and
compiles both as `.MZM` and as `.MZT` with attribute `0x04`, and an
assembler source file written by SP-2201 (attribute `0x04`) has been
verified end to end — loaded, assembled with SP-2101, linked with
SP-2301 and executed successfully (2026-11-23).
A prebuilt binary is available from the Releases page
(`emuz80k_mzm_binary_2026-01-01.zip`); rename `mz80k_mb.exe` to
`mz80k.exe` to use it. Licensed under GPL-2.0-or-later, following the
upstream project.
