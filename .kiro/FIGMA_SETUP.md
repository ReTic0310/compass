# Figma MCP Setup Guide

Figma MCPをKiro SDD v2に統合するためのセットアップガイド

---

## Prerequisites

1. **Figma アカウント**
   - Figmaアカウント（無料または有料）
   - デザインファイルへのアクセス権限

2. **Figma Access Token**
   - Personal Access Tokenの生成が必要
   - URL: https://www.figma.com/developers/api#access-tokens

3. **Cursor with MCP Support**
   - Cursor IDE (MCP対応版)
   - Figma MCP Server のインストール

---

## Step-by-Step Setup

### Step 1: Figma Access Token の取得

1. Figmaにログイン
2. Settings → Account → Personal Access Tokens
3. "Generate new token" をクリック
4. Token名を入力（例: "Kiro SDD Development"）
5. Scopeを選択:
   - ✅ File content (read)
   - ✅ File variables (read)
   - ✅ Comments (read) - オプション
6. "Generate token" をクリック
7. トークンをコピー（⚠️ 一度しか表示されません）

### Step 2: 環境変数の設定

#### .env ファイルの作成

プロジェクトルートに `.env` ファイルを作成:

```bash
# .env
FIGMA_ACCESS_TOKEN=your_figma_personal_access_token_here
```

⚠️ **重要**: `.env` ファイルは `.gitignore` に追加されていることを確認してください。

#### .gitignore の確認

```.gitignore
# Figma credentials
.env
.figma-config.json

# Figma snapshots (optional - commit if you want version control)
.kiro/figma-snapshots/
```

### Step 3: Figma設定ファイルの作成

テンプレートをコピー:

```bash
cp .figma-config.template.json .figma-config.json
```

`.figma-config.json` を編集:

```json
{
  "figma_access_token": "${FIGMA_ACCESS_TOKEN}",
  "figma_file_key": "ABC123DEF456",
  "figma_file_url": "https://www.figma.com/file/ABC123DEF456/Navigation-App",
  "target_platforms": {
    "flutter": {
      "enabled": true,
      "output_path": "flutter_app/lib/design_system/",
      "asset_path": "flutter_app/assets/"
    },
    "swiftui": {
      "enabled": true,
      "output_path": "watch_app/WatchApp/DesignSystem/",
      "asset_path": "watch_app/WatchApp/Assets.xcassets/"
    }
  },
  "export_options": {
    "icons": {
      "format": "svg",
      "scales": [1]
    },
    "images": {
      "format": "png",
      "scales": [1, 2, 3]
    }
  },
  "sync_options": {
    "auto_update_specs": false,
    "create_snapshots": true,
    "snapshot_interval_hours": 24
  }
}
```

**Figma File Key の取得方法**:
- Figma URLから抽出: `https://www.figma.com/file/ABC123DEF456/Project-Name`
- `ABC123DEF456` がFile Key

### Step 4: Figma MCP Server のインストール

#### Option A: Cursor内蔵のMCP（推奨）

最新のCursorを使用している場合、Figma MCPは自動的に利用可能です。

#### Option B: 手動インストール

```bash
# npmでインストール
npm install -g @anthropic-ai/mcp-figma

# または
npx @anthropic-ai/mcp-figma
```

### Step 5: 接続テスト

```bash
# Figma接続テスト
/kiro/figma review [your-figma-file-url]
```

成功すると、Figmaファイルの概要が表示されます。

---

## Figma File の準備

### 推奨ファイル構造

Figmaファイルを以下のように整理すると、Kiro SDDとの統合がスムーズです:

```
Figma File: [Your Project Name]
├── 📄 Cover
│   └── プロジェクト概要、リンク集
│
├── 🎨 Design System
│   ├── Colors (Variables として定義)
│   ├── Typography (Text Styles として定義)
│   ├── Spacing (Variables として定義)
│   ├── Icons (Components)
│   └── Components (再利用可能コンポーネント)
│
├── 📱 Flutter Screens
│   ├── Home Screen
│   ├── Navigation Screen
│   ├── Settings Screen
│   └── ...
│
├── ⌚ watchOS Screens
│   ├── Watch Home
│   ├── Compact Navigation
│   ├── Complications
│   └── ...
│
└── 🔗 Prototypes
    ├── Flutter Flow
    └── watchOS Flow
```

### Design System の設定

#### 1. Figma Variables（推奨）

色、スペーシング、その他の値は **Figma Variables** として定義:

**Colors**:
- `Primary/Blue` = #007AFF
- `Secondary/LightBlue` = #5AC8FA
- `Success/Green` = #34C759
- ...

**Spacing**:
- `Spacing/XXS` = 2
- `Spacing/XS` = 4
- `Spacing/S` = 8
- ...

#### 2. Text Styles

テキストスタイルを定義:
- `Heading/H1`: SF Pro Display, 34pt, Bold
- `Heading/H2`: SF Pro Display, 28pt, Bold
- `Body/Regular`: SF Pro Text, 17pt, Regular
- ...

#### 3. Components

再利用可能なコンポーネントを作成:
- `Button/Primary`
- `Button/Secondary`
- `Input/TextField`
- `Card/Elevated`
- ...

**命名規則**:
- カテゴリ/バリエーション形式
- Flutter/SwiftUIコード名と一致させる

### Prototypes の作成

インタラクションをプロトタイプで定義:
1. Flowsを作成（Flutter Flow, watchOS Flow）
2. 画面遷移をリンク
3. アニメーションを設定（duration, easing）
4. ジェスチャーを定義（tap, swipe, drag）

---

## 使い方

### 基本ワークフロー

#### 1. デザインレビュー

```bash
/kiro/figma review https://www.figma.com/file/ABC123/Navigation-App
```

Figmaファイル全体を解析し、`.kiro/figma-review.md` を生成。

#### 2. 機能初期化（Figma参照付き）

```bash
/kiro/init feature "Navigation screen with real-time route display"
```

#### 3. Figmaから要件生成

```bash
/kiro/spec navigation-display requirements --from-figma NavigationScreen
```

Figmaのデザインとプロトタイプから要件を自動抽出。

#### 4. Figma参照で設計生成

```bash
/kiro/spec navigation-display design --with-figma
```

Figmaコンポーネント、デザイントークン、アニメーションを設計に反映。

#### 5. アセットエクスポート

```bash
/kiro/figma export NavigationScreen
```

アイコン、画像をプラットフォーム別にエクスポート。

#### 6. デザイントークン生成

```bash
/kiro/figma tokens
```

Figma Variablesからコード（Dart/Swift）を生成。

---

## 継続的な同期

### デザイン変更の検出

```bash
# 定期的に実行（例: 毎日1回）
/kiro/figma diff navigation-display
```

Figmaデザインの変更を検出し、影響範囲を報告。

### 仕様の更新

```bash
# 手動レビュー後、仕様を更新
/kiro/spec navigation-display design --sync-figma
/kiro/spec navigation-display tasks --sync-figma
```

### 自動同期（慎重に使用）

```bash
# 完全自動同期（既存の手動編集が上書きされる可能性あり）
/kiro/figma sync navigation-display --auto
```

---

## トラブルシューティング

### 接続エラー

**問題**: Figmaに接続できない

**解決策**:
1. `.env` の `FIGMA_ACCESS_TOKEN` を確認
2. トークンが有効か確認（Figma Settingsで再生成可能）
3. ファイルへのアクセス権限を確認

```bash
# 環境変数が読み込まれているか確認
echo $FIGMA_ACCESS_TOKEN
```

### コンポーネントが見つからない

**問題**: 指定したコンポーネントが見つからない

**解決策**:
1. Figma内でコンポーネント名を確認（大文字・小文字区別）
2. コンポーネントが公開されているか確認
3. 利用可能なコンポーネント一覧を表示:

```bash
/kiro/figma list
```

### デザイントークンが生成されない

**問題**: `/kiro/figma tokens` で何も生成されない

**解決策**:
1. Figma Variablesを使用しているか確認（Local Stylesではなく）
2. Variablesが公開されているか確認
3. 少なくとも1つのVariableが定義されているか確認

---

## ベストプラクティス

### 1. Figma変数の命名

**推奨**:
- `Primary/Blue` ✅
- `Background/Primary` ✅
- `Spacing/M` ✅

**非推奨**:
- `blue` ❌（カテゴリがない）
- `primary_color` ❌（アンダースコア）
- `背景色` ❌（日本語）

### 2. コンポーネント命名

コードと一致させる:
- Figma: `Button/Primary`
- Flutter: `PrimaryButton`
- SwiftUI: `PrimaryButton`

### 3. 定期的な同期

週に1回はFigmaの変更をチェック:
```bash
/kiro/figma diff [all-features]
```

### 4. スナップショットの管理

重要なマイルストーンでスナップショットを保存:
```bash
# Figmaの現在の状態をスナップショット
/kiro/figma snapshot --tag "v1.0-release"
```

---

## チーム共有

### 設定ファイルの共有

**コミットする**:
- ✅ `.figma-config.template.json` - 設定のテンプレート
- ✅ `.kiro/FIGMA_SETUP.md` - このセットアップガイド
- ✅ `.kiro/figma-review.md` - Figmaレビュー結果

**コミットしない**:
- ❌ `.env` - 個人のアクセストークン
- ❌ `.figma-config.json` - 個人の設定
- ❌ `.kiro/figma-snapshots/` (optional) - スナップショット

### 新しいチームメンバーの追加

1. このガイドを読む
2. Figma Access Tokenを取得
3. `.env` ファイルを作成
4. `.figma-config.json` をコピー・編集
5. 接続テスト: `/kiro/figma review [url]`

---

## 参考リンク

- [Figma API Documentation](https://www.figma.com/developers/api)
- [Figma Variables Guide](https://help.figma.com/hc/en-us/articles/15339657135383-Guide-to-variables-in-Figma)
- [Figma Prototyping](https://help.figma.com/hc/en-us/articles/360040314193-Guide-to-prototyping-in-Figma)
- [Anthropic MCP Documentation](https://docs.anthropic.com/mcp)

---

## まとめ

Figma MCPの統合により:
- ✅ デザインから仕様への自動変換
- ✅ デザイントークンのコード生成
- ✅ アセットの自動エクスポート
- ✅ デザイン変更の影響範囲可視化
- ✅ デザインとコードの一貫性保証

セットアップ完了後、`/kiro/figma review [your-url]` から始めましょう！

