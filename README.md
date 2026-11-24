# Edax

Edax is a very strong othello program. Its main features are:
- fast bitboard based & multithreaded engine.
- accurate midgame-evaluation function.
- opening book learning capability.
- text based rich interface.
- multi-protocol support to connect to graphical interfaces or play on Internet (GGS).
- multi-OS support to run under MS-Windows, Linux and Mac OS X.

## Installation
From [the release section of github](https://github.com/abulmo/edax-reversi/releases), you must 7unzip both an executable of your favorite OS, and the evaluation weights (data/eval.dat) in the same directory.
Only 64 bit executable with popcount support are provided.

## ビルドと実行 (macOS / Linux)

### ローカルビルド・実行（Apple Silicon macOS 例）

`src/makefile` に従い、コンパイラは `CC=clang`、アーキテクチャは `ARCH=armv8.5-a`（または `ARCH=native`）を指定してビルドします。生成されたバイナリは `bin/mEdax-armv8.5-a` に出力されます。

```sh
mkdir -p bin
cd src
make build ARCH=armv8.5-a CC=clang OS=osx
cd ..
```

> `pgo-build` を行う場合は、先に `bin/data/eval.dat` および `bin/problem/` を配置したうえで `make pgo-build ARCH=armv8.5-a CC=clang OS=osx` を実行してください。

評価関数データ `eval.dat`（[GitHub Releases](https://github.com/abulmo/edax-reversi/releases) の `eval.7z` を解凍）を `bin/data/eval.dat` に配置します。  
Edax はデフォルトでカレントディレクトリ基準の `data/eval.dat` と `data/book.dat` を読み込むため、`bin/` ディレクトリに移動して起動します。

```sh
cd bin
./mEdax-armv8.5-a
```

### Docker

```sh
docker build . -t edax
docker run --name "edax" -v "$(pwd)/:/home/edax/" -it edax

cd /home/edax/
mkdir -p bin
cd src
make build ARCH=x86-64-v3 COMP=clang OS=linux

cd ..
curl -OL https://github.com/abulmo/edax-reversi/releases/download/v4.4/eval.7z # e.g. use v4.4 eval.dat
7z x eval.7z

./bin/lEdax-x64
```

## 定跡ブックの設定 (bin/data/book.dat)

`data/eval.dat` は評価関数の重みデータであり、定跡（オープニングブック）は別ファイルの `data/book.dat` に格納されます。別パスのブックを使う場合は起動時または対話コマンドで `-book-file <path>`（または `book-file <path>`）を指定できます。

強力な公開 Edax 形式ブックの導入例として、Kyrin 氏公開の `othello.drawline.dat`（Edax level 30 / depth 30、約 196 MB）が利用できます。

- **ダウンロード**: https://www.axfc.net/u/4072793 （解説: https://kyrinothello.livedoor.blog/archives/3011430.html）
- **SHA256**: `957f4de3f0a15b7bb332d143ba79c084a2d50bf18d75a70a210564b21e757364`

配置・検証コマンド例：

```sh
mkdir -p bin/data
cp ~/Downloads/othello.drawline.dat bin/data/book.dat
shasum -a 256 bin/data/book.dat
```

起動後の確認例（`bin/` から実行）：

```sh
cd bin
./mEdax-armv8.5-a
# プロンプト (>) で以下を実行して読み込みを確認
# book info
# book show
```

> ※ `edax+tlb.zip` 同梱の Top Level Book（`srbook.bin`）は Windows GUI（`sreversi.exe`）専用の独自形式のため、CLI 版 Edax では直接読み込めません。

## 基本的な使い方・よく使うコマンド

### 主な起動オプション

| オプション | 説明 |
| :--- | :--- |
| `-l <n>` | 探索深さの上限を指定（デフォルト: `21`） |
| `-n <n>` | 並列探索スレッド数を指定（デフォルト: `1`） |
| `-h <n>` | ハッシュテーブルサイズをビット数で指定（デフォルト: `21`） |
| `-eval-file <path>` | 評価関数ファイルのパスを指定（デフォルト: `data/eval.dat`） |
| `-book-file <path>` | 定跡ブックのパスを指定（デフォルト: `data/book.dat`） |
| `-book-usage <on\|off>` | 定跡ブックを使用するか指定 |

起動例（探索深さ 24・4 スレッドで実行）：

```sh
cd bin
./mEdax-armv8.5-a -l 24 -n 4
```

### 対話プロンプト (`>`) の主なコマンド

#### 対局・盤面操作

| コマンド | 説明 |
| :--- | :--- |
| `init` (`i`) | 標準初期配置から新規対局を開始 |
| `<座標>` (例: `F5`) | 指定の座標に着手 |
| `go` | Edax に次の手を探索・着手させる |
| `mode <0\|1\|2\|3>` (`m`) | 対局モード切替（`0`: Edax 黒 / `1`: Edax 白 / `2`: Edax 同士 / `3`: 手動） |
| `undo` (`u`) / `redo` (`r`) | 1 手戻す / やり直す |
| `play <moves>` | 手順を連続入力（例: `play F5D6C3`） |
| `save <file>` (`s`) / `open <file>` (`o`) | 棋譜の保存 / 読み込み |
| `quit` (`q`) | Edax を終了 |

#### 解析・設定

| コマンド | 説明 |
| :--- | :--- |
| `hint [n]` | 上位 `n` 手の候補手と評価値を表示 |
| `analyze [n]` (`a`) | 対局の振り返り解析を実行 |
| `level <n>` (`l`) | 探索深さ（レベル）を変更 |
| `n-tasks <n>` (`n`) | 並列探索スレッド数を変更 |
| `help` (`?`) | ヘルプを表示 |

#### Book コマンド（`b` または `book`）

| コマンド | 説明 |
| :--- | :--- |
| `book info` | 登録局面数・レベル・深さなどブックの情報を表示 |
| `book show` | 現在局面のブック候補手・評価値を表示 |
| `book on` / `book off` | ブックの使用を有効化 / 無効化 |
| `book analyze [n]` | ブックを用いた棋譜の振り返り解析 |
| `book load <file>` / `book save <file>` | バイナリブックの読み込み / 保存 |

## Document

```sh
cd src
doxygen
open ../doc/html/index.html
```
## version 4.6
version 4.6 is an evolution of version 4.4 that tried to incorporate changes made by Toshihiko Okuhara in version 4.5.3 and :
 - keep the code encapsulated: I revert many pieces of code from version 4.5.3 with manually inlined code.
 - remove assembly code (intrinsics are good enough)
 - make some changes easily reversible with a macro switch (USE_SIMD, USE_SOLID, etc.)
 - remove buggy code and/or buggy file path.
 - disable code (#if 0) that I found too slow on my cpu.
 - make soft CRC32c behave the same as the hardware CRC32c (version 4.5.3 is buggy here).
 - the code switch from c99 to c17 and use stdatomic.h threads.h (if available) stdalign.h
 - remove bench.c: most of the functions get optimized out and could not be measured.
 - support only 64 bit OSes. 

## makefile
the major change is that the ARCH options are no longer the same, as they are too many possible options to enable avx2, avx512, CRC32c, etc.
Use make -help for a list of options. 


