<meta>
description: Unified specification workflow - requirements, design, tasks, and implementation
argument-hint: [feature-name] [phase] [options]
</meta>

# Unified Specification Workflow

統合仕様コマンド - 要件定義から実装まで一貫したワークフローを提供します。

Tool policy: Use Cursor file tools (read_file/list_dir/glob_file_search/search_replace/write); no shell.

## Usage

```bash
# 対話的モード（推奨）- 現在のフェーズから次へ進む
/kiro/spec [feature-name]

# 特定フェーズを実行
/kiro/spec [feature-name] requirements
/kiro/spec [feature-name] design
/kiro/spec [feature-name] tasks
/kiro/spec [feature-name] impl [task-numbers]

# 自動承認フラグ
/kiro/spec [feature-name] design -y    # 要件を自動承認してデザイン生成
/kiro/spec [feature-name] tasks -y     # 要件とデザインを自動承認

# 一気通貫モード（実験的）
/kiro/spec [feature-name] --auto-all   # 全フェーズを自動実行（慎重に使用）
```

## Interactive Mode (Default)

引数がフィーチャー名のみの場合、現在のフェーズを検出して次のステップを対話的に実行：

1. **Initialized** → Requirements生成を提案
2. **Requirements Generated** → レビュープロンプト → Design生成
3. **Design Generated** → レビュープロンプト → Tasks生成
4. **Tasks Generated** → レビュープロンプト → Implementation開始
5. **In Implementation** → 進捗表示と次のタスク提案

## Phase Detection and Workflow

### Current Phase Detection

`.kiro/specs/[feature-name]/spec.json` から現在のフェーズを読み取る：

```json
{
  "phase": "initialized|requirements-generated|design-generated|tasks-generated|in-implementation|completed",
  "approvals": {
    "requirements": { "generated": bool, "approved": bool },
    "design": { "generated": bool, "approved": bool },
    "tasks": { "generated": bool, "approved": bool }
  }
}
```

### Workflow Logic

```
Phase: initialized
├─> Action: Generate requirements
└─> Output: requirements.md + update spec.json

Phase: requirements-generated
├─> Prompt: "要件をレビューしましたか？ [y/N]"
├─> If yes: Generate design.md
└─> If no: "要件を確認後、再度実行してください"

Phase: design-generated  
├─> Prompt: "設計をレビューしましたか？ [y/N]"
├─> If yes: Generate tasks.md
└─> If no: "設計を確認後、再度実行してください"

Phase: tasks-generated
├─> Prompt: "タスクをレビューしましたか？実装を開始しますか？ [y/N]"
├─> If yes: Start implementation (first pending task)
└─> If no: "タスクを確認後、再度実行してください"

Phase: in-implementation
├─> Show: Progress status
└─> Prompt: "次のタスクを実行しますか？ [task-number]"
```

---

## Phase 1: Requirements Generation

### Task: Generate Requirements Document

**Prerequisites**: 
- Feature initialized via `/kiro/init feature`
- `spec.json` and `requirements.md` template exist

**Context Loading**:
- `.kiro/steering/product.md`
- `.kiro/steering/tech.md`
- `.kiro/steering/structure.md`
- `.kiro/steering/security.md`
- Platform-specific steering (conditional)
- `.kiro/specs/[feature-name]/requirements.md` (template)
- `.kiro/specs/[feature-name]/spec.json`

**Language Policy**:
- Read `spec.json` for `language` field (default: "ja")
- Generate document in specified language
- Think in English, output in target language

**Process**:
1. Extract project description from requirements.md template
2. Generate EARS-based requirements
3. Map requirements to target platforms:
   - Flutter-specific requirements
   - Swift/watchOS-specific requirements
   - Shared/sync requirements

**EARS Format**:
```markdown
### Requirement X: [Major Area]
**Objective**: As a [role], I want [capability], so that [benefit]

**Target Platforms**: [Flutter | watchOS | Both]

#### Acceptance Criteria
1. WHEN [event] THEN [system] SHALL [response]
2. IF [condition] THEN [system] SHALL [response]
3. WHILE [state] THE [system] SHALL [behavior]
4. WHERE [context] THE [system] SHALL [behavior]
```

**Platform-Specific Considerations**:
- Flutter requirements: UI/UX, navigation, state management
- watchOS requirements: Complications, Digital Crown, battery efficiency
- Sync requirements: Data consistency, conflict resolution, offline support

**Document Structure**:
```markdown
# Requirements Document

## Introduction
[Feature overview and business value]

## Target Platforms
- [x] Flutter (Android/iOS)
- [x] SwiftUI (watchOS)
- [x] Data Synchronization

## Requirements

### Flutter Platform Requirements
[Flutter-specific requirements with EARS format]

### watchOS Platform Requirements
[watchOS-specific requirements with EARS format]

### Cross-Platform Requirements
[Data sync and shared requirements]

### Non-Functional Requirements
- Performance
- Security
- Usability
- Maintainability
```

**Metadata Update**:
```json
{
  "phase": "requirements-generated",
  "approvals": {
    "requirements": { "generated": true, "approved": false }
  },
  "updated_at": "current_timestamp"
}
```

**Validation Checks**:
- [ ] All acceptance criteria use proper EARS syntax
- [ ] Platform targets are clearly specified
- [ ] Sync requirements are defined for cross-platform features
- [ ] Requirements are testable and observable
- [ ] No ambiguous or subjective wording

**Output**:
```
✅ 要件定義を生成しました

📄 .kiro/specs/[feature-name]/requirements.md

プラットフォーム対象:
- Flutter: X件の要件
- watchOS: Y件の要件
- 同期: Z件の要件

次のステップ:
1. requirements.md をレビュー
2. 必要に応じて修正
3. `/kiro/spec [feature-name]` で設計フェーズへ進む
```

---

## Phase 2: Design Generation

### Task: Generate Technical Design Document

**Prerequisites**:
- Requirements generated
- User confirmation or `-y` flag

**Prerequisite Check**:
```
If requirements.approved == false AND no `-y` flag:
  → Error: "要件が未承認です。レビュー後、`-y`フラグ付きで実行してください"
  → Suggest: "/kiro/spec [feature-name] design -y"
```

**Context Loading**:
- All steering documents (core + platform-specific + domain-specific)
- `.kiro/specs/[feature-name]/requirements.md`
- `.kiro/specs/[feature-name]/spec.json`
- Existing `.kiro/specs/[feature-name]/design.md` (if exists for merge mode)

**File Handling**:
```
If design.md exists:
  Prompt: "[o]verwrite / [m]erge / [c]ancel"
  - Overwrite: Generate new design
  - Merge: Use existing as reference context
  - Cancel: Stop execution
```

**Discovery & Analysis Phase**:

#### A. Feature Classification
- **New Feature**: Full technology selection and architecture
- **Extension**: Integration analysis, minimal changes
- **Simple Addition**: Follow established patterns
- **Complex Integration**: Comprehensive analysis

#### B. Platform Target Analysis
From `spec.json.affected_platforms` and requirements:
- **Flutter-only**: Single platform design
- **watchOS-only**: Single platform design  
- **Both**: Dual platform + sync architecture

#### C. Existing Implementation Analysis
- Analyze Flutter codebase: `flutter_app/`
- Analyze watchOS codebase: `watch_app/`
- Identify reusable components
- Map integration points

#### D. Steering Alignment
- Verify compliance with all active steering documents
- Check platform-specific best practices
- Validate security guidelines
- Document deviations with rationale

#### E. External Dependencies (if any)
- Use WebSearch for documentation
- Use WebFetch for API references
- Verify compatibility
- Document integration requirements

**Design Document Structure**:

```markdown
# Technical Design: [Feature Name]

## Overview
[Feature purpose, users, impact]

### Goals
- Primary objectives

### Non-Goals
- Explicitly excluded

## Platform Architecture

### Flutter Implementation (if applicable)
**Architecture**: [Clean / MVVM / etc.]
**Key Components**:
- [Component list]
**Integration**: [How it fits existing Flutter app]

### watchOS Implementation (if applicable)
**Architecture**: [MVVM / etc.]
**Key Components**:
- [Component list]
**Integration**: [How it fits existing watchOS app]

### Data Synchronization (if cross-platform)
**Sync Method**: [WatchConnectivity / CloudKit / etc.]
**Data Flow**: [Bidirectional flow description]
**Conflict Resolution**: [Strategy]

## Architecture Diagram
\`\`\`mermaid
graph TB
    FlutterApp[Flutter App]
    WatchApp[watchOS App]
    SyncLayer[Sync Layer]
    
    FlutterApp -->|Data| SyncLayer
    SyncLayer -->|Data| WatchApp
    WatchApp -->|Events| SyncLayer
    SyncLayer -->|Events| FlutterApp
\`\`\`

## Technology Decisions

### Key Design Decision 1: [Title]
- **Decision**: [What was decided]
- **Context**: [Why this decision was needed]
- **Alternatives**: [Other options considered]
- **Selected Approach**: [Chosen solution]
- **Rationale**: [Why this is optimal]
- **Trade-offs**: [Pros and cons]

[Repeat for 1-3 critical decisions]

## System Flows

### [Flow Name] (Sequence Diagram)
\`\`\`mermaid
sequenceDiagram
    participant User
    participant Flutter
    participant Sync
    participant watchOS
    
    User->>Flutter: Trigger action
    Flutter->>Sync: Send data
    Sync->>watchOS: Transfer
    watchOS->>User: Update UI
\`\`\`

## Components and Interfaces

### Flutter Platform

#### [Component Name]
**Responsibility**: [Single clear statement]
**Domain Boundary**: [Which domain]
**Dependencies**:
- Inbound: [Who depends on this]
- Outbound: [What this depends on]

**Contract Definition**:
\`\`\`dart
class [ComponentName]Service {
  Future<Result<T, E>> methodName(InputType input);
}
\`\`\`

### watchOS Platform

#### [Component Name]
**Responsibility**: [Single clear statement]
**Dependencies**:
- Inbound: [Who depends on this]
- Outbound: [What this depends on]

**Contract Definition**:
\`\`\`swift
protocol [ComponentName]Service {
    func methodName(input: InputType) async throws -> OutputType
}
\`\`\`

### Sync Layer

#### Data Synchronization Service
**Responsibility**: Cross-platform data sync
**Methods**:
- Flutter → watchOS sync
- watchOS → Flutter sync
- Conflict resolution

## Data Models

### Shared Data Models
\`\`\`json
{
  "version": "1.0.0",
  "type": "ModelName",
  "data": {
    "field1": "type",
    "field2": "type"
  }
}
\`\`\`

**Flutter Implementation**:
\`\`\`dart
class ModelName {
  final String field1;
  final String field2;
  
  Map<String, dynamic> toJson();
  factory ModelName.fromJson(Map<String, dynamic> json);
}
\`\`\`

**Swift Implementation**:
\`\`\`swift
struct ModelName: Codable {
    let field1: String
    let field2: String
}
\`\`\`

## Error Handling

### Error Strategy
[Platform-specific error handling approaches]

### Flutter Error Handling
- User errors: [Strategy]
- System errors: [Strategy]
- Sync errors: [Strategy]

### watchOS Error Handling
- User errors: [Strategy]
- System errors: [Strategy]
- Sync errors: [Strategy]

## Testing Strategy

### Flutter Testing
- Unit tests: [Key test cases]
- Widget tests: [UI test cases]
- Integration tests: [Cross-component tests]

### watchOS Testing
- Unit tests: [Key test cases]
- UI tests: [SwiftUI test cases]
- Integration tests: [Cross-component tests]

### Sync Testing
- Mock connectivity layer
- Test offline scenarios
- Conflict resolution tests
- Real device tests required

## Security Considerations
[If feature handles auth, sensitive data, or permissions]

## Performance & Scalability
[If feature has specific performance requirements]
```

**Metadata Update**:
```json
{
  "phase": "design-generated",
  "approvals": {
    "requirements": { "generated": true, "approved": true },
    "design": { "generated": true, "approved": false }
  },
  "updated_at": "current_timestamp"
}
```

**Output**:
```
✅ 技術設計を生成しました

📄 .kiro/specs/[feature-name]/design.md

設計概要:
- Flutterコンポーネント: X個
- watchOSコンポーネント: Y個
- 同期レイヤー: [あり/なし]

次のステップ:
1. design.md をレビュー
2. アーキテクチャ図を確認
3. `/kiro/spec [feature-name]` でタスク生成へ進む
```

---

## Phase 3: Tasks Generation

### Task: Generate Implementation Tasks

**Prerequisites**:
- Design generated and approved
- User confirmation or `-y` flag

**Context Loading**:
- All steering documents
- `.kiro/specs/[feature-name]/requirements.md`
- `.kiro/specs/[feature-name]/design.md`
- `.kiro/specs/[feature-name]/spec.json`

**Task Generation Rules**:

1. **Natural Language Descriptions**: Describe functionality, not code structure
2. **Platform-Specific Tasks**: Separate Flutter and watchOS tasks
3. **Integration Tasks**: Explicit sync and integration tasks
4. **Sequential Major Numbering**: 1, 2, 3, 4... (NO reusing numbers)
5. **Sub-task Hierarchy**: Maximum 2 levels (e.g., 1.1, 1.2, but NO 1.1.1)
6. **Incremental Integration**: Each task builds on previous outputs
7. **Requirements Mapping**: Reference requirements in tasks

**Task Structure**:
```markdown
# Implementation Plan

## Setup Phase
- [ ] 1. プロジェクト基盤セットアップ
  - 必要な依存関係の追加
  - プロジェクト構造の整備
  - 共通ユーティリティの準備
  - _Requirements: All_

## Flutter Implementation

- [ ] 2. Flutter: [機能名] の実装
- [ ] 2.1 [サブ機能A] の開発
  - [詳細1]
  - [詳細2]
  - _Requirements: X.X, Y.Y_
  - _Platform: Flutter_

- [ ] 2.2 [サブ機能B] の開発
  - [詳細1]
  - [詳細2]
  - _Requirements: X.X_
  - _Platform: Flutter_

## watchOS Implementation

- [ ] 3. watchOS: [機能名] の実装
- [ ] 3.1 [サブ機能A] の開発
  - [詳細1]
  - [詳細2]
  - _Requirements: Z.Z_
  - _Platform: watchOS_

- [ ] 3.2 [サブ機能B] の開発
  - [詳細1]
  - [詳細2]
  - _Requirements: Z.Z_
  - _Platform: watchOS_

## Data Synchronization

- [ ] 4. データ同期の実装
- [ ] 4.1 Flutter側同期レイヤーの開発
  - Platform Channelの実装
  - データシリアライゼーション
  - _Requirements: Sync.1_
  - _Platform: Flutter_

- [ ] 4.2 watchOS側同期レイヤーの開発
  - WatchConnectivityの実装
  - データデシリアライゼーション
  - _Requirements: Sync.1_
  - _Platform: watchOS_

- [ ] 4.3 競合解決ロジックの実装
  - タイムスタンプ比較
  - マージロジック
  - _Requirements: Sync.2_
  - _Platform: Both_

## Testing Phase

- [ ] 5. テストの実装
- [ ] 5.1 Flutter ユニット・ウィジェットテスト
  - [テストケースリスト]
  - _Platform: Flutter_

- [ ] 5.2 watchOS ユニット・UIテスト
  - [テストケースリスト]
  - _Platform: watchOS_

- [ ] 5.3 統合テスト（同期機能）
  - Mock connectivity テスト
  - オフラインシナリオテスト
  - 実機テスト
  - _Platform: Both_

## Integration & Polish

- [ ] 6. 統合と仕上げ
- [ ] 6.1 プラットフォーム間の動作確認
  - エンドツーエンドテスト
  - パフォーマンステスト
  - _Platform: Both_

- [ ] 6.2 エラーハンドリングとエッジケース対応
  - エラーメッセージの統一
  - 境界条件のテスト
  - _Platform: Both_
```

**Requirements Coverage Check**:
- Ensure ALL requirements are mapped to at least one task
- Cross-reference requirement IDs
- Report any unmapped requirements

**Metadata Update**:
```json
{
  "phase": "tasks-generated",
  "approvals": {
    "requirements": { "generated": true, "approved": true },
    "design": { "generated": true, "approved": true },
    "tasks": { "generated": true, "approved": false }
  },
  "ready_for_implementation": false,
  "updated_at": "current_timestamp"
}
```

**Output**:
```
✅ 実装タスクを生成しました

📄 .kiro/specs/[feature-name]/tasks.md

タスク概要:
- セットアップ: X個
- Flutter実装: Y個
- watchOS実装: Z個
- 同期実装: W個
- テスト: V個

合計: XX個のタスク

次のステップ:
1. tasks.md をレビュー
2. タスクの粒度と順序を確認
3. `/kiro/spec [feature-name]` で実装を開始
```

---

## Phase 4: Implementation

### Task: Execute Implementation Tasks

**Prerequisites**:
- Tasks generated and approved
- User confirmation or specific task number

**Implementation Options**:
```bash
# 次の未完了タスクを実行
/kiro/spec [feature-name] impl

# 特定のタスクを実行
/kiro/spec [feature-name] impl 2.1

# 複数タスクを実行
/kiro/spec [feature-name] impl 2.1,2.2,2.3

# すべての未完了タスクを実行（慎重に）
/kiro/spec [feature-name] impl --all
```

**Context Loading**:
- All steering documents (focus on platform-specific ones)
- Requirements, Design, Tasks documents
- Spec metadata
- Existing codebase for integration

**TDD Methodology** (Kent Beck's approach):

For each task:
1. **RED**: Write failing test first
2. **GREEN**: Write minimal code to pass test
3. **REFACTOR**: Clean up and improve

**Platform-Specific Implementation**:

**For Flutter tasks**:
- Load `flutter-best-practices.md`
- Follow Dart conventions
- Use established state management
- Write Widget/Unit tests
- Integrate with existing Flutter app structure

**For watchOS tasks**:
- Load `swift-best-practices.md`
- Follow Swift API Design Guidelines
- Use SwiftUI patterns
- Write XCTests
- Integrate with existing watchOS app structure

**For Sync tasks**:
- Load `data-sync.md`
- Implement on both platforms
- Test with mocks first
- Validate with real devices

**Task Completion**:
1. Implement functionality
2. Write and run tests
3. Verify all tests pass
4. Update checkbox: `- [ ]` → `- [x]` in tasks.md
5. Update spec.json:
   ```json
   {
     "phase": "in-implementation",
     "implementation_progress": {
       "total_tasks": XX,
       "completed_tasks": YY,
       "current_task": "2.1"
     }
   }
   ```

**Output**:
```
✅ タスク 2.1 を完了しました

実装内容:
- [実装したファイルリスト]
- [追加したテスト]

テスト結果: ✅ すべてパス

進捗: YY / XX タスク完了 (ZZ%)

次のタスク: 2.2 [サブ機能B] の開発

続けますか？ [y/N]
```

---

## Auto-All Mode (実験的)

⚠️ **警告**: このモードは慎重に使用してください。すべてのフェーズを自動実行します。

```bash
/kiro/spec [feature-name] --auto-all
```

**Process**:
1. 要件生成 → 自動承認
2. 設計生成 → 自動承認
3. タスク生成 → 自動承認
4. 実装開始 → 最初の未完了タスクのみ実行して停止

**Use Case**: 小規模な機能で、テンプレート的な実装の場合

---

## Error Handling

### Spec Not Found
```
❌ エラー: 機能 '[feature-name]' が見つかりません

先に初期化してください:
/kiro/init feature <機能説明>
```

### Approval Required
```
❌ エラー: [前フェーズ]が未承認です

レビュー後、以下のいずれかを実行:
1. `/kiro/spec [feature-name] [phase] -y` (自動承認)
2. 前フェーズのファイルを確認してから再実行
```

### File Conflict
```
⚠️ 警告: [file] は既に存在します

[o]verwrite / [m]erge / [c]ancel:
```

---

## Progress Tracking

各フェーズ完了時に `spec.json` を更新：
- `phase`: 現在のフェーズ
- `approvals`: 各フェーズの承認状態
- `updated_at`: 最終更新日時
- `implementation_progress`: 実装進捗（impl時のみ）

---

## Success Criteria

- [ ] ユーザーの意図を正確に解釈
- [ ] 現在のフェーズを正しく検出
- [ ] 適切な次のステップを提案
- [ ] プラットフォーム固有の考慮事項を反映
- [ ] 承認フローを適切に管理
- [ ] 進捗を明確に表示

ultrathink
