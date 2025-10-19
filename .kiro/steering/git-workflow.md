<!-- Inclusion Mode: Always -->

# Git/GitHub Workflow for Kiro SDD

仕様駆動開発とGit/GitHubを統合したバージョン管理ワークフロー

## Overview

Kiro SDDの各フェーズ（要件定義、設計、タスク、実装）をGitブランチ戦略とPRワークフローに完全統合します。

**推奨**: SSH認証を使用（セキュリティと利便性のため）

詳細: `.kiro/SSH_SETUP.md`

---

## Branch Strategy

### Branch Types

```
main (protected)
├── feature/{feature-name}-requirements
├── feature/{feature-name}-design
├── feature/{feature-name}-tasks
└── feature/{feature-name}-impl
```

#### Main Branch
- **Purpose**: 安定版のコードベース
- **Protection**: 
  - Direct push禁止
  - PR経由のみマージ可能
  - レビュー必須（推奨）
- **Contains**: 承認済みの仕様とテスト済みの実装

#### Feature Branches
- **Naming**: `feature/{feature-name}-{phase}`
- **Lifespan**: フェーズ完了まで
- **Merge**: PR経由でmainにマージ

---

## Workflow by Phase

### Phase 1: Requirements (要件定義)

```bash
# 1. ブランチ作成
/kiro/git branch navigation-display requirements

# 内部実行:
# - git checkout main
# - git pull origin main
# - git checkout -b feature/navigation-display-requirements

# 2. 要件生成
/kiro/spec navigation-display requirements

# 3. コミット
/kiro/git commit navigation-display requirements

# 内部実行:
# - git add .kiro/specs/navigation-display/requirements.md
# - git add .kiro/specs/navigation-display/spec.json
# - git commit -m "docs(navigation-display): add requirements specification"
# - git push origin feature/navigation-display-requirements

# 4. PR作成
/kiro/git pr navigation-display requirements --draft

# 内部実行:
# - gh pr create --title "Requirements: Navigation Display" \
#                --body "..." \
#                --base main \
#                --head feature/navigation-display-requirements \
#                --draft
```

**生成されるPR**:
```markdown
## Requirements Specification: Navigation Display

### Summary
- Platform targets: Flutter, watchOS
- Total requirements: 8
- EARS compliance: ✅

### Checklist
- [ ] All requirements use EARS format
- [ ] Platform targets are clear
- [ ] Requirements are testable

### Files
- .kiro/specs/navigation-display/requirements.md
- .kiro/specs/navigation-display/spec.json
```

**Review → Approve → Merge**

---

### Phase 2: Design (技術設計)

```bash
# 前提: Requirements PRがマージ済み

# 1. ブランチ作成（mainから）
/kiro/git branch navigation-display design

# 2. 設計生成
/kiro/spec navigation-display design -y  # 要件を自動承認

# 3. Figma参照（オプション）
/kiro/figma tokens
/kiro/spec navigation-display design --with-figma

# 4. コミット
/kiro/git commit navigation-display design

# 5. PR作成
/kiro/git pr navigation-display design --draft
```

**生成されるPR**:
```markdown
## Technical Design: Navigation Display

### Summary
- Requirements: ✅ Approved (PR #123)
- Components: 5
- Architecture diagrams: 2
- Platform coverage: Flutter, watchOS

### Key Design Decisions
1. Use CustomPaint for route overlay (Flutter)
2. WatchConnectivity for data sync
3. LRU cache for map tiles

### Files
- .kiro/specs/navigation-display/design.md
- .kiro/specs/navigation-display/spec.json
```

**Review → Approve → Merge**

---

### Phase 3: Tasks (タスク分解)

```bash
# 前提: Design PRがマージ済み

# 1. ブランチ作成
/kiro/git branch navigation-display tasks

# 2. タスク生成
/kiro/spec navigation-display tasks -y  # 設計を自動承認

# 3. コミット
/kiro/git commit navigation-display tasks

# 4. PR作成
/kiro/git pr navigation-display tasks --draft
```

**生成されるPR**:
```markdown
## Implementation Tasks: Navigation Display

### Summary
- Total tasks: 15
- Flutter: 6 tasks
- watchOS: 5 tasks
- Sync: 2 tasks
- Test: 2 tasks

### Task Breakdown
- Setup: 1 task
- Flutter Implementation: 6 tasks
- watchOS Implementation: 5 tasks
- Data Sync: 2 tasks
- Testing: 1 task

### Files
- .kiro/specs/navigation-display/tasks.md
- .kiro/specs/navigation-display/spec.json
```

**Review → Approve → Merge**

---

### Phase 4: Implementation (実装)

```bash
# 前提: Tasks PRがマージ済み

# 1. ブランチ作成
/kiro/git branch navigation-display implementation

# 2. タスクごとに実装
/kiro/spec navigation-display impl 2.1

# 実装後、コミット
git add flutter_app/lib/features/navigation/...
git commit -m "feat(navigation-display): implement map view component"

# 次のタスク
/kiro/spec navigation-display impl 2.2

git add flutter_app/lib/features/navigation/...
git commit -m "feat(navigation-display): implement route overlay"

# すべてのタスク完了後、PR作成
/kiro/git pr navigation-display implementation --ready
```

**生成されるPR**:
```markdown
## Implementation: Navigation Display

### Summary
- Tasks completed: 15/15 ✅
- Platform: Flutter, watchOS
- Tests: All passing ✅

### Implementation Details
Implemented real-time navigation display with cross-platform synchronization.

### Changes
- Added MapView component (Flutter)
- Added RouteOverlay rendering (Flutter)
- Added NavigationView (watchOS)
- Implemented WatchConnectivity sync layer
- Added unit and integration tests

### Testing
- [x] Unit tests pass (85% coverage)
- [x] Widget tests pass
- [x] Integration tests pass
- [x] Manual testing on iPhone and Apple Watch

### Files
- flutter_app/lib/features/navigation/...
- watch_app/WatchApp/Views/...
- .kiro/specs/navigation-display/tasks.md (updated checkboxes)
```

**Review → Approve → Merge**

---

## Integrated Workflow (Auto-Git Mode)

### 自動化されたワークフロー

```bash
# すべてのGit操作を自動化
/kiro/spec navigation-display requirements --git

# 内部で実行される:
# 1. /kiro/git branch navigation-display requirements
# 2. /kiro/spec navigation-display requirements
# 3. /kiro/git commit navigation-display requirements
# 4. /kiro/git pr navigation-display requirements --draft
```

**フルワークフロー例**:
```bash
# 機能初期化（Gitブランチ作成込み）
/kiro/init feature "Navigation display" --git

# 要件定義（自動コミット・PR作成）
/kiro/spec navigation-display requirements --git
# → PR #123 created (Draft)

# PR承認待ち → Merge

# 設計（新ブランチ作成・自動コミット・PR作成）
/kiro/spec navigation-display design --git
# → PR #124 created (Draft)

# PR承認待ち → Merge

# タスク（新ブランチ作成・自動コミット・PR作成）
/kiro/spec navigation-display tasks --git
# → PR #125 created (Draft)

# PR承認待ち → Merge

# 実装（新ブランチ作成）
/kiro/spec navigation-display impl --git
# 実装中は手動コミット
# 完了後、PR作成
/kiro/git pr navigation-display implementation --ready
# → PR #126 created (Ready for review)
```

---

## Commit Message Convention

### Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### Types

- `feat`: 新機能
- `fix`: バグ修正
- `docs`: ドキュメント（仕様書含む）
- `style`: コードスタイル（機能に影響しない）
- `refactor`: リファクタリング
- `test`: テスト追加・修正
- `chore`: ビルド・設定変更

### Scope

- Feature name: `navigation-display`, `search`, etc.
- General: `kiro`, `project`, `ci`, etc.

### Examples

```bash
# Requirements phase
docs(navigation-display): add requirements specification

# Design phase
docs(navigation-display): add technical design

# Implementation
feat(navigation-display): implement map view component
feat(navigation-display): add route overlay rendering
test(navigation-display): add map view tests

# Bugfix
fix(navigation-display): correct route calculation logic

# Refactor
refactor(navigation-display): extract route service
```

---

## PR Templates

### Requirements PR Template

```markdown
## Requirements Specification: {Feature Name}

### Summary
- Platform targets: {platforms}
- Total requirements: {count}
- EARS compliance: {yes/no}

### Requirements Overview
{Brief description}

### Checklist
- [ ] All requirements use EARS format
- [ ] Platform targets are clear
- [ ] Requirements are testable
- [ ] No ambiguous wording
- [ ] Sync requirements defined (if cross-platform)

### Related Documents
- Requirements: `.kiro/specs/{feature}/requirements.md`
- Spec metadata: `.kiro/specs/{feature}/spec.json`

### Next Steps
After approval:
- Merge this PR
- Create design: `/kiro/git branch {feature} design`
```

### Design PR Template

```markdown
## Technical Design: {Feature Name}

### Summary
- Requirements: ✅ Approved (PR #{pr-number})
- Components: {count}
- Architecture diagrams: {count}
- Platform coverage: {platforms}

### Design Overview
{Brief description}

### Key Design Decisions
1. {Decision 1}
2. {Decision 2}
3. {Decision 3}

### Checklist
- [ ] Architecture diagram included
- [ ] All requirements covered
- [ ] Component interfaces defined
- [ ] Data models documented
- [ ] Error handling strategy defined
- [ ] Testing strategy outlined

### Related Documents
- Design: `.kiro/specs/{feature}/design.md`
- Requirements: `.kiro/specs/{feature}/requirements.md`

### Next Steps
After approval:
- Merge this PR
- Create tasks: `/kiro/git branch {feature} tasks`
```

### Tasks PR Template

```markdown
## Implementation Tasks: {Feature Name}

### Summary
- Requirements: ✅ Approved
- Design: ✅ Approved
- Total tasks: {count}
- Platform breakdown:
  - Flutter: {count}
  - watchOS: {count}
  - Sync: {count}
  - Test: {count}

### Task Overview
{Brief description}

### Checklist
- [ ] All requirements mapped to tasks
- [ ] Tasks sized appropriately (1-3h each)
- [ ] Platform targets clear
- [ ] Dependencies identified
- [ ] Test tasks included

### Related Documents
- Tasks: `.kiro/specs/{feature}/tasks.md`
- Design: `.kiro/specs/{feature}/design.md`

### Next Steps
After approval:
- Merge this PR
- Start implementation: `/kiro/git branch {feature} implementation`
```

### Implementation PR Template

```markdown
## Implementation: {Feature Name}

### Summary
- Tasks completed: {completed}/{total}
- Platform: {platforms}
- Tests: {status}

### Implementation Details
{Description of what was implemented}

### Changes
- {Change 1}
- {Change 2}
- {Change 3}

### Testing
- [ ] Unit tests pass ({coverage}% coverage)
- [ ] Widget/UI tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed
- [ ] Performance acceptable

### Screenshots/Videos
{If applicable}

### Breaking Changes
{If any}

### Related Documents
- Spec: `.kiro/specs/{feature}/`
- Tasks: `.kiro/specs/{feature}/tasks.md` (checkboxes updated)

### Next Steps
After merge:
- Feature is complete ✅
- Deploy to staging/production
```

---

## Merge Strategy

### Requirements/Design/Tasks PRs

**Strategy**: Squash and Merge
- Single commit per phase in main branch
- Clean history
- Commit message: PR title

**Example**:
```
docs(navigation-display): add requirements specification (#123)
docs(navigation-display): add technical design (#124)
docs(navigation-display): add implementation tasks (#125)
```

### Implementation PRs

**Strategy**: Merge Commit (推奨) or Rebase and Merge
- Preserve individual commits
- Full implementation history
- Easy to revert specific changes

**Example**:
```
Merge pull request #126 from user/feature/navigation-display-impl

feat(navigation-display): implement navigation display
  - feat(navigation-display): implement map view
  - feat(navigation-display): add route overlay
  - feat(navigation-display): implement watchOS view
  - feat(navigation-display): add data sync layer
  - test(navigation-display): add tests
```

---

## Branch Protection Rules

### Main Branch

**Recommended Settings**:
```yaml
# .github/branch-protection.yml
main:
  required_pull_request_reviews:
    required_approving_review_count: 1
  required_status_checks:
    strict: true
    contexts:
      - "test"
      - "lint"
  enforce_admins: false
  restrictions: null
```

---

## GitHub Actions (Optional)

### Spec Validation

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
            if ! jq empty "$spec"; then
              echo "Invalid JSON: $spec"
              exit 1
            fi
          done
      
      - name: Check EARS format
        run: |
          # Add validation for EARS format in requirements.md
          echo "Validating EARS format..."
```

---

## Conflict Resolution

### Spec File Conflicts

仕様ファイルでコンフリクトが発生した場合：

#### 1. requirements.md
```bash
# 通常は最新の要件を採用
git checkout --theirs .kiro/specs/{feature}/requirements.md

# または手動でマージ
# - 両方の要件を統合
# - 重複を削除
```

#### 2. design.md
```bash
# 設計の場合も最新を採用（通常は自分の変更）
git checkout --ours .kiro/specs/{feature}/design.md

# または手動でマージ
# - 両方の設計を統合
# - 矛盾を解消
```

#### 3. spec.json
```bash
# JSONの場合は注意深くマージ
# - 両方の変更を確認
# - フィールドごとに判断
```

---

## Best Practices

### ✅ DO

1. **フェーズごとにブランチを分ける**
   - Requirements → Design → Tasks → Implementation
   - 各フェーズは独立したPR

2. **Draft PRを活用**
   - 作業中はDraft
   - 完了したらReady for review

3. **意味のあるコミットメッセージ**
   - Conventional Commits準拠
   - scopeに機能名を含める

4. **レビューを待つ**
   - 各フェーズでレビュー
   - フィードバックを反映

5. **定期的にsync**
   - `/kiro/git sync`
   - mainからの変更を取り込む

### ❌ DON'T

1. **mainに直接push**
   - 必ずPR経由

2. **フェーズをスキップ**
   - Requirements → Design → Tasks → Implementation
   - この順序を守る

3. **巨大なPR**
   - 機能を小さく分割
   - 1PR = 1フェーズまたは数タスク

4. **コミットメッセージを適当に**
   - 規約に従う
   - 後から見て分かるように

5. **コンフリクトを放置**
   - すぐに解決
   - 不明な場合は相談

---

## Troubleshooting

### Q: PRが多すぎる（4 PRs per feature）

**A**: 必要に応じて統合可能
```bash
# Requirements + Design を1つのPRに
/kiro/git branch navigation-display spec
/kiro/spec navigation-display requirements
/kiro/spec navigation-display design -y
/kiro/git commit navigation-display spec
/kiro/git pr navigation-display spec
```

### Q: レビュー待ちで次に進めない

**A**: 並行開発可能
```bash
# Feature A のレビュー待ち中、Feature B を開始
/kiro/init feature "Search functionality" --git
/kiro/spec search requirements --git
```

### Q: Gitコマンドがエラーになる

**A**: Git CLIとGitHub CLIの確認
```bash
# Git確認
git --version

# GitHub CLI確認
gh --version

# GitHub CLI認証
gh auth login
```

---

## Summary

Kiro SDD v2のGit/GitHub統合により：
- ✅ 各フェーズが明確なブランチとPRに対応
- ✅ 自動化されたコミットとPR作成
- ✅ レビュープロセスの統合
- ✅ クリーンなGit履歴
- ✅ 仕様とコードのバージョン管理の一元化

次は実際にプロジェクトで試してみましょう！

