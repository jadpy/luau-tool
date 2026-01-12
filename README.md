# Luau Tool

Luau用のコード操作ツール。ASTベースのミニファイアー、フォーマッター、リンターを提供します。

## 機能

### 1. フォーマッター
コードを読みやすく整形します
- 適切なインデント
- 改行の配置
- スペース管理

```bash
luau-tool format code.luau
luau-tool format code.luau -o formatted.luau
```

### 2. ミニファイアー  
コードを圧縮します
- ローカル変数名の短縮化 (a, b, c... など)
- グローバル変数は保持
- コードサイズを削減

```bash
luau-tool minify code.luau
luau-tool minify code.luau -o minified.luau
```

### 3. リンター
エラーと問題を検出します
- 未定義の変数を検出
- 構文エラーを報告
- 潜在的な問題を指摘

```bash
luau-tool lint code.luau
```

### 4. コード分析
AST と詳細なエラー情報を表示

```bash
luau-tool analyze code.luau
```

## インストール

```bash
npm install
npm run build
```

## ビルド

```bash
npm run build
```

開発モード（ファイル変更を監視）:
```bash
npm run dev
```

## 使用方法

### CLI コマンド

```bash
# フォーマット
luau-tool format input.luau

# ミニファイ
luau-tool minify input.luau -o output.luau

# リント
luau-tool lint input.luau

# 分析
luau-tool analyze input.luau
```

### TypeScript APIとして使用

```typescript
import { LuauTool } from './src/index';

const tool = new LuauTool();

// コードをフォーマット
const result = tool.formatCode(code);
if (result.success) {
  console.log(result.result);
}

// コードをミニファイ
const result = tool.minifyCode(code);
if (result.success) {
  console.log(result.result);
}

// コードをリント
const result = tool.lintCode(code);
if (result.success) {
  result.errors?.forEach(error => {
    console.log(`${error.code}: ${error.message}`);
  });
}

// 詳細分析
const result = tool.analyzeCode(code);
console.log(result.ast);
console.log(result.errors);
```

## プロジェクト構造

```
src/
├── tokenizer.ts    # トークン定義
├── lexer.ts       # 字句解析（トークン生成）
├── ast.ts         # AST ノード定義
├── parser.ts      # 構文解析（AST 生成）
├── formatter.ts   # コードフォーマッター
├── minifier.ts    # コードミニファイアー
├── linter.ts      # エラー検出
├── index.ts       # メインツール（パブリック API）
└── cli.ts         # コマンドラインインターフェース
```

## 対応している Luau 構文

### ステートメント
- `local` 変数宣言
- `function` 関数定義
- `if/then/elseif/else/end` 条件分岐
- `while/do/end` ループ
- `for/do/end` ループ
- `return` リターン
- `break` ブレーク

### 式
- 二項演算子 `+`, `-`, `*`, `/`, `%`, `^`, `..`, `==`, `~=`, `<`, `<=`, `>`, `>=`, `and`, `or`
- 単項演算子 `not`, `-`
- 関数呼び出し
- テーブルアクセス `.`、`[]`
- テーブル構築 `{}`
- リテラル（数値、文字列、boolean、nil）

## サンプル

### フォーマット例

入力:
```lua
local function add(a,b)
return a+b
end
local result=add(2,3)
print(result)
```

出力:
```lua
local function add(a, b)
  return a + b
end
local result = add(2, 3)
print(result)
```

### ミニファイ例

入力:
```lua
local function greet(name)
  print("Hello, " .. name)
end
local message = greet("World")
```

出力:
```lua
function greet(a)
  print("Hello, " .. a)
end
local b = greet("World")
```

## TODO

- [ ] より詳細なエラー報告
- [ ] マルチバイト文字のサポート改善
- [ ] パフォーマンス最適化
- [ ] 包括的なテストスイート
- [ ] CLI の拡張機能
- [ ] ソースマップの生成
