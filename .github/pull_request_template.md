# Pull Request: [Title]

<!-- 
このテンプレートはKiro SDD v2のGit統合により自動生成されます。
手動でPRを作成する場合は、下記を参考にしてください。
-->

## 種類
<!-- 該当するものにチェックを入れてください -->
- [ ] Requirements (要件定義)
- [ ] Design (技術設計)
- [ ] Tasks (実装タスク)
- [ ] Implementation (実装)
- [ ] Bugfix (バグ修正)
- [ ] Test (テスト追加)
- [ ] Refactor (リファクタリング)
- [ ] Documentation (ドキュメント)

## 概要
<!-- このPRの目的と変更内容を簡潔に説明してください -->

## 詳細

### Requirements PRの場合
- **プラットフォーム**: Flutter / watchOS / Both
- **要件数**: X件
- **EARS準拠**: ✅ / ❌

#### 要件概要
<!-- 主要な要件を箇条書きで -->

### Design PRの場合
- **関連Requirements PR**: #XXX
- **コンポーネント数**: X個
- **アーキテクチャ図**: ✅ / ❌
- **プラットフォームカバレッジ**: Flutter / watchOS / Both

#### 主要な設計決定
1. 
2. 
3. 

### Tasks PRの場合
- **関連Design PR**: #XXX
- **総タスク数**: X個
- **プラットフォーム内訳**:
  - Flutter: X個
  - watchOS: Y個
  - 同期: Z個
  - テスト: W個

### Implementation PRの場合
- **完了タスク**: X/Y
- **プラットフォーム**: Flutter / watchOS / Both
- **テスト**: ✅ / ⏳ / ❌

#### 実装内容
<!-- 実装した機能や変更点を説明 -->

#### スクリーンショット/動画
<!-- 該当する場合、UIの変更やデモを添付 -->

## チェックリスト

### Requirements
- [ ] すべての要件がEARS形式
- [ ] プラットフォームターゲットが明確
- [ ] 要件がテスト可能
- [ ] 曖昧な表現がない

### Design
- [ ] アーキテクチャ図が含まれている
- [ ] すべての要件がカバーされている
- [ ] コンポーネントインターフェースが定義されている
- [ ] データモデルが文書化されている
- [ ] エラーハンドリング戦略が定義されている
- [ ] テスト戦略の概要がある

### Tasks
- [ ] すべての要件がタスクにマップされている
- [ ] タスクが適切にサイズ調整されている（1-3時間）
- [ ] プラットフォームターゲットが明確
- [ ] 依存関係が特定されている
- [ ] テストタスクが含まれている

### Implementation
- [ ] ユニットテストが通る
- [ ] Widgetテスト/UIテストが通る（該当する場合）
- [ ] 統合テストが通る
- [ ] 手動テストが完了
- [ ] コードレビューの準備ができている
- [ ] パフォーマンスが許容範囲内
- [ ] Breaking Changesがない（またはドキュメント化されている）

### Code Quality
- [ ] Lintエラーがない
- [ ] コードがプロジェクトの規約に従っている
- [ ] 適切なコメントが追加されている
- [ ] 型安全性が保たれている（`any`型を使用していない）

## 関連ファイル
<!-- 変更されたファイルやドキュメントのパス -->
- `.kiro/specs/[feature-name]/`

## 影響範囲
<!-- このPRが影響する範囲を記述 -->
- [ ] Flutter (Android)
- [ ] Flutter (iOS)
- [ ] watchOS
- [ ] データ同期レイヤー
- [ ] 共通ライブラリ
- [ ] ドキュメント

## テスト方法
<!-- レビュアーがこのPRをテストする方法 -->

### Flutter
```bash
flutter test
flutter run
```

### watchOS
```bash
xcodebuild test -workspace ... -scheme ...
```

## 次のステップ
<!-- このPRがマージされた後の次のアクション -->

## 備考
<!-- その他、レビュアーに伝えたいこと -->

---

<!-- 
Kiro SDD v2 Workflow:
- Phase: [requirements/design/tasks/implementation]
- Feature: [feature-name]
- Branch: [branch-name]
-->

