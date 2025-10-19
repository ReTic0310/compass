# Migration Guide: cc-sdd v1 → Kiro SDD v2

cc-sddからカスタマイズされたKiro SDD v2への移行ガイド

## 変更概要

### コマンド統合

**旧システム (10コマンド)**:
```
/kiro/steering
/kiro/steering-custom
/kiro/spec-init
/kiro/spec-requirements
/kiro/spec-design
/kiro/spec-tasks
/kiro/spec-impl
/kiro/spec-status
/kiro/validate-design
/kiro/validate-gap
```

**新システム (4コマンド)**:
```
/kiro/init       # steering + steering-custom + spec-init
/kiro/spec       # spec-requirements + spec-design + spec-tasks + spec-impl + validate-*
/kiro/status     # spec-status (enhanced)
/kiro/test       # 新機能: テスト戦略とTDD/BDD
```

---

## コマンド対応表

| 旧コマンド | 新コマンド | 備考 |
|-----------|-----------|------|
| `/kiro/steering` | `/kiro/init project` | プロジェクト初期化に統合 |
| `/kiro/steering-custom` | `/kiro/init project` | 自動的にカスタムステアリング生成 |
| `/kiro/spec-init [desc]` | `/kiro/init feature [desc]` | feature初期化に名称変更 |
| `/kiro/spec-requirements [f]` | `/kiro/spec [f] requirements` | spec配下に統合 |
| `/kiro/spec-design [f]` | `/kiro/spec [f] design` | spec配下に統合 |
| `/kiro/spec-tasks [f]` | `/kiro/spec [f] tasks` | spec配下に統合 |
| `/kiro/spec-impl [f]` | `/kiro/spec [f] impl` | spec配下に統合 |
| `/kiro/spec-status [f]` | `/kiro/status [f]` | 名称変更・機能拡張 |
| `/kiro/validate-design [f]` | `/kiro/spec [f] design` | design内に統合 |
| `/kiro/validate-gap [f]` | `/kiro/spec [f] design` | design内に統合 |

---

## 移行手順

### Step 1: 既存プロジェクトの確認

既存の `.kiro/` 構造を確認:
```bash
tree .kiro/
```

既存ファイル:
- `.kiro/steering/` - ステアリングドキュメント
- `.kiro/specs/` - 既存の仕様

### Step 2: 新コマンドファイルの配置確認

新しいコマンドファイルが配置されていることを確認:
```
.cursor/commands/kiro/
├── init.md      # 新規
├── spec.md      # 新規
├── status.md    # 新規
├── test.md      # 新規
└── [旧コマンドファイル群] # 互換性のため残存
```

### Step 3: 既存仕様の互換性確認

既存の `spec.json` に新フィールドを追加:

**必要な新フィールド**:
```json
{
  "platforms": ["flutter", "swift"],
  "affected_platforms": ["flutter", "swift"],
  "dependencies": [],
  "implementation_progress": {
    "total_tasks": 0,
    "completed_tasks": 0,
    "current_task": ""
  }
}
```

**手動更新スクリプト例**:
```bash
# 既存のspec.jsonに新フィールドを追加
for spec in .kiro/specs/*/spec.json; do
  # バックアップ
  cp "$spec" "$spec.bak"
  
  # platforms フィールドを追加（まだない場合）
  jq '. + {platforms: ["flutter"], affected_platforms: ["flutter"], dependencies: []}' "$spec" > "$spec.tmp"
  mv "$spec.tmp" "$spec"
done
```

### Step 4: 新コマンドのテスト

既存の仕様で新コマンドをテスト:

```bash
# ステータス確認
/kiro/status [existing-feature]

# 既存仕様の続きを新コマンドで実行
/kiro/spec [existing-feature]
```

### Step 5: 混成プロジェクト設定（該当する場合）

Flutter + SwiftUIプロジェクトの場合、ステアリングを更新:

```bash
# プロジェクトタイプを混成に設定
/kiro/init project --hybrid "Existing navigation app with Flutter and watchOS"
```

これにより追加生成:
- `flutter-best-practices.md`
- `swift-best-practices.md`
- `data-sync.md`

既存のステアリングは上書きせず、新規追加のみ実行。

---

## 後方互換性

### 旧コマンドのサポート

旧コマンドファイルは **残存** しており、以下は引き続き動作します:

```bash
# 旧コマンド（まだ使える）
/kiro/spec-requirements [feature]
/kiro/spec-design [feature]
/kiro/spec-status [feature]
```

しかし、**新コマンドの使用を推奨** します:
- 機能統合による使いやすさ向上
- 混成プロジェクト対応
- TDD/BDD統合
- 改善された進捗追跡

### 既存仕様ファイルの互換性

既存の仕様ファイル（requirements.md, design.md, tasks.md）は **完全互換** です。

新コマンドは以下をサポート:
- 既存ファイルの読み込み
- マージモード（既存内容を保持）
- 段階的な移行

---

## 主要な新機能

### 1. プラットフォーム対応

新しい `spec.json` フォーマット:
```json
{
  "platforms": ["flutter", "swift"],
  "affected_platforms": ["flutter", "swift"]
}
```

**requirements.md** でプラットフォームターゲットを明示:
```markdown
### Requirement 1.1: Feature Name
**Target Platforms**: Flutter, watchOS

#### Acceptance Criteria
1. WHEN ... THEN ... - _Platform: Flutter_
2. WHEN ... THEN ... - _Platform: watchOS_
3. WHEN ... THEN ... - _Platform: Both (Sync)_
```

### 2. 対話的ワークフロー

```bash
/kiro/spec [feature-name]
```

現在のフェーズを検出し、次のステップを提案:
- 要件生成済み → 設計フェーズを提案
- 設計生成済み → タスクフェーズを提案
- タスク生成済み → 実装開始を提案

### 3. テスト統合

新しい `/kiro/test` コマンド:
```bash
/kiro/test [feature] strategy    # テスト戦略生成
/kiro/test [feature] unit --tdd  # TDDモード
/kiro/test [feature] coverage    # カバレッジ追跡
```

### 4. 依存関係管理

```bash
/kiro/status --dependencies      # 依存関係の可視化
/kiro/status --blockers          # ブロッカーの特定
```

### 5. 多言語サポート

`spec.json` で言語を指定:
```json
{
  "language": "ja"
}
```

英語/日本語の切り替えが可能。

---

## ベストプラクティス

### 新規機能開発

新規機能は **新コマンド** を使用:

```bash
# 推奨: 新コマンド
/kiro/init feature "New feature description"
/kiro/spec [feature-name]
```

### 既存機能の更新

既存機能の更新も **新コマンド** で:

```bash
# 既存機能の要件を更新
/kiro/spec [existing-feature] requirements

# 設計を再生成（マージモード）
/kiro/spec [existing-feature] design
> [m]erge を選択
```

### 段階的移行

一度にすべてを移行する必要はありません:

1. **新機能**: 新コマンドで開発
2. **既存機能**: 必要に応じて新コマンドに移行
3. **完了済み機能**: そのまま（移行不要）

---

## トラブルシューティング

### Q: 旧コマンドが動かない

**A**: 旧コマンドファイルが残っているか確認:
```bash
ls .cursor/commands/kiro/spec-*.md
```

存在しない場合、プロジェクトを再クローンするか、旧ファイルを復元。

### Q: 新コマンドで既存仕様が読めない

**A**: `spec.json` に新フィールドを追加:
```json
{
  "platforms": ["flutter"],
  "affected_platforms": ["flutter"],
  "dependencies": []
}
```

### Q: 混成プロジェクトのステアリングが生成されない

**A**: `--hybrid` フラグを使用:
```bash
/kiro/init project --hybrid "Project description"
```

### Q: テストコマンドが動かない

**A**: 新しい `test.md` が配置されているか確認:
```bash
ls .cursor/commands/kiro/test.md
```

---

## ロールバック

新コマンドに問題がある場合、旧コマンドに戻せます:

### 旧コマンドの使用

旧コマンドファイルは削除されていないため、そのまま使用可能:

```bash
/kiro/spec-requirements [feature]
/kiro/spec-design [feature]
/kiro/spec-status [feature]
```

### 新ファイルの削除

新コマンドで生成されたファイルのみを削除:

```bash
# 新コマンドファイルを削除
rm .cursor/commands/kiro/{init,spec,status,test}.md

# 新ステアリングファイルを削除（必要に応じて）
rm .kiro/steering/{flutter-best-practices,swift-best-practices,data-sync,security}.md
rm .kiro/steering/_registry.json
```

### AGENTS.md の復元

バックアップから `AGENTS.md` を復元:

```bash
# バックアップがあれば
cp AGENTS.md.bak AGENTS.md
```

---

## サポート

問題が発生した場合:

1. **ドキュメント確認**: `AGENTS.md` の Troubleshooting セクション
2. **ステータス確認**: `/kiro/status` で現在の状態を確認
3. **旧コマンド使用**: 緊急時は旧コマンドにフォールバック

---

## まとめ

### ✅ やるべきこと

1. 新コマンドの動作確認
2. 既存 `spec.json` への新フィールド追加（必要に応じて）
3. 新機能は新コマンドで開発
4. テストコマンドの活用

### ❌ やらなくていいこと

1. すべての既存仕様を一度に移行
2. 旧コマンドファイルの削除
3. 完了済み機能の再生成

### 🎯 推奨アクション

新規機能から段階的に新コマンドを使用し、利点を実感してから既存機能の移行を検討してください。

---

## フィードバック

このカスタマイズに関するフィードバックは歓迎します。改善提案があれば、プロジェクトのIssueまたはDiscussionで共有してください。


