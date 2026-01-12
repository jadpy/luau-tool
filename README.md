# Lua Tool - Luau Code Processor

Luauコード処理ツール。ASTベースの解析を用いて、圧縮・フォーマット・リントを行います。Roblox開発向けの簡易サポートも含みます。

## 概要

- 圧縮 (compress): 行単位でコメントと空行を削除し、残ったステートメントをセミコロン (`;`) で結合して1行化します（例: `if true then; print(1); end`）。
- フォーマット (format): セミコロン区切りやキーワードを解析して改行とインデントを復元し、読みやすいコードに戻します。
- リント (lint): 未終了ブロックなどの簡易構文チェックを実行します。

## 必要環境

- Lua 5.4 (`lua54` コマンドが利用可能であること)

## インストール

```bash
git clone <repository>
cd lua-tool
```

## CLI 使用方法

- 圧縮（デフォルト出力：`<basename>.out.lua`）:

```bash
lua54 cli.lua compress file.lua
```

- 圧縮（出力ファイル指定）:

```bash
lua54 cli.lua compress file.lua -o out.lua
```

- フォーマット（デフォルト出力：`<basename>.out.lua`）:

```bash
lua54 cli.lua format file.lua
```

- フォーマット（出力ファイル指定）:

```bash
lua54 cli.lua format file.lua -o formatted.lua
```

- リント:

```bash
lua54 cli.lua lint file.lua
```

オプション:

- `--indent-size <n>`: インデント幅（デフォルト2）
- `--indent-char <tab|space>`: インデント文字（デフォルトはタブ）
- `--lang <code>`: メッセージの言語（`en` または `ja`、デフォルト `en`）

```bash
lua54 cli.lua compress file.lua --lang ja
lua54 cli.lua format file.lua --lang ja
```

## 実装のポイント

- `compress` はソースの各行をトリムしてコメントを削除し、空でない行を `; ` で結合します。これによりステートメントが1行にまとまり、配布・最小化などに使えます。
- `format` はトークン化した結果を使い、キーワード（`if`, `then`, `else`, `end`, `for` 等）とセミコロンを基に改行・インデントを復元します。
- `linter` は未閉ブロック（`if` に対応する `end` がない等）を検出します。

## プロジェクト構成

```
src/
├── main.lua          メインエントリーポイント（ファイル/文字列処理）
├── ast.lua           トークン化と簡易ASTのエントリ
├── formatter.lua     フォーマッター（圧縮された入力から復元可能）
├── compressor.lua    圧縮（行→セミコロン結合）
├── linter.lua        簡易リント/エラー検出
├── roblox.lua        Roblox API定義（補完/参照用の簡易リスト）
└── utils.lua         ユーティリティ関数

tests/
└── test.lua          テストスイート（基本機能の検証）

examples/
├── compress.lua      圧縮の例
├── format.lua        フォーマットの例
└── lint.lua          リントの例
```

## 使い方の例

- 圧縮してからフォーマットで復元するワークフロー:

```bash
lua54 cli.lua compress script.lua        # script.lua を圧縮して script.out.lua に出力
lua54 cli.lua format script.out.lua -o script.lua    # 圧縮済みを整形して script.lua に復元
```

## 次の改善候補

- 括弧や引数内のスペース規則の厳密化（例: `print("a")` のような自動修正）
- 複数行/ブロックコメントや複雑な式の扱いを強化する完全ASTパーサ
- Roblox API の型注釈や補完データを拡充

---
更新: CLIでの上書き動作と、圧縮がセミコロン結合である点を反映しました。
更新: 出力ファイル名を `<basename>.out<ext>` に変更、`--lang ja` で日本語メッセージに対応しました。
