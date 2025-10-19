# Kiro SDD - Hybrid Multi-Platform Spec-Driven Development

カスタマイズされた仕様駆動開発フレームワーク - Flutter + SwiftUI混成プロジェクト対応

## Project Context

### Project Type
- **Type**: Hybrid Multi-Platform Navigation App
- **Platforms**: Android (Flutter), iOS (Flutter), watchOS (SwiftUI)
- **Architecture**: Feature-based modular architecture with cross-platform data sync

### Directory Structure
```
compass/
├── flutter_app/           # Flutter アプリケーション (Android/iOS)
├── watch_app/             # watchOS アプリケーション (SwiftUI)
├── shared_models/         # 共有データモデル定義
├── .kiro/                 # 仕様駆動開発ファイル
│   ├── steering/          # プロジェクト全体ガイドライン
│   └── specs/             # 機能別仕様
└── .cursor/commands/kiro/ # Kiro コマンド定義
```

### Core Concepts

**Steering** (`.kiro/steering/`) - プロジェクト全体のルールとコンテキストでAIをガイド
- コアステアリング: product, tech, structure, security
- プラットフォーム固有: flutter-best-practices, swift-best-practices
- ドメイン固有: data-sync

**Specs** (`.kiro/specs/`) - 個別機能の開発プロセスを形式化
- 要件定義 (requirements.md)
- 技術設計 (design.md)
- 実装タスク (tasks.md)
- テスト戦略 (test-strategy.md)

---

## Getting Started

### First-Time Setup

1. **プロジェクト初期化** (混成プロジェクト)
   ```bash
   /kiro/init project --hybrid "Navigation app for Android, iOS, and Apple Watch"
   ```
   
   これにより自動生成されるもの:
   - `.kiro/steering/` 配下のすべてのステアリングドキュメント
   - プラットフォーム固有のベストプラクティスガイド
   - データ同期戦略ドキュメント

2. **ステータス確認**
   ```bash
   /kiro/status
   ```

### Creating Your First Feature

1. **機能仕様の初期化**
   ```bash
   /kiro/init feature "User can view real-time navigation on both iPhone and Apple Watch"
   ```

2. **統合ワークフローで進める** (推奨)
   ```bash
   /kiro/spec [feature-name]
   ```
   
   対話的に以下を実行:
   - 要件定義生成 → レビュー
   - 技術設計生成 → レビュー
   - タスク生成 → レビュー
   - 実装開始

---

## Simplified Command Structure

### 🆕 統合コマンド (学習コスト削減)

#### `/kiro/init` - 初期化コマンド
```bash
# プロジェクト初期化
/kiro/init project [--hybrid] <description>

# 機能仕様初期化
/kiro/init feature <description>
```

#### `/kiro/spec` - 統合仕様ワークフロー
```bash
# 対話的モード (推奨)
/kiro/spec [feature-name]

# 特定フェーズ実行
/kiro/spec [feature-name] requirements
/kiro/spec [feature-name] design [-y]
/kiro/spec [feature-name] tasks [-y]
/kiro/spec [feature-name] impl [task-numbers]

# 自動承認フラグ
/kiro/spec [feature-name] design -y    # 要件を自動承認
/kiro/spec [feature-name] tasks -y     # 要件と設計を自動承認
```

#### `/kiro/status` - 進捗確認
```bash
# プロジェクト全体のステータス
/kiro/status

# 特定機能の詳細
/kiro/status [feature-name]

# 依存関係表示
/kiro/status --dependencies

# ブロッカー表示
/kiro/status --blockers
```

#### `/kiro/test` - テスト戦略と生成
```bash
# テスト戦略生成
/kiro/test [feature-name] strategy

# テスト生成
/kiro/test [feature-name] unit
/kiro/test [feature-name] integration
/kiro/test [feature-name] e2e

# TDDモード
/kiro/test [feature-name] unit --tdd

# カバレッジレポート
/kiro/test [feature-name] coverage
```

---

## Development Workflow

### 📋 Standard Workflow

```mermaid
graph TB
    Start([Start]) --> Init[/kiro/init feature]
    Init --> Spec[/kiro/spec feature-name]
    Spec --> Req{要件レビュー}
    Req -->|OK| Design[設計生成]
    Req -->|修正| Spec
    Design --> Des{設計レビュー}
    Des -->|OK| Tasks[タスク生成]
    Des -->|修正| Design
    Tasks --> Task{タスクレビュー}
    Task -->|OK| Impl[実装開始]
    Task -->|修正| Tasks
    Impl --> Test[テスト]
    Test --> Done([Complete])
```

### Detailed Phase Breakdown

#### Phase 1: Requirements (要件定義)
**Goal**: 何を作るかを明確にする

**Input**: 機能の説明
**Output**: `requirements.md` - EARS形式の要件定義

**Checklist**:
- [ ] すべての要件がEARS形式（WHEN/IF/WHILE/WHERE）
- [ ] プラットフォームターゲットが明確（Flutter/watchOS/Both）
- [ ] 同期要件が定義されている（クロスプラットフォームの場合）
- [ ] 要件が観測可能でテスト可能
- [ ] 曖昧な表現がない

**Next**: レビュー後、`/kiro/spec [feature-name]` で設計フェーズへ

#### Phase 2: Design (技術設計)
**Goal**: どのように作るかを設計する

**Input**: 承認された要件
**Output**: `design.md` - 技術設計ドキュメント

**Checklist**:
- [ ] アーキテクチャ図が明確
- [ ] プラットフォーム別のコンポーネント定義
- [ ] データモデルの定義（共有スキーマ含む）
- [ ] 同期レイヤーの設計（クロスプラットフォームの場合）
- [ ] エラーハンドリング戦略
- [ ] テスト戦略の概要
- [ ] セキュリティ考慮事項（該当する場合）

**Next**: レビュー後、`/kiro/spec [feature-name]` でタスク生成へ

#### Phase 3: Tasks (タスク分解)
**Goal**: 実装可能な単位にタスクを分解

**Input**: 承認された設計
**Output**: `tasks.md` - 実装タスクリスト

**Checklist**:
- [ ] タスクは自然言語で記述（コード構造ではなく機能）
- [ ] プラットフォーム別にタスクが分離
- [ ] 同期タスクが明確に定義
- [ ] すべての要件がタスクにマップされている
- [ ] タスクの依存関係が適切
- [ ] テストタスクが含まれている

**Next**: レビュー後、`/kiro/spec [feature-name] impl` で実装開始

#### Phase 4: Implementation (実装)
**Goal**: TDDでタスクを実装

**Process**:
1. テストを先に書く (RED)
2. テストを通す最小実装 (GREEN)
3. コードをリファクタリング (REFACTOR)

**Commands**:
```bash
# 次のタスクを実行
/kiro/spec [feature-name] impl

# 特定タスクを実行
/kiro/spec [feature-name] impl 2.1

# テスト生成（TDDモード）
/kiro/test [feature-name] unit --tdd
```

---

## Hybrid Multi-Platform Features

### Cross-Platform Development Support

#### 1. Platform-Specific Steering
各プラットフォームに特化したベストプラクティスが自動適用:
- **Flutter作業時**: `flutter-best-practices.md` が自動ロード
- **watchOS作業時**: `swift-best-practices.md` が自動ロード
- **同期実装時**: `data-sync.md` が常時ロード

#### 2. Platform Targeting in Requirements
要件にプラットフォームターゲットを明示:
```markdown
### Requirement 1.1: Display Navigation Route
**Target Platforms**: Flutter, watchOS

**Objective**: As a user, I want to see my navigation route on both my phone and watch

#### Acceptance Criteria
1. WHEN user starts navigation on Flutter app THEN route SHALL be displayed on phone
   - _Platform: Flutter_
2. WHEN route is displayed on Flutter THEN route data SHALL sync to watchOS
   - _Platform: Both (Sync)_
3. WHEN route data arrives on watchOS THEN route SHALL be displayed on watch
   - _Platform: watchOS_
```

#### 3. Dual-Platform Design
設計ドキュメントがプラットフォームごとにセクション分け:
```markdown
## Platform Architecture

### Flutter Implementation
[Flutter-specific design]

### watchOS Implementation
[watchOS-specific design]

### Data Synchronization
[Sync layer design]
```

#### 4. Separated Tasks with Sync Integration
タスクがプラットフォーム別に分離され、統合タスクも含む:
```markdown
## Flutter Implementation
- [ ] 2.1 Flutter: ナビゲーション表示機能
  - _Platform: Flutter_

## watchOS Implementation
- [ ] 3.1 watchOS: ナビゲーション表示機能
  - _Platform: watchOS_

## Data Synchronization
- [ ] 4.1 データ同期レイヤーの実装
  - _Platform: Both_
```

---

## Advanced Features

### Test-Driven Development (TDD) Integration

Kiroは完全なTDD/BDDサポートを提供:

```bash
# TDDモードでテストを先に生成
/kiro/test [feature-name] unit --tdd

# BDD形式（Gherkin）のテスト生成
/kiro/test [feature-name] integration --bdd
```

**TDDワークフロー**:
1. 🔴 RED: 失敗するテストを生成
2. 🟢 GREEN: テストを通す最小実装
3. 🔵 REFACTOR: コードを改善

### Security & Best Practices

すべての機能で自動的にセキュリティチェック:
- 機密データのハードコーディング検出
- 入力バリデーション要件
- セキュアな通信（HTTPS）の強制
- プラットフォーム固有のセキュリティ（Keychain, secure storage）

### AI Memory Management

大規模仕様への対応:
- モジュール境界の明確化
- 依存関係の可視化
- コンテキスト制約への適応
- 段階的な仕様更新

---

## Development Guidelines

### Language Policy
- **思考**: 英語で思考（AIの精度向上）
- **生成**: 日本語で出力（`spec.json`の`language`フィールドで制御）
- **切替**: `spec.json`を編集して言語切替可能

### Code Quality Standards
1. **型安全性**: `any`型の使用禁止（Flutter/Dart, Swift両方）
2. **Null安全性**: Null Safetyの徹底
3. **テストカバレッジ**: Flutter 80%以上、watchOS 70%以上
4. **ベストプラクティス準拠**: 各プラットフォームの公式ガイドライン遵守

### Approval Workflow
各フェーズで明示的な承認が必要:
- 自動承認フラグ (`-y`) を使用可能
- レビューなしでの進行は推奨されない
- 承認状態は `spec.json` で追跡

### Dependency Management
依存関係を明確に:
```json
{
  "dependencies": ["feature-auth", "feature-location"],
  "blocked_by": [],
  "blocking": ["feature-notifications"]
}
```

---

## Troubleshooting

### Common Issues

**Q: コマンドが多すぎて覚えられない**
A: 主に3つだけ覚えればOK:
- `/kiro/init` - 初期化
- `/kiro/spec` - 仕様作成・実装
- `/kiro/status` - 進捗確認

**Q: 混成プロジェクトでどのファイルを編集すればいい？**
A: `/kiro/status [feature-name]` でプラットフォーム別の進捗を確認してください。

**Q: テストをどう書けばいい？**
A: `/kiro/test [feature-name] strategy` でテスト戦略を生成後、`/kiro/test [feature-name] unit --tdd` でTDD形式で進めてください。

**Q: 仕様変更時の整合性が心配**
A: 各フェーズのファイルは相互参照されています。変更時は該当フェーズから再生成することで整合性を保ちます。

**Q: AIが仕様を勝手に補填するのが心配**
A: 各フェーズで明示的なレビューと承認を行うことで防止できます。曖昧な部分はコメントで明記されます。

---

## Steering Configuration

### Active Steering Files

#### Core Steering (Always Loaded)
- `product.md` - Product context and business objectives
- `tech.md` - Technology stack and architectural decisions
- `structure.md` - Project structure and file organization
- `security.md` - Security guidelines and best practices

#### Platform-Specific Steering (Conditional)
- `flutter-best-practices.md` - Loaded when working with Flutter files
  - Pattern: `flutter_app/**/*`, `lib/**/*`, `*.dart`
- `swift-best-practices.md` - Loaded when working with Swift files
  - Pattern: `watch_app/**/*`, `*.swift`

#### Domain-Specific Steering (Always Loaded)
- `data-sync.md` - Cross-platform data synchronization strategy

### Custom Steering Files
<!-- Additional custom steering can be added via /kiro/init project -->

### Inclusion Modes
- **Always**: すべてのインタラクションで読み込まれる
- **Conditional**: 特定のファイルパターンでのみ読み込まれる
- **Manual**: `@filename.md` で明示的に参照

---

## Active Specifications

<!-- Managed by /kiro/init feature -->
<!-- Format:
### [feature-name]
- **Description**: [brief description]
- **Phase**: [current phase]
- **Platforms**: [Flutter | watchOS | Both]
- **Status**: [Active | Completed | Blocked]
-->

---

## Project Evolution

### Version History
- **v2.0** (Current): Hybrid multi-platform support, simplified commands, TDD integration
- **v1.0**: Original cc-sdd implementation

### Roadmap
- [ ] AI memory persistence for large specifications
- [ ] Automated dependency resolution
- [ ] CI/CD integration for test execution
- [ ] Visual specification editor
- [ ] Multi-language documentation support (English/Japanese)

---

## Git/GitHub Integration

### 🆕 バージョン管理統合

Kiro SDD v2は完全なGit/GitHub統合を提供します。

#### セットアップ

```bash
# SSH鍵セットアップ（初回のみ・推奨）
ssh-keygen -t ed25519 -C "your_email@example.com"
# → GitHubに公開鍵を登録
# 詳細: .kiro/SSH_SETUP.md

# Gitリポジトリ初期化（SSH認証・推奨）
/kiro/git init git@github.com:username/compass.git

# または短縮形
/kiro/git init username/compass

# GitHub CLI認証（PR作成に必要）
gh auth login  # SSH選択
```

#### 基本ワークフロー

```bash
# 各フェーズで自動的にブランチ・コミット・PR作成
/kiro/spec [feature-name] requirements --git
# → ブランチ: feature/[feature-name]-requirements
# → コミット: docs([feature-name]): add requirements
# → PR #123作成（Draft）

# PR承認・マージ後、次のフェーズ
/kiro/spec [feature-name] design --git
# → ブランチ: feature/[feature-name]-design
# → コミット: docs([feature-name]): add design
# → PR #124作成（Draft）

# タスクと実装も同様
/kiro/spec [feature-name] tasks --git
/kiro/spec [feature-name] impl --git
```

#### Git コマンド

```bash
# ブランチ作成
/kiro/git branch [feature-name] [phase]

# コミット
/kiro/git commit [feature-name] [phase]

# PR作成
/kiro/git pr [feature-name] [phase]

# ステータス確認
/kiro/git status

# 同期
/kiro/git sync
```

#### ブランチ戦略

```
main (protected)
├── feature/[feature-name]-requirements → PR → Merge
├── feature/[feature-name]-design → PR → Merge
├── feature/[feature-name]-tasks → PR → Merge
└── feature/[feature-name]-impl → PR → Merge
```

#### 利点

- ✅ フェーズごとに明確なPR
- ✅ レビュープロセスの統合
- ✅ 仕様とコードのバージョン管理
- ✅ 自動化されたコミットメッセージ
- ✅ クリーンなGit履歴

詳細: `.kiro/steering/git-workflow.md`

---

## Figma Integration

### 🎨 デザインツール統合

Figma MCPを使用したデザイン駆動開発。

#### セットアップ

```bash
# 環境変数設定（.env）
FIGMA_ACCESS_TOKEN=your_token

# 設定ファイル準備
cp .figma-config.template.json .figma-config.json
# Figma File Keyを編集
```

#### 基本ワークフロー

```bash
# Figmaデザインをレビュー
/kiro/figma review https://www.figma.com/file/ABC123/App

# Figmaから要件を自動生成
/kiro/spec [feature-name] requirements --from-figma [frame-name]

# Figma参照で設計生成
/kiro/spec [feature-name] design --with-figma

# デザイントークンをコード化
/kiro/figma tokens

# アセットをエクスポート
/kiro/figma export [component-name]

# デザイン変更を検出
/kiro/figma diff [feature-name]
```

#### 利点

- ✅ デザインから仕様を自動生成
- ✅ デザイントークンの自動コード化
- ✅ アセットの自動エクスポート
- ✅ デザイン変更の影響範囲可視化

詳細: `.kiro/FIGMA_SETUP.md`, `.kiro/steering/figma-integration.md`

---

## Quick Reference

### Essential Commands
```bash
# 初期化
/kiro/init project --hybrid "<description>"
/kiro/init feature "<description>"

# 開発
/kiro/spec [feature-name]                    # 対話的モード
/kiro/spec [feature-name] requirements       # 要件生成
/kiro/spec [feature-name] design -y          # 設計生成（自動承認）
/kiro/spec [feature-name] tasks -y           # タスク生成（自動承認）
/kiro/spec [feature-name] impl               # 実装

# Git統合（SSH認証推奨）
/kiro/git init git@github.com:user/repo      # Git初期化（SSH）
/kiro/git init user/repo                     # Git初期化（短縮形）
/kiro/spec [feature-name] requirements --git # 要件+Git自動化
/kiro/git pr [feature-name] [phase]          # PR作成

# Figma統合
/kiro/figma review [url]                     # Figmaレビュー
/kiro/figma tokens                           # デザイントークン生成
/kiro/spec [feature-name] design --with-figma # Figma参照設計

# テスト
/kiro/test [feature-name] strategy           # テスト戦略
/kiro/test [feature-name] unit --tdd         # TDDモード

# 進捗確認
/kiro/status                                 # 全体ステータス
/kiro/status [feature-name]                  # 機能詳細
/kiro/status --dependencies                  # 依存関係
/kiro/status --blockers                      # ブロッカー
/kiro/git status                             # Gitステータス
```

### File Locations
```
.kiro/
├── steering/           # プロジェクト全体ガイドライン
└── specs/
    └── [feature-name]/
        ├── spec.json               # メタデータ
        ├── requirements.md         # 要件定義
        ├── design.md              # 技術設計
        ├── tasks.md               # 実装タスク
        └── test-strategy.md       # テスト戦略
```

