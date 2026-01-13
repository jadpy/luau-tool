# Lua Tool - Luau Code Processor

Luauコード処理ツール。トークンベースの解析を用いて、圧縮・フォーマット・リント・自動修正などを行います。Roblox開発向けの基本操作をサポートします。

## 概要

本ツールは以下の7つの操作を提供します：

### 基本操作（3つ）

- **compress**（圧縮）: 行単位でコメントと空行を削除し、残ったステートメントをセミコロン (`;`) で結合して1行化します。テーブル内（`{}` の中）ではセミコロンを追加しません。例：`if true then; print(1); end`
  
- **format**（フォーマット）: セミコロン区切りやキーワード配置を解析して、改行とインデントを復元し、読みやすいコードに戻します。

- **lint**（リント）: 未終了ブロックなどの簡易構文チェックを実行し、詳細なエラー位置とメッセージを表示します。

### 修正・変換操作（4つ）

- **fixcode**（自動修正）: 
  - `if`、`elseif`、`function`、`while`、`for`、`repeat` などの未終了ブロックを自動検出
  - 不足している `end` / `until` を自動追加
  - 不足している `then`（`if` / `elseif` の後）を自動補完
  - 不足している `do`（`while` / `for` の後）を自動補完
  - **新機能**: 余分な `end` を自動削除（既に閉じられているコードをシミュレートして修正をリセット）
  - 修正内容を修正ログで表示

- **local**（ローカル変数分析）: ソース内のグローバルスコープにある `local` 宣言の数と変数名をリスト表示します。

- **dlocal**（ローカル削除）: 
  - グローバルスコープの `local` キーワードを削除してグローバル変数に変換
  - 値を代入している `local x = 1` のみ削除（→ `x = 1`）
  - 初期化なしの `local a` は保持
  - コメント内の `local` は削除しません

- **rename**（難読化）: 
  - 変数名をランダムな1文字に置き換え、コードを読みづらくします
  - Luaキーワード（`if`、`function` など）と50以上の組み込み関数は保持
  - 文字列・コメント内は変更されません

## 必要環境

- Lua 5.4（`lua54` コマンドが利用可能であること）

## インストール

```bash
git clone <repository>
cd lua-tool
```

## CLI 使用方法

### コマンド一覧

```bash
lua54 cli.lua compress file.lua           # 圧縮
lua54 cli.lua format file.lua             # フォーマット
lua54 cli.lua lint file.lua               # リント
lua54 cli.lua fixcode file.lua            # 自動修正
lua54 cli.lua local file.lua              # ローカル変数分析
lua54 cli.lua dlocal file.lua             # ローカル削除
lua54 cli.lua rename file.lua             # 難読化
lua54 cli.lua help                        # ヘルプ表示
lua54 cli.lua version                     # バージョン表示
```

### 出力オプション

#### デフォルト出力（`<basename>.out<ext>` ファイルに自動生成）

```bash
lua54 cli.lua compress code.lua
# → code.out.lua に出力
```

#### 明示的な出力ファイル指定

```bash
lua54 cli.lua compress code.lua -o custom_out.lua
```

#### 標準出力（stdout）に出力

```bash
lua54 cli.lua compress code.lua -o -
# または cat code.lua | lua54 cli.lua compress -o -
```

### オプション

- **`--output <file>`, `-o <file>`**: 出力先ファイル指定（デフォルト: `<input>.out<ext>`）
- **`--indent-size <n>`**: インデント幅（デフォルト: 2）
- **`--indent-char <tab|space>`**: インデント文字（デフォルト: タブ）
- **`--lang <code>`**: メッセージの言語（`en` または `ja`、デフォルト: `en`）
- **`--dry-run`**: 修正内容をプレビュー（ファイルには書き込まない）

### 日本語対応

圧縮・フォーマット・修正などのコマンドで日本語メッセージを表示：

```bash
lua54 cli.lua compress file.lua --lang ja
lua54 cli.lua format file.lua --lang ja
lua54 cli.lua fixcode file.lua --lang ja
lua54 cli.lua dlocal file.lua --lang ja
```

出力例（圧縮）:
```
圧縮して code.out.lua に書き込みました
```

### ドライランモード

修正・圧縮・フォーマット・難読化コマンドで実際にはファイルに書き込まず、何が行われるかをプレビュー：

```bash
lua54 cli.lua fixcode file.lua --dry-run
# [DRY RUN] Would write fixed code to file.out.lua

lua54 cli.lua rename file.lua --dry-run
# [DRY RUN] Would write obfuscated code to file.out.lua
```

### 統計情報

すべてのコマンド実行時に、以下の統計情報を表示します：

```
[STATS] Lines: 150, Tokens: 840, Bytes: 4256
```

- **Lines**: ソースコードの行数
- **Tokens**: トークン化された要素の個数（キーワード、識別子、文字列など）
- **Bytes**: ソースコードのバイト数

## 詳細な使用例

### 1. compress（圧縮）

入力ファイル `original.lua`:
```lua
if x > 0 then
    print("positive")
end
```

実行：
```bash
lua54 cli.lua compress original.lua
```

出力ファイル `original.out.lua`:
```lua
if x > 0 then;print("positive");end
```

### 2. fixcode（自動修正）

入力ファイル `broken.lua`:
```lua
function processData(data)
    if data == nil
        print("Error")
        return
    elseif #data > 0
        for i, v in ipairs(data) do
            if v > 0
                print(v)
```

実行：
```bash
lua54 cli.lua fixcode broken.lua
```

出力ファイル `broken.out.lua` と修正ログ:
```
[FIXES APPLIED]
[FIX] line 2: added 'then' after 'if'
[FIX] line 5: added 'then' after 'elseif'
[FIX] line 8: added 'then' after 'if'

[STATS] Lines: 10, Tokens: 56, Bytes: 180
```

#### fixcode の内部処理

fixcode は3つのパスで処理します：

**Pass 1: ブロック追跡**
- `if`, `elseif`, `function`, `while`, `for`, `repeat`, `do` の開始を検出
- 対応する `end` や `until` で閉鎖
- ネストされたブロックをスタック管理
- 処理後、未閉鎖ブロックのスタックを取得

**Pass 2: キーワード補完**
- `if` / `elseif` の後に `then` がなければ追加
- `while` / `for` の後に `do` がなければ追加
- 不足している `end` キーワードを追加

**Pass 3: 余分な `end` 削除**
- トークン再カウント：開き括弧（`if`, `while`, `for`, `function`, `repeat`, `do`）をカウント
- 閉じ括弧（`end`, `until`）をカウント
- **特別処理**: `for...do` または `while...do` の場合、`do` は開き括弧としてカウント**しない**（1つのブロックだから）
- 閉じ括弧が開き括弧より多い場合、末尾から余分な `end` を削除
- 削除されたキーワードを修正ログに記録

例：
```lua
-- Pass 3 で検出される例
function test()  -- function = 1 opener
    if x then  -- if = 2 openers, then = 補完
        for i=1,10 do  -- for = 3 openers, do は part of for なのでカウントしない
            print(i)
        end  -- 1 closer (1st for)
    end  -- 2nd closer (if)
end  -- 3rd closer (function)
end  -- 4th closer = **余分** → 削除

-- 結果: 3 openers vs 4 closers → 1つ削除
```

### 3. local（ローカル変数分析）

入力ファイル `vars.lua`:
```lua
local x = 1
local y = 2

function inner()
    local z = 3  -- グローバルスコープではない
end

local a = x + y
```

実行：
```bash
lua54 cli.lua local vars.lua
```

出力：
```
[STATS] Lines: 8, Tokens: 45, Bytes: 120

Total local declarations: 4
Variables:
  1. x
  2. y
  3. function
  4. a
```

### 4. dlocal（ローカル削除）

入力ファイル `global.lua`:
```lua
local x = 1
local y = 2
local myFunc = function() return x + y end
print(myFunc())
```

実行：
```bash
lua54 cli.lua dlocal global.lua
```

出力ファイル `global.out.lua`:
```lua
x = 1
y = 2
myFunc = function() return x + y end
print(myFunc())
```

### 5. rename（難読化）

入力ファイル `code.lua`:
```lua
local myVariable = 10
local anotherVar = 20
print(myVariable + anotherVar)
```

実行：
```bash
lua54 cli.lua rename code.lua
```

出力ファイル `code.out.lua` (例):
```lua
local p = 10
local Q = 20
print(p + Q)
```

注：`print` は組み込み関数として保持されます。

## 実装のポイント

### トークンベース解析
すべての処理は **トークン化された結果** を使用します。これにより：
- 文字列内の `if` や `end` を誤検出しない
- コメント内のキーワードを無視
- 完全に安全な変換が可能

### スコープ追跡
`local` / `dlocal` / `countLocals` では、深度ベースのスコープ追跡により：
- グローバルスコープのみを操作
- ネストされた関数内のローカル変数は無視

### 保護リスト（rename）
50以上のLua組み込み関数とキーワードを保護：
- キーワード: `if`, `then`, `else`, `function`, `return`, など
- 組み込み関数: `print`, `type`, `pairs`, `ipairs`, `tonumber`, `tostring`, `assert`, `error`, `pcall`, など
- モジュール: `os`, `io`, `math`, `string`, `table`, `debug`, `coroutine`
- メタテーブル: `setmetatable`, `getmetatable`, `rawget`, `rawset` など

### for...do の正確な処理
`for i=1,10 do` では `for` と `do` を1つのブロックと見なし、重複カウントを防止：
- Pass 1 で `for` の直後で `do` が続く場合、スタックに追加する際に `do` をスキップ
- Pass 3 でトークン再カウント時、`do` の前20トークン以内に終了なしの `for` / `while` があれば、`do` は開き括弧としてカウント**しない**

## プロジェクト構成

```
src/
├── main.lua          メインエントリーポイント（ファイル/文字列処理）
├── ast.lua           トークン化エンジン
├── compressor.lua    圧縮処理（行→セミコロン結合）
├── formatter.lua     フォーマット処理（復元）
├── linter.lua        リント/エラー検出
└── utils.lua         ユーティリティ関数

cli.lua              コマンドラインインターフェース
README.md            このファイル
```

## 次の改善候補

- **括弧や引数内のスペース規則の厳密化**: `print("a")` のようなフォーマッタで関数呼び出し前のスペース除去、括弧内のスペース管理
- **複数行/ブロックコメント対応**: `--[[...]]` 構文に対応した完全ASTパーサ、複雑な式の正確な解析
- **Roblox API 型注釈**: Roblox API定義の型注釈や補完データの拡充、開発時の補助情報提供
- **AST駆動型の rename**: スコープ別に変数名を管理（グローバル変数と関数引数を区別）
- **reach-ability 解析**: 到達不可能コード（`return` 後の処理）を検出
- **変数シャドウ警告**: ネストされたスコープで同じ名前のローカル変数を警告
- **設定ファイル対応**: `luatool.conf` で操作のデフォルトオプション指定
- **複数ファイル処理**: ディレクトリ単位での一括処理
- **マージ機能**: 複数ファイルを結合してローカルスコープを保持

## ライセンス

MITライセンス（詳細は LICENSE ファイルを参照）
