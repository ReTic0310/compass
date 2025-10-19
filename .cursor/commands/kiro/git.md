<meta>
description: Git/GitHub integration for spec-driven development workflow
argument-hint: [action] [options]
</meta>

# Git/GitHub Integration

Kiro SDDのワークフローに完全統合されたGit/GitHubバージョン管理

## Usage

```bash
# プロジェクト初期化（リモート設定）
/kiro/git init [remote-url]

# 機能ブランチ作成
/kiro/git branch [feature-name] [phase]

# 自動コミット
/kiro/git commit [feature-name] [phase]

# PR作成
/kiro/git pr [feature-name] [phase]

# ステータス確認
/kiro/git status

# 同期（fetch + pull）
/kiro/git sync

# ヘルプ
/kiro/git --help
```

---

## Command: init

### Task: Initialize Git Repository with Remote

Gitリポジトリを初期化し、リモートリポジトリを設定します。

**Usage**:
```bash
# SSH認証（推奨）
/kiro/git init git@github.com:username/compass.git

# または repository名から自動生成
/kiro/git init username/compass

# HTTPS認証（オプション）
/kiro/git init https://github.com/username/compass.git --https
```

**Authentication Method Detection**:
- `git@github.com:` → SSH認証
- `https://github.com/` → HTTPS認証
- `username/repo` → SSH認証（デフォルト）

**Process**:

1. **Check Git Installation**
   ```bash
   git --version
   ```

2. **Check SSH Key** (if SSH method)
   ```bash
   ssh -T git@github.com
   
   # If SSH key not set up, show setup instructions
   ```

3. **Initialize Repository** (if not initialized)
   ```bash
   git init
   ```

4. **Configure Git** (if needed)
   ```bash
   git config user.name
   git config user.email
   # If not set, prompt user to configure
   ```

5. **Add Remote**
   ```bash
   # SSH
   git remote add origin git@github.com:username/compass.git
   
   # HTTPS
   git remote add origin https://github.com/username/compass.git
   ```

5. **Create .gitignore** (if not exists)
   ```
   # Flutter
   .dart_tool/
   .flutter-plugins
   .flutter-plugins-dependencies
   .packages
   build/
   
   # iOS
   *.pbxuser
   *.mode1v3
   *.mode2v3
   *.perspectivev3
   xcuserdata/
   *.xcworkspace/
   Pods/
   
   # Android
   *.iml
   .gradle
   local.properties
   
   # Figma
   .env
   .figma-config.json
   
   # Kiro
   .kiro/figma-snapshots/
   
   # IDE
   .vscode/
   .idea/
   *.swp
   *.swo
   *~
   ```

6. **Initial Commit**
   ```bash
   git add .gitignore README.md AGENTS.md .kiro/
   git commit -m "chore: initialize project with Kiro SDD v2"
   ```

7. **Create Main Branch** (if needed)
   ```bash
   git branch -M main
   ```

8. **Set Upstream**
   ```bash
   git push -u origin main
   ```

**Generate**: `.kiro/git-config.json`
```json
{
  "remote": {
    "origin": "https://github.com/username/compass.git"
  },
  "default_branch": "main",
  "branch_naming": {
    "feature": "feature/{feature-name}-{phase}",
    "bugfix": "bugfix/{feature-name}",
    "hotfix": "hotfix/{description}"
  },
  "auto_commit": true,
  "auto_push": false,
  "pr_template": true,
  "commit_message_template": {
    "requirements": "docs: add requirements for {feature-name}",
    "design": "docs: add technical design for {feature-name}",
    "tasks": "docs: add implementation tasks for {feature-name}",
    "implementation": "feat: implement {task-description}",
    "test": "test: add tests for {feature-name}"
  }
}
```

**Output**:
```
✅ Git repository initialized

Remote: origin → https://github.com/username/compass.git
Branch: main
Initial commit: ✅

Next steps:
1. Configure your GitHub authentication (gh auth login)
2. Start developing: /kiro/git branch [feature-name] requirements
```

---

## Command: branch

### Task: Create Feature Branch

機能開発用のブランチを作成します。

**Usage**:
```bash
/kiro/git branch [feature-name] [phase]

# Examples:
/kiro/git branch navigation-display requirements
/kiro/git branch navigation-display design
/kiro/git branch navigation-display tasks
/kiro/git branch navigation-display implementation
```

**Branch Naming Convention**:
```
feature/{feature-name}-{phase}

Examples:
- feature/navigation-display-requirements
- feature/navigation-display-design
- feature/navigation-display-tasks
- feature/navigation-display-impl
```

**Process**:

1. **Validate Current State**
   ```bash
   # Check if working tree is clean
   git status --porcelain
   
   # If dirty, prompt user:
   # "Working tree has uncommitted changes. Commit or stash? [c/s/a]"
   # c = commit, s = stash, a = abort
   ```

2. **Update Main Branch**
   ```bash
   git checkout main
   git pull origin main
   ```

3. **Create Feature Branch**
   ```bash
   git checkout -b feature/{feature-name}-{phase}
   ```

4. **Update spec.json**
   ```json
   {
     "git": {
       "current_branch": "feature/navigation-display-requirements",
       "base_branch": "main",
       "phase_branches": {
         "requirements": "feature/navigation-display-requirements"
       }
     }
   }
   ```

**Output**:
```
✅ Branch created: feature/navigation-display-requirements

Current branch: feature/navigation-display-requirements
Base branch: main

You can now work on requirements:
/kiro/spec navigation-display requirements
```

---

## Command: commit

### Task: Auto-commit Spec Phase

フェーズ完了時に自動でコミットします。

**Usage**:
```bash
/kiro/git commit [feature-name] [phase]

# Examples:
/kiro/git commit navigation-display requirements
/kiro/git commit navigation-display design
/kiro/git commit navigation-display tasks
```

**Process**:

1. **Validate Phase Completion**
   - Check `spec.json` for phase status
   - Ensure phase is marked as "generated"

2. **Stage Files**
   ```bash
   # For requirements phase:
   git add .kiro/specs/{feature-name}/requirements.md
   git add .kiro/specs/{feature-name}/spec.json
   
   # For design phase:
   git add .kiro/specs/{feature-name}/design.md
   git add .kiro/specs/{feature-name}/spec.json
   
   # For tasks phase:
   git add .kiro/specs/{feature-name}/tasks.md
   git add .kiro/specs/{feature-name}/spec.json
   
   # For implementation:
   git add [implementation files]
   git add .kiro/specs/{feature-name}/tasks.md  # Updated checkboxes
   ```

3. **Generate Commit Message**
   ```
   Template: {type}: {scope} - {description}
   
   Examples:
   - docs(navigation-display): add requirements specification
   - docs(navigation-display): add technical design
   - docs(navigation-display): add implementation tasks
   - feat(navigation-display): implement map view component
   ```

4. **Commit**
   ```bash
   git commit -m "{commit-message}"
   ```

5. **Optional: Push**
   ```bash
   # If auto_push is enabled in git-config.json
   git push origin {current-branch}
   ```

**Output**:
```
✅ Committed: docs(navigation-display): add requirements specification

Branch: feature/navigation-display-requirements
Files changed: 2
- .kiro/specs/navigation-display/requirements.md
- .kiro/specs/navigation-display/spec.json

Next steps:
1. Review the changes: git log -1 --stat
2. Create PR: /kiro/git pr navigation-display requirements
3. Or continue: /kiro/spec navigation-display design
```

---

## Command: pr

### Task: Create Pull Request

GitHub PRを自動作成します（GitHub CLI使用）。

**Prerequisites**:
- GitHub CLI (`gh`) installed
- Authenticated: `gh auth login`

**Usage**:
```bash
/kiro/git pr [feature-name] [phase]

# Examples:
/kiro/git pr navigation-display requirements
/kiro/git pr navigation-display design --draft
/kiro/git pr navigation-display implementation --ready
```

**Options**:
- `--draft`: Draft PRとして作成
- `--ready`: Ready for reviewとして作成（デフォルト）
- `--assignee [username]`: Assigneeを指定
- `--reviewer [username]`: Reviewerを指定

**Process**:

1. **Check GitHub CLI**
   ```bash
   gh --version
   # If not installed, show installation instructions
   ```

2. **Push Branch** (if not pushed)
   ```bash
   git push origin {current-branch}
   ```

3. **Generate PR Title**
   ```
   Template: {Phase}: {Feature Name}
   
   Examples:
   - Requirements: Navigation Display
   - Design: Navigation Display
   - Tasks: Navigation Display
   - Implementation: Navigation Display
   ```

4. **Generate PR Body**
   
   **For Requirements Phase**:
   ```markdown
   ## Requirements Specification: {Feature Name}
   
   This PR adds the requirements specification for {feature-name}.
   
   ### Summary
   - Platform targets: {platforms}
   - Total requirements: {count}
   - EARS compliance: ✅
   
   ### Requirements Overview
   {Brief summary from requirements.md}
   
   ### Checklist
   - [ ] All requirements use EARS format
   - [ ] Platform targets are clear
   - [ ] Requirements are testable
   - [ ] No ambiguous wording
   
   ### Next Steps
   After approval:
   - Create design: `/kiro/spec {feature-name} design -y`
   - Or: `/kiro/git branch {feature-name} design`
   
   ---
   
   Related files:
   - `.kiro/specs/{feature-name}/requirements.md`
   - `.kiro/specs/{feature-name}/spec.json`
   ```

   **For Design Phase**:
   ```markdown
   ## Technical Design: {Feature Name}
   
   This PR adds the technical design for {feature-name}.
   
   ### Summary
   - Requirements: ✅ Approved
   - Components: {count}
   - Architecture diagrams: {count}
   - Platform coverage: {platforms}
   
   ### Design Overview
   {Brief summary from design.md}
   
   ### Key Design Decisions
   {List 2-3 major decisions}
   
   ### Checklist
   - [ ] Architecture diagram included
   - [ ] All requirements covered
   - [ ] Component interfaces defined
   - [ ] Testing strategy outlined
   
   ### Next Steps
   After approval:
   - Create tasks: `/kiro/spec {feature-name} tasks -y`
   - Or: `/kiro/git branch {feature-name} tasks`
   
   ---
   
   Related files:
   - `.kiro/specs/{feature-name}/design.md`
   - `.kiro/specs/{feature-name}/spec.json`
   ```

   **For Tasks Phase**:
   ```markdown
   ## Implementation Tasks: {Feature Name}
   
   This PR adds implementation tasks for {feature-name}.
   
   ### Summary
   - Requirements: ✅ Approved
   - Design: ✅ Approved
   - Total tasks: {count}
   - Platform breakdown:
     - Flutter: {count}
     - watchOS: {count}
     - Sync: {count}
   
   ### Task Overview
   {Brief summary from tasks.md}
   
   ### Checklist
   - [ ] All requirements mapped to tasks
   - [ ] Tasks are sized appropriately (1-3h each)
   - [ ] Platform targets are clear
   - [ ] Dependencies identified
   
   ### Next Steps
   After approval:
   - Start implementation: `/kiro/spec {feature-name} impl`
   - Or: `/kiro/git branch {feature-name} implementation`
   
   ---
   
   Related files:
   - `.kiro/specs/{feature-name}/tasks.md`
   - `.kiro/specs/{feature-name}/spec.json`
   ```

   **For Implementation Phase**:
   ```markdown
   ## Implementation: {Feature Name}
   
   This PR implements {feature-name}.
   
   ### Summary
   - Tasks completed: {completed}/{total}
   - Platform: {platform}
   - Tests: {test-status}
   
   ### Implementation Details
   {Brief description of what was implemented}
   
   ### Changes
   - {List of main changes}
   
   ### Testing
   - [ ] Unit tests pass
   - [ ] Integration tests pass
   - [ ] Manual testing completed
   
   ### Screenshots (if applicable)
   {Screenshots or GIFs}
   
   ---
   
   Related spec:
   - `.kiro/specs/{feature-name}/`
   ```

5. **Create PR**
   ```bash
   gh pr create \
     --title "{pr-title}" \
     --body "{pr-body}" \
     --base main \
     --head {current-branch} \
     --draft  # if --draft flag
   ```

6. **Update spec.json**
   ```json
   {
     "git": {
       "pr": {
         "requirements": {
           "number": 123,
           "url": "https://github.com/username/compass/pull/123",
           "status": "open"
         }
       }
     }
   }
   ```

**Output**:
```
✅ Pull Request created

PR #123: Requirements: Navigation Display
URL: https://github.com/username/compass/pull/123
Status: Open (Draft)

Reviewers can view the requirements at:
https://github.com/username/compass/pull/123/files

After approval:
1. Merge the PR
2. Continue to design phase:
   /kiro/git branch navigation-display design
   /kiro/spec navigation-display design -y
```

---

## Command: status

### Task: Git Status Overview

Git状態とKiro SDDの統合状況を表示します。

**Usage**:
```bash
/kiro/git status
/kiro/git status --verbose
```

**Output**:
```markdown
# Git Status

## Repository
- Remote: origin → https://github.com/username/compass.git
- Default branch: main
- Current branch: feature/navigation-display-requirements

## Working Tree
- Modified: 2 files
- Staged: 0 files
- Untracked: 0 files

## Branches

### Active Feature Branches
| Feature | Phase | Branch | PR Status | Last Commit |
|---------|-------|--------|-----------|-------------|
| navigation-display | requirements | feature/navigation-display-requirements | #123 Open | 2h ago |
| search | design | feature/search-design | #124 Open | 1d ago |

### Merged Features
| Feature | Merged | PR | Merged By |
|---------|--------|-----|-----------|
| home-screen | 3d ago | #120 | username |

## Pending Actions

### ⚠️ Uncommitted Changes
- `.kiro/specs/navigation-display/requirements.md` (modified)
- `.kiro/specs/navigation-display/spec.json` (modified)

**Action**: `/kiro/git commit navigation-display requirements`

### 📋 Open PRs Awaiting Review
- PR #123: Requirements: Navigation Display (2 hours old)
- PR #124: Design: Search (1 day old)

### 🔄 Branches Behind Main
- `feature/search-design` is 3 commits behind main
  **Action**: `/kiro/git sync`

## Recent Activity
- 2h ago: Created branch `feature/navigation-display-requirements`
- 1d ago: Merged PR #122: Implementation: Home Screen
- 2d ago: Created PR #124: Design: Search
```

---

## Command: sync

### Task: Sync with Remote

リモートと同期し、競合を検出します。

**Usage**:
```bash
/kiro/git sync
/kiro/git sync --rebase
```

**Process**:

1. **Fetch from Remote**
   ```bash
   git fetch origin
   ```

2. **Check for Conflicts**
   ```bash
   git diff origin/main..HEAD
   ```

3. **Merge or Rebase**
   ```bash
   # Default: merge
   git merge origin/main
   
   # With --rebase flag
   git rebase origin/main
   ```

4. **Handle Conflicts** (if any)
   ```
   ⚠️ Conflicts detected in:
   - .kiro/specs/navigation-display/design.md
   
   Options:
   1. Resolve manually
   2. Accept local changes
   3. Accept remote changes
   4. Abort sync
   
   Choose: [1/2/3/4]
   ```

**Output**:
```
✅ Synced with origin/main

Changes pulled: 3 commits
- feat(home): implement home screen
- docs(search): add design
- chore: update dependencies

Your branch is up to date.
```

---

## Integrated Workflow

### Auto-Git Mode

既存のKiroコマンドに `--git` フラグを追加することで、Git操作を自動化できます。

#### Spec Workflow with Git

```bash
# 1. 機能初期化 + ブランチ作成
/kiro/init feature "Navigation display" --git

# 内部的に実行される:
# - /kiro/init feature "Navigation display"
# - /kiro/git branch navigation-display requirements

# 2. 要件生成 + コミット + PR
/kiro/spec navigation-display requirements --git

# 内部的に実行される:
# - /kiro/spec navigation-display requirements
# - /kiro/git commit navigation-display requirements
# - /kiro/git pr navigation-display requirements --draft

# 3. PR承認後、次のフェーズ
# (PR #123 merged)

/kiro/spec navigation-display design --git

# 内部的に実行される:
# - git checkout main
# - git pull origin main
# - /kiro/git branch navigation-display design
# - /kiro/spec navigation-display design
# - /kiro/git commit navigation-display design
# - /kiro/git pr navigation-display design --draft
```

---

## Git Hooks (Optional)

Kiro SDD用のGit hooksを自動生成できます。

**Generate Hooks**:
```bash
/kiro/git hooks install
```

**Generated Hooks**:

### pre-commit
```bash
#!/bin/bash
# .git/hooks/pre-commit

# Check if committing spec files
if git diff --cached --name-only | grep -q ".kiro/specs"; then
  echo "🔍 Validating spec files..."
  
  # Validate spec.json
  for spec in $(git diff --cached --name-only | grep "spec.json"); do
    if ! jq empty "$spec" 2>/dev/null; then
      echo "❌ Invalid JSON: $spec"
      exit 1
    fi
  done
  
  echo "✅ Spec validation passed"
fi
```

### commit-msg
```bash
#!/bin/bash
# .git/hooks/commit-msg

# Validate commit message format
commit_msg=$(cat "$1")

# Check for conventional commits format
if ! echo "$commit_msg" | grep -qE "^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+"; then
  echo "❌ Invalid commit message format"
  echo "Expected: type(scope): description"
  echo "Example: docs(navigation): add requirements"
  exit 1
fi
```

### pre-push
```bash
#!/bin/bash
# .git/hooks/pre-push

echo "🔍 Running pre-push checks..."

# Check if all tests pass (if implementation branch)
if [[ $(git branch --show-current) == *"-impl" ]]; then
  echo "Running tests..."
  # Add test commands here
fi

echo "✅ Pre-push checks passed"
```

---

## Configuration

### .kiro/git-config.json

```json
{
  "remote": {
    "origin": "https://github.com/username/compass.git"
  },
  "default_branch": "main",
  "branch_naming": {
    "feature": "feature/{feature-name}-{phase}",
    "bugfix": "bugfix/{feature-name}",
    "hotfix": "hotfix/{description}"
  },
  "auto_commit": true,
  "auto_push": false,
  "auto_pr": false,
  "pr_template": true,
  "pr_defaults": {
    "draft": true,
    "auto_merge": false,
    "delete_branch": true
  },
  "commit_message_template": {
    "requirements": "docs({feature-name}): add requirements specification",
    "design": "docs({feature-name}): add technical design",
    "tasks": "docs({feature-name}): add implementation tasks",
    "implementation": "feat({feature-name}): {task-description}",
    "test": "test({feature-name}): add tests"
  },
  "hooks": {
    "pre-commit": true,
    "commit-msg": true,
    "pre-push": true
  }
}
```

---

## Error Handling

### Not a Git Repository
```
❌ Error: Not a git repository

Initialize git:
/kiro/git init [remote-url]
```

### Remote Not Set
```
❌ Error: No remote repository configured

Add remote:
/kiro/git init https://github.com/username/compass.git
```

### Uncommitted Changes
```
⚠️ Warning: Uncommitted changes detected

Options:
1. Commit changes: /kiro/git commit [feature] [phase]
2. Stash changes: git stash
3. Abort operation

Choose: [1/2/3]
```

### GitHub CLI Not Installed
```
❌ Error: GitHub CLI not installed

Install gh:
- macOS: brew install gh
- Linux: https://github.com/cli/cli#installation
- Windows: https://github.com/cli/cli#installation

After installation:
gh auth login
```

### PR Already Exists
```
⚠️ Warning: PR already exists for this branch

PR #123: Requirements: Navigation Display
URL: https://github.com/username/compass/pull/123

Options:
1. Update PR description
2. View PR: gh pr view 123
3. Continue without creating PR
```

---

## Best Practices

### Branch Strategy

```
main (protected)
├── feature/navigation-display-requirements → PR #123 → Merge
├── feature/navigation-display-design → PR #124 → Merge
├── feature/navigation-display-tasks → PR #125 → Merge
└── feature/navigation-display-impl → PR #126 → Merge
```

### Commit Frequency

- **Requirements**: 1 commit per phase completion
- **Design**: 1 commit per phase completion
- **Tasks**: 1 commit per phase completion
- **Implementation**: Multiple commits (per task or logical unit)

### PR Review Process

1. **Draft PR**: Create when phase is in progress
2. **Ready for Review**: Mark when phase is complete
3. **Approval**: Reviewer approves PR
4. **Merge**: Auto-merge or manual merge
5. **Next Phase**: Create new branch for next phase

---

## Success Criteria

- [ ] Git repository initialized with remote
- [ ] Branches follow naming convention
- [ ] Commits follow conventional commits format
- [ ] PRs are created automatically with templates
- [ ] Spec files are tracked and versioned
- [ ] No merge conflicts in spec files
- [ ] GitHub Actions (optional) validate specs

ultrathink

