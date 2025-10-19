# Git/GitHub Integration - Quick Start Guide

Kiro SDD v2でGit/GitHubを使い始めるための5分ガイド

---

## 前提条件

### 必須
- ✅ Git インストール済み
- ✅ GitHub アカウント

### オプション（PR作成に必要）
- GitHub CLI (`gh`) インストール

---

## セットアップ（3ステップ）

### Step 1: SSH鍵セットアップ（推奨）

```bash
# SSH鍵がない場合、生成
ssh-keygen -t ed25519 -C "your_email@example.com"

# GitHubに公開鍵を登録
cat ~/.ssh/id_ed25519.pub
# → GitHub Settings → SSH and GPG keys に追加

# 接続テスト
ssh -T git@github.com
```

詳細: `.kiro/SSH_SETUP.md`

### Step 2: Git初期化

```bash
# SSH認証（推奨）
/kiro/git init git@github.com:username/compass.git

# または短縮形
/kiro/git init username/compass

# HTTPS認証（オプション）
/kiro/git init https://github.com/username/compass.git
```

これにより自動実行：
- `git init`
- `git remote add origin [url]`
- `.gitignore` 作成（まだない場合）
- Initial commit
- `git push -u origin main`

**出力例**:
```
✅ Git repository initialized

Remote: origin → https://github.com/username/compass.git
Branch: main
Initial commit: ✅

Next steps:
1. Configure GitHub CLI: gh auth login
2. Start developing: /kiro/init feature "..."
```

### Step 3: GitHub CLI認証（PR作成用）

```bash
gh auth login
```

インタラクティブに以下を選択：
- GitHub.com
- **SSH** （推奨）
- Login with a web browser

### Step 4: 設定確認

```bash
/kiro/git status
```

**出力例**:
```
# Git Status

## Repository
- Remote: origin → https://github.com/username/compass.git
- Current branch: main
- Status: Clean

✅ Ready to start development!
```

---

## 基本的な使い方

### パターン1: 手動Git操作（推奨：学習用）

各ステップでGitコマンドを明示的に実行します。

```bash
# 1. 機能を初期化
/kiro/init feature "Navigation display"

# 2. 要件定義用のブランチを作成
/kiro/git branch navigation-display requirements
# → ブランチ: feature/navigation-display-requirements

# 3. 要件を生成
/kiro/spec navigation-display requirements

# 4. コミット
/kiro/git commit navigation-display requirements
# → コミット: docs(navigation-display): add requirements

# 5. PR作成
/kiro/git pr navigation-display requirements --draft
# → PR #123 created

# === PR承認・マージ待ち ===

# 6. 次のフェーズ（設計）
/kiro/git branch navigation-display design
# → 自動的にmainから最新を取得してブランチ作成

/kiro/spec navigation-display design -y
/kiro/git commit navigation-display design
/kiro/git pr navigation-display design --draft

# === PR承認・マージ待ち ===

# 7. タスクフェーズ
/kiro/git branch navigation-display tasks
/kiro/spec navigation-display tasks -y
/kiro/git commit navigation-display tasks
/kiro/git pr navigation-display tasks --draft

# === PR承認・マージ待ち ===

# 8. 実装フェーズ
/kiro/git branch navigation-display implementation
/kiro/spec navigation-display impl

# 実装中は適宜コミット
git add ...
git commit -m "feat(navigation-display): implement map view"

# 完了後、PR作成
/kiro/git pr navigation-display implementation --ready
```

---

### パターン2: 自動Git操作（推奨：実務用）

`--git` フラグで全Git操作を自動化します。

```bash
# 1. 機能初期化（ブランチ作成込み）
/kiro/init feature "Navigation display" --git

# 2. 要件定義（ブランチ・コミット・PR自動）
/kiro/spec navigation-display requirements --git

# 内部で自動実行:
# - /kiro/git branch navigation-display requirements
# - /kiro/spec navigation-display requirements
# - /kiro/git commit navigation-display requirements
# - /kiro/git pr navigation-display requirements --draft

# === PR #123 created (Draft) ===
# → レビュー・承認・マージ

# 3. 設計（同様に自動）
/kiro/spec navigation-display design --git

# === PR #124 created (Draft) ===
# → レビュー・承認・マージ

# 4. タスク（同様に自動）
/kiro/spec navigation-display tasks --git

# === PR #125 created (Draft) ===
# → レビュー・承認・マージ

# 5. 実装
/kiro/spec navigation-display impl --git

# 実装中は手動コミット
# 完了後:
/kiro/git pr navigation-display implementation --ready

# === PR #126 created (Ready for review) ===
```

---

## ブランチ戦略

### 自動生成されるブランチ

```
main (protected)
├── feature/navigation-display-requirements
│   └── PR #123 → Merge
├── feature/navigation-display-design
│   └── PR #124 → Merge
├── feature/navigation-display-tasks
│   └── PR #125 → Merge
└── feature/navigation-display-impl
    └── PR #126 → Merge
```

### ブランチ命名規則

- **要件**: `feature/{feature-name}-requirements`
- **設計**: `feature/{feature-name}-design`
- **タスク**: `feature/{feature-name}-tasks`
- **実装**: `feature/{feature-name}-impl` または `feature/{feature-name}-implementation`

---

## コミットメッセージ規約

### フォーマット

```
<type>(<scope>): <description>
```

### 自動生成される例

```bash
# 要件フェーズ
docs(navigation-display): add requirements specification

# 設計フェーズ
docs(navigation-display): add technical design

# タスクフェーズ
docs(navigation-display): add implementation tasks

# 実装フェーズ（手動）
feat(navigation-display): implement map view component
feat(navigation-display): add route overlay rendering
test(navigation-display): add map view tests
fix(navigation-display): correct route calculation
refactor(navigation-display): extract route service
```

---

## PR作成

### Draft PR（作業中）

```bash
/kiro/git pr navigation-display requirements --draft
```

### Ready for Review

```bash
/kiro/git pr navigation-display implementation --ready
```

### PRの内容

各PRには自動的に以下が含まれます：
- ✅ 意味のあるタイトル
- ✅ 詳細な説明（サマリー、チェックリスト）
- ✅ 関連ファイルのリスト
- ✅ 次のステップの提案
- ✅ 適切なラベル（オプション）

---

## よくある操作

### 現在の状態を確認

```bash
/kiro/git status
```

**出力例**:
```
# Git Status

## Repository
- Remote: origin → https://github.com/username/compass.git
- Current branch: feature/navigation-display-requirements

## Active Feature Branches
| Feature | Phase | Branch | PR | Status |
|---------|-------|--------|-----|--------|
| navigation-display | requirements | feature/...-requirements | #123 | Open |

## Pending Actions
⚠️ Uncommitted changes: 2 files
   Action: /kiro/git commit navigation-display requirements
```

### リモートと同期

```bash
/kiro/git sync
```

mainブランチの最新を取得してマージします。

### ブランチ切り替え

```bash
# mainに戻る
git checkout main

# 特定のフェーズに戻る
git checkout feature/navigation-display-design
```

---

## トラブルシューティング

### Q: GitHub CLIがない

```bash
# macOS
brew install gh

# Linux/Windows
https://github.com/cli/cli#installation
```

インストール後：
```bash
gh auth login
```

### Q: PRが作成できない

**原因**: GitHub CLI未認証

**解決**:
```bash
gh auth login
gh auth status
```

### Q: Uncommitted changes でブランチ切り替えできない

**解決1**: コミットする
```bash
/kiro/git commit [feature-name] [phase]
```

**解決2**: Stashする
```bash
git stash
# ブランチ切り替え
git stash pop
```

### Q: Conflictが発生した

**仕様ファイルのコンフリクト**:
```bash
# 通常は自分の変更を優先
git checkout --ours .kiro/specs/[feature]/[file].md

# または手動でマージ
# エディタでファイルを開いてマージ
```

---

## チーム開発

### レビュアーの指定

```bash
/kiro/git pr navigation-display requirements --reviewer username
```

### Assigneeの指定

```bash
/kiro/git pr navigation-display requirements --assignee username
```

### PRのステータス確認

```bash
gh pr list
gh pr view 123
```

---

## GitHub Actions統合（オプション）

仕様ファイルの自動検証を追加できます。

**`.github/workflows/validate-specs.yml`**:
```yaml
name: Validate Specs

on:
  pull_request:
    paths:
      - '.kiro/specs/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Validate spec.json
        run: |
          for spec in .kiro/specs/*/spec.json; do
            jq empty "$spec" || exit 1
          done
```

---

## ベストプラクティス

### ✅ DO

1. **フェーズごとにPRを分ける**
   - 要件、設計、タスク、実装で別々のPR
   - レビューしやすい

2. **Draft PRを活用**
   - 作業中はDraft
   - 完了したらReady for review

3. **意味のあるコミットメッセージ**
   - Conventional Commits準拠
   - 将来見返してわかる内容

4. **定期的に同期**
   - `/kiro/git sync` でmainと同期
   - コンフリクトを早期に発見

5. **小さくリリース**
   - 機能を小分けに
   - 早くフィードバックを得る

### ❌ DON'T

1. **mainに直接push**
   - 必ずPR経由

2. **巨大なPR**
   - レビューが困難
   - 機能を分割

3. **不明瞭なコミットメッセージ**
   - "fix", "update"だけはNG
   - 何を変更したか明確に

4. **テストなしで実装PR**
   - 実装にはテストを含める
   - カバレッジを維持

---

## まとめ

### 最小限の3コマンド

```bash
# 1. 初期化（1回だけ）
/kiro/git init https://github.com/username/compass.git

# 2. 機能開発（繰り返し）
/kiro/spec [feature-name] [phase] --git

# 3. ステータス確認（いつでも）
/kiro/git status
```

### フルワークフロー（自動化版）

```bash
# セットアップ
/kiro/git init https://github.com/username/compass.git
gh auth login

# 開発
/kiro/init feature "Navigation display" --git
/kiro/spec navigation-display requirements --git  # → PR #123
/kiro/spec navigation-display design --git         # → PR #124
/kiro/spec navigation-display tasks --git          # → PR #125
/kiro/spec navigation-display impl --git           # → ブランチ作成
# 実装...
/kiro/git pr navigation-display implementation --ready  # → PR #126
```

---

## 次のステップ

1. ✅ Git初期化: `/kiro/git init [url]`
2. ✅ GitHub CLI認証: `gh auth login`
3. ✅ 最初の機能開発: `/kiro/init feature "..." --git`
4. ✅ レビュー・マージ・反復

詳細: `.kiro/steering/git-workflow.md`

Git統合で、仕様駆動開発がさらにスムーズに！🚀

