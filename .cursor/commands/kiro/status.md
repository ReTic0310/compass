<meta>
description: Show project status, specifications progress, and recommendations
argument-hint: [feature-name]
</meta>

# Project & Specification Status

プロジェクト全体または特定機能の進捗状況を表示します。

Tool policy: Use Cursor file tools (read_file/list_dir/glob_file_search); no shell.

## Usage

```bash
# プロジェクト全体の状態表示
/kiro/status

# 特定機能の詳細な進捗表示
/kiro/status [feature-name]

# 依存関係グラフ表示
/kiro/status --dependencies

# ブロッカー表示
/kiro/status --blockers
```

---

## Project Overview Mode (No Arguments)

### Task: Display Overall Project Status

**Context Loading**:
- `.kiro/steering/_registry.json` (if exists)
- All `.kiro/specs/*/spec.json` files
- `.kiro/steering/` directory contents
- Project root files for health check

**Display Sections**:

#### 1. Project Information
```markdown
# Project Status

## プロジェクト情報
- **名前**: [プロジェクト名]
- **タイプ**: [Single Platform | Hybrid Multi-Platform]
- **プラットフォーム**: [Flutter, Swift/SwiftUI, etc.]
- **初期化日**: [date]
- **最終更新**: [date]
```

#### 2. Steering Status
```markdown
## ステアリング状態

### コアステアリング
- [x] product.md - プロダクト概要
- [x] tech.md - 技術スタック
- [x] structure.md - プロジェクト構造
- [x] security.md - セキュリティガイドライン

### プラットフォーム固有
- [x] flutter-best-practices.md (Flutter)
- [x] swift-best-practices.md (Swift/watchOS)

### ドメイン固有
- [x] data-sync.md - データ同期
```

#### 3. Specifications Overview
```markdown
## 機能仕様の概要

### アクティブな仕様: X件

| 機能名 | フェーズ | 進捗 | プラットフォーム | 最終更新 |
|--------|---------|------|-----------------|---------|
| feature-1 | タスク生成済 | 0/15 | Flutter, watchOS | 2025-10-18 |
| feature-2 | 実装中 | 8/12 | Flutter | 2025-10-17 |
| feature-3 | 完了 | 10/10 | watchOS | 2025-10-15 |

### フェーズ別集計
- 初期化済み: X件
- 要件生成済み: Y件
- 設計生成済み: Z件
- タスク生成済み: W件
- 実装中: V件
- 完了: U件
```

#### 4. Health Checks
```markdown
## プロジェクトヘルスチェック

### ステアリング整合性
- [x] すべてのコアステアリングが存在
- [x] registry.json が有効
- [ ] ⚠️ 3日以上更新されていないステアリングあり

### 仕様整合性
- [x] すべての仕様に spec.json が存在
- [ ] ⚠️ 承認待ちの仕様が2件あります
- [ ] ❌ 1件の仕様で要件カバレッジに欠落あり

### 依存関係
- [x] 循環依存なし
- [ ] ⚠️ feature-1 が feature-2 の完了を待機中

### テストカバレッジ
- Flutter: XX% (目標: 80%)
- watchOS: YY% (目標: 70%)
```

#### 5. Recommendations
```markdown
## 推奨事項

### 🔴 優先度: 高
1. feature-3 の要件に欠落あり - `/kiro/spec feature-3 requirements` で再生成
2. feature-1 が feature-2 をブロック中 - feature-2 を先に完了させる

### 🟡 優先度: 中
1. steering/tech.md が3日間更新されていません - 最近の変更を反映させましょう
2. テストカバレッジが目標未達 - テスト追加を検討

### 🟢 優先度: 低
1. documentation の更新を検討
```

#### 6. Quick Actions
```markdown
## クイックアクション

次にやるべきこと:
- `/kiro/spec feature-2` - 実装中の feature-2 を続行
- `/kiro/spec feature-1 tasks -y` - feature-1 のタスク生成（設計承認済み）
- `/kiro/status feature-2` - feature-2 の詳細を確認
```

---

## Feature Detail Mode (With Feature Name)

### Task: Display Detailed Feature Progress

**Context Loading**:
- `.kiro/specs/[feature-name]/spec.json`
- `.kiro/specs/[feature-name]/requirements.md`
- `.kiro/specs/[feature-name]/design.md` (if exists)
- `.kiro/specs/[feature-name]/tasks.md` (if exists)
- Related steering documents

**Display Sections**:

#### 1. Feature Overview
```markdown
# Feature Status: [feature-name]

## 概要
- **作成日**: [date]
- **最終更新**: [date]
- **言語**: [ja/en]
- **プラットフォーム**: [Flutter | watchOS | Both]
- **現在のフェーズ**: [phase]
- **実装準備**: [Ready | Not Ready]
```

#### 2. Phase Progress Detail
```markdown
## フェーズ進捗

### ✅ 要件定義フェーズ (100%)
- [x] 要件生成済み
- [x] 承認済み
- **要件数**: X件
  - Flutter: Y件
  - watchOS: Z件
  - 同期: W件
- **EARS準拠**: ✅
- **プラットフォームターゲット明確**: ✅

### ✅ 設計フェーズ (100%)
- [x] 設計生成済み
- [x] 承認済み
- **コンポーネント数**:
  - Flutter: X個
  - watchOS: Y個
  - 同期レイヤー: Z個
- **アーキテクチャ図**: ✅
- **データモデル定義**: ✅
- **エラーハンドリング戦略**: ✅

### 🔄 タスクフェーズ (実行中)
- [x] タスク生成済み
- [ ] 承認待ち
- **タスク概要**:
  - 総タスク数: 15
  - Flutter: 6タスク
  - watchOS: 5タスク
  - 同期: 2タスク
  - テスト: 2タスク
- **要件カバレッジ**: 100% (X/X)

### ⏳ 実装フェーズ (未開始)
- 実装開始前
```

OR if in implementation:

```markdown
### 🔄 実装フェーズ (53% 完了)
- **進捗**: 8/15 タスク完了
- **現在のタスク**: 4.2 watchOS側同期レイヤーの開発
- **完了したタスク**:
  - [x] 1. プロジェクト基盤セットアップ
  - [x] 2.1 [サブ機能A] の開発
  - [x] 2.2 [サブ機能B] の開発
  - ...
- **残りタスク**: 7個
  - Flutter: 1個
  - watchOS: 3個
  - 同期: 2個
  - テスト: 1個
```

#### 3. Requirements Traceability
```markdown
## 要件トレーサビリティ

| 要件ID | 要件概要 | 設計コンポーネント | 実装タスク | 状態 |
|--------|---------|------------------|-----------|------|
| 1.1 | [要件] | ComponentA | 2.1 | ✅ 完了 |
| 1.2 | [要件] | ComponentB | 2.2 | 🔄 実装中 |
| 2.1 | [要件] | ServiceX | 3.1 | ⏳ 未着手 |
```

#### 4. Platform Breakdown
```markdown
## プラットフォーム別進捗

### Flutter (Android/iOS)
- **要件**: 5件
- **コンポーネント**: 3個
- **タスク**: 6個 (4完了, 2残)
- **テスト**: 
  - Unit tests: 12件 (10 pass, 2 pending)
  - Widget tests: 8件 (8 pass)
- **ステアリング準拠**: ✅ flutter-best-practices.md

### watchOS
- **要件**: 4件
- **コンポーネント**: 2個
- **タスク**: 5個 (3完了, 2残)
- **テスト**:
  - Unit tests: 8件 (6 pass, 2 pending)
  - UI tests: 4件 (pending)
- **ステアリング準拠**: ✅ swift-best-practices.md

### データ同期
- **要件**: 3件
- **コンポーネント**: 同期レイヤー
- **タスク**: 3個 (1完了, 2残)
- **テスト**: 
  - Mock tests: 4件 (2 pass, 2 pending)
  - 実機テスト: pending
- **ステアリング準拠**: ✅ data-sync.md
```

#### 5. Quality Metrics
```markdown
## 品質メトリクス

### 設計品質
- **アーキテクチャ整合性**: ✅ 既存パターンに準拠
- **コンポーネント結合度**: 低 (Good)
- **責務の明確さ**: ✅ 単一責任原則遵守
- **型安全性**: ✅ `any` 型の不使用

### 要件品質
- **EARS準拠率**: 100%
- **テスト可能性**: ✅ すべての要件が観測可能
- **曖昧性**: なし
- **プラットフォーム明確性**: ✅

### テスト品質
- **カバレッジ**: 
  - Flutter: 85% (目標: 80%)
  - watchOS: 72% (目標: 70%)
- **テストタイプバランス**: ✅
  - Unit: 60%
  - Integration: 25%
  - E2E: 15%
```

#### 6. Dependencies & Blockers
```markdown
## 依存関係とブロッカー

### このFeatureが依存
- `feature-auth` - 認証システム (✅ 完了)
- `feature-location` - 位置情報サービス (🔄 実装中)

### このFeatureをブロック中
- `feature-notifications` - 通知システム (⏳ 待機中)

### 外部依存
- `watch_connectivity` package (Flutter)
- WatchConnectivity Framework (iOS)
```

#### 7. Risk Assessment
```markdown
## リスク評価

### 🔴 高リスク
- WatchConnectivityの実機テストが未実施 - 早期の実機検証が必要

### 🟡 中リスク
- watchOSのバッテリー消費が未測定 - パフォーマンステストの追加を推奨

### 🟢 低リスク
- テストカバレッジは良好
```

#### 8. Recommendations & Next Steps
```markdown
## 推奨事項

### 次のステップ
1. **即座に**: `/kiro/spec [feature-name] impl 4.2` で現在のタスクを続行
2. **レビュー後**: 完了したコンポーネントのコードレビュー
3. **テスト**: 実機でのWatchConnectivity動作確認

### 改善提案
- watchOSのバッテリー効率テストを追加
- オフラインシナリオのテストカバレッジ強化
- エラーメッセージの多言語対応検討
```

---

## Dependencies Mode

### Task: Visualize Feature Dependencies

```bash
/kiro/status --dependencies
```

**Output**:
```markdown
# Feature Dependencies

## 依存関係グラフ

\`\`\`mermaid
graph TD
    Auth[feature-auth: 完了]
    Location[feature-location: 実装中]
    Navigation[feature-navigation: タスク生成済]
    Notifications[feature-notifications: 要件生成済]
    
    Navigation -->|depends on| Auth
    Navigation -->|depends on| Location
    Notifications -->|depends on| Navigation
    
    style Auth fill:#90EE90
    style Location fill:#FFD700
    style Navigation fill:#87CEEB
    style Notifications fill:#D3D3D3
\`\`\`

## 実装順序の推奨
1. ✅ feature-auth (完了)
2. 🔄 feature-location (実装中) ← 優先
3. ⏳ feature-navigation (location完了待ち)
4. ⏳ feature-notifications (navigation完了待ち)

## 循環依存チェック
✅ 循環依存は検出されませんでした
```

---

## Blockers Mode

### Task: Identify and Report Blockers

```bash
/kiro/status --blockers
```

**Output**:
```markdown
# Blockers & Issues

## 🚫 クリティカルブロッカー

### 1. feature-navigation のブロッカー
- **ブロックされている内容**: タスク 2.3 の実装開始不可
- **原因**: feature-location の位置情報サービスAPIが未完成
- **影響**: 3タスク (2.3, 2.4, 3.1) が待機状態
- **解決策**: `/kiro/spec feature-location` で feature-location を優先実装
- **推定遅延**: 2-3日

## ⚠️ 承認待ち

### 1. feature-maps
- **フェーズ**: 設計生成済み
- **状態**: レビュー・承認待ち
- **待機時間**: 2日
- **アクション**: design.md をレビューして `/kiro/spec feature-maps tasks -y` を実行

## 🔍 潜在的問題

### 1. テストカバレッジ不足
- **Feature**: feature-sync
- **現在**: 45% (目標: 80%)
- **推奨**: 統合テストとエッジケーステストの追加

### 2. 外部依存のバージョン競合
- **Feature**: feature-notifications
- **問題**: `flutter_local_notifications` と `firebase_messaging` のバージョン互換性
- **推奨**: pubspec.yaml の依存関係解決
```

---

## Implementation Details

### Parsing spec.json
```typescript
interface SpecMetadata {
  feature_name: string;
  created_at: string;
  updated_at: string;
  language: "ja" | "en";
  platforms: string[];
  phase: string;
  approvals: {
    requirements: { generated: boolean; approved: boolean };
    design: { generated: boolean; approved: boolean };
    tasks: { generated: boolean; approved: boolean };
  };
  dependencies?: string[];
  affected_platforms: string[];
  implementation_progress?: {
    total_tasks: number;
    completed_tasks: number;
    current_task: string;
  };
  ready_for_implementation: boolean;
}
```

### Parsing tasks.md
- Count total checkboxes: `- [ ]` and `- [x]`
- Calculate completion percentage
- Extract platform tags: `_Platform: Flutter|watchOS|Both_`
- Extract requirement mappings: `_Requirements: X.X_`

### Health Check Logic
1. **Steering integrity**: すべてのコアファイルが存在するか
2. **Spec consistency**: すべてのspecにspec.jsonが存在するか
3. **Approval flow**: 適切な順序で承認されているか
4. **Requirements coverage**: すべての要件がタスクにマップされているか
5. **Dependency resolution**: 循環依存がないか

---

## Output Language

`spec.json` の `language` フィールドに基づいて出力言語を決定:
- `"ja"`: 日本語で出力
- `"en"`: 英語で出力
- 未定義: デフォルトは日本語

---

## Error Handling

### No Specs Found
```
ℹ️ アクティブな仕様が見つかりません

機能の追加を開始:
/kiro/init feature <機能説明>
```

### Invalid spec.json
```
❌ エラー: [feature-name]/spec.json が無効です

spec.json を修正するか、`/kiro/init feature` で再初期化してください
```

### Missing Files
```
⚠️ 警告: [feature-name] で不整合を検出

- design.md が存在しますが、requirements.md がありません

推奨: `/kiro/spec [feature-name] requirements` で要件を生成
```

---

## Success Criteria

- [ ] プロジェクト全体の状態を1画面で把握できる
- [ ] 各機能の詳細な進捗が明確
- [ ] ブロッカーと依存関係が可視化されている
- [ ] 次に取るべきアクションが明確
- [ ] プラットフォーム別の進捗が分かる
- [ ] 品質メトリクスが追跡されている

ultrathink
