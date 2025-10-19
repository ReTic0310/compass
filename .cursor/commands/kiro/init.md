<meta>
description: Initialize project with steering documents and optional feature specification
argument-hint: [project|feature] <description>
</meta>

# Kiro Project & Feature Initialization

統合初期化コマンド - プロジェクト全体のステアリング作成、または機能仕様の初期化を実行します。

Tool policy: Use Cursor file tools (read_file/list_dir/glob_file_search/search_replace/write); no shell.

## Usage

```bash
# プロジェクト初期化（初回セットアップ）
/kiro/init project <project-description>

# 機能仕様の初期化
/kiro/init feature <feature-description>

# 混成プロジェクトの初期化（Flutter + Swift）
/kiro/init project --hybrid <description>
```

## Mode Detection

引数の最初の単語で動作モードを判定：
- `project` → プロジェクト初期化モード
- `feature` → 機能仕様初期化モード
- その他 → 既存の挙動を維持（feature指定と見なす）

## Project Initialization Mode

### Task: Create Project Steering Documents

プロジェクトの基盤となるステアリングドキュメントを作成します。

#### 1. Detect Project Type

プロジェクトタイプを自動検出または引数から判定：
- `--hybrid` フラグ: Flutter + SwiftUI混成プロジェクト
- ファイル検出: `pubspec.yaml` (Flutter), `Package.swift` (Swift), `*.xcodeproj` (iOS)
- デフォルト: 単一技術スタックプロジェクト

#### 2. Create Core Steering Documents

`.kiro/steering/` ディレクトリに以下を作成：

**A. product.md** - プロダクト概要
```markdown
# Product Overview

## Product Description
[プロジェクト説明から自動生成]

## Core Features
- [主要機能をリスト化]

## Target Users
- [ターゲットユーザー]

## Value Proposition
[提供価値]

## Target Platforms
<!-- For hybrid projects -->
- Android (Flutter)
- iOS (Flutter)
- watchOS (SwiftUI)
```

**B. tech.md** - 技術スタック

単一プロジェクト向け：
```markdown
# Technology Stack

## Architecture
[アーキテクチャ概要]

## Frontend/Mobile
[フレームワークと主要ライブラリ]

## Development Environment
[必要なツールとセットアップ]

## Common Commands
[よく使うコマンド]
```

混成プロジェクト向け（`--hybrid`時）：
```markdown
# Technology Stack

## Project Type
Hybrid Multi-Platform Project (Flutter + SwiftUI)

## Platform Architecture

### Flutter Platform (Android/iOS)
**Framework**: Flutter [version]
**Language**: Dart [version]
**Architecture**: [Clean Architecture / MVVM / etc.]
**Key Libraries**:
- [主要パッケージリスト]

### SwiftUI Platform (watchOS)
**Framework**: SwiftUI
**Language**: Swift [version]
**Architecture**: [MVVM / etc.]
**Key Frameworks**:
- WatchKit
- HealthKit (if applicable)
- [その他のフレームワーク]

## Data Synchronization Strategy
**Sync Method**: [WatchConnectivity / CloudKit / etc.]
**Data Flow**: [Flutter ↔ watchOS data exchange pattern]
**Conflict Resolution**: [競合解決戦略]

## Development Environment

### Flutter Development
- Flutter SDK: [version]
- Dart SDK: [version]
- IDE: VS Code / Android Studio
- Required Plugins: [リスト]

### Swift Development
- Xcode: [version]
- Swift: [version]
- watchOS SDK: [version]

## Common Commands

### Flutter
\`\`\`bash
flutter pub get
flutter run
flutter test
flutter build ios
flutter build apk
\`\`\`

### Swift/watchOS
\`\`\`bash
xcodebuild -workspace [workspace] -scheme [scheme]
\`\`\`

## Project Structure Integration
[Flutter プロジェクトと Swift プロジェクトの配置関係]
```

**C. structure.md** - プロジェクト構造

単一プロジェクト：
```markdown
# Project Structure

## Root Directory Organization
\`\`\`
project/
├── lib/           # メインコード
├── test/          # テストコード
├── docs/          # ドキュメント
└── .kiro/         # 仕様駆動開発ファイル
\`\`\`

## Code Organization
[コード構成パターン]

## File Naming Conventions
[命名規則]
```

混成プロジェクト：
```markdown
# Project Structure

## Hybrid Project Organization

\`\`\`
compass/
├── flutter_app/              # Flutter アプリケーション
│   ├── lib/
│   │   ├── features/         # 機能別モジュール
│   │   ├── core/             # 共通コア機能
│   │   ├── shared/           # 共有ユーティリティ
│   │   └── data/             # データ層（watchOS連携含む）
│   ├── test/
│   └── pubspec.yaml
│
├── watch_app/                # watchOS アプリケーション
│   ├── WatchApp/
│   │   ├── Views/            # SwiftUI ビュー
│   │   ├── ViewModels/       # ビューモデル
│   │   ├── Models/           # データモデル
│   │   ├── Services/         # サービス層（Flutter連携含む）
│   │   └── Utilities/        # ユーティリティ
│   ├── WatchAppTests/
│   └── WatchApp.xcodeproj
│
├── shared_models/            # 共有データモデル定義
│   ├── README.md            # データ同期仕様
│   └── schemas/             # スキーマ定義
│
└── .kiro/                   # 仕様駆動開発
    ├── steering/            # プロジェクト全体ガイドライン
    └── specs/               # 機能別仕様
\`\`\`

## Flutter App Structure

### Feature-based Organization
\`\`\`
lib/features/[feature_name]/
├── data/                    # データソース、リポジトリ実装
├── domain/                  # ビジネスロジック、エンティティ
├── presentation/            # UI、状態管理
└── watch_sync/              # watchOS 連携コード
\`\`\`

### Naming Conventions
- Files: `snake_case.dart`
- Classes: `PascalCase`
- Functions/Variables: `camelCase`
- Private: `_prefixed`

## watchOS App Structure

### MVVM Organization
\`\`\`
WatchApp/
├── Views/                   # SwiftUI ビュー
│   └── [FeatureName]View.swift
├── ViewModels/              # ビューモデル
│   └── [FeatureName]ViewModel.swift
├── Models/                  # データモデル
│   └── [ModelName].swift
└── Services/
    ├── WatchConnectivityService.swift  # Flutter連携
    └── [ServiceName]Service.swift
\`\`\`

### Naming Conventions
- Files: `PascalCase.swift`
- Types: `PascalCase`
- Functions/Variables: `camelCase`
- Private: `private` keyword

## Cross-Platform Patterns

### Data Synchronization
1. **Flutter → watchOS**: FlutterのデータをWatchConnectivity経由で送信
2. **watchOS → Flutter**: watchOSのイベントをFlutterで受信
3. **Conflict Resolution**: タイムスタンプベースまたはLast-Write-Wins

### Shared Data Models
- JSON Schema定義を`shared_models/`に配置
- 各プラットフォームでのモデルクラス生成
- バージョン管理とマイグレーション戦略
```

#### 3. Create Platform-Specific Steering (for hybrid projects)

混成プロジェクトの場合、追加のステアリングファイルを作成：

**D. .kiro/steering/flutter-best-practices.md**
```markdown
<!-- Inclusion Mode: Conditional: "flutter_app/**/*" -->

# Flutter Best Practices

## Dart Language Best Practices
- Null Safety の徹底
- `const` コンストラクタの活用
- Immutable データモデル

## Widget Design
- StatelessWidget 優先
- Widget 分割の粒度
- BuildContext の適切な使用

## State Management
- [採用する状態管理手法: Riverpod/Bloc/Provider]
- 状態スコープの設計
- 副作用の管理

## Performance
- 不要な rebuild の回避
- ListView.builder の使用
- Image caching

## Testing
- Widget tests
- Unit tests
- Integration tests

## Code Style
- Effective Dart に準拠
- `dart format` の使用
- Linter ルールの設定
```

**E. .kiro/steering/swift-best-practices.md**
```markdown
<!-- Inclusion Mode: Conditional: "watch_app/**/*" -->

# Swift & SwiftUI Best Practices

## Swift Language Best Practices
- Protocol-Oriented Programming
- Value Types (struct) 優先
- Optional の安全な扱い
- guard let vs if let の使い分け

## SwiftUI Design
- View の分割と再利用
- @State, @Binding, @ObservedObject の使い分け
- Environment の活用

## watchOS Specific
- Digital Crown 対応
- Complications 設計
- バッテリー効率の考慮
- 画面サイズ対応

## WatchConnectivity
- Session activation
- Message vs Context vs File transfer
- Background transfer considerations

## Testing
- XCTest framework
- UI Testing
- Async testing with expectations

## Code Style
- Swift API Design Guidelines準拠
- SwiftLint の使用
- Naming conventions
```

**F. .kiro/steering/data-sync.md**
```markdown
<!-- Inclusion Mode: Always -->

# Cross-Platform Data Synchronization

## Synchronization Architecture

### Communication Layer
**Primary Method**: WatchConnectivity Framework
- **Application Context**: 最新状態の同期
- **User Info Transfer**: バックグラウンド転送
- **Messages**: リアルタイム通信（アプリがアクティブ時）

### Data Flow Patterns

#### Flutter to watchOS
1. Flutter でデータ変更発生
2. Platform Channel 経由で iOS ネイティブコードへ
3. WatchConnectivity で watchOS へ送信
4. watchOS で受信・処理・UI更新

#### watchOS to Flutter
1. watchOS でデータ変更発生
2. WatchConnectivity で iOS へ送信
3. iOS で Platform Channel 経由で Flutter へ
4. Flutter で受信・処理・UI更新

## Data Model Versioning

### Schema Versioning
- `version` フィールドをすべてのデータモデルに含める
- 破壊的変更時はバージョン番号をインクリメント
- 下位互換性のためのマイグレーションパス

### Example Schema
\`\`\`json
{
  "version": "1.0.0",
  "type": "NavigationRoute",
  "data": {
    "routeId": "string",
    "waypoints": [],
    "currentPosition": {}
  }
}
\`\`\`

## Conflict Resolution Strategy

### Last-Write-Wins (LWW)
- タイムスタンプベースの競合解決
- 最新のタイムスタンプを持つデータが優先

### Platform Priority
- 特定のデータ型では特定プラットフォームを優先
- 例: 位置情報は Flutter優先、ヘルスデータは watchOS優先

## Error Handling

### Connection Failures
- Retry with exponential backoff
- Queue unsent data
- User notification for critical failures

### Data Validation
- Strict schema validation on both platforms
- Graceful degradation for unknown fields
- Error logging and monitoring

## Testing Strategy

### Integration Tests
- Mock WatchConnectivity layer
- Test data serialization/deserialization
- Verify conflict resolution logic
- Test offline scenarios

### Manual Testing
- Real device testing required
- Test with poor network conditions
- Battery impact testing
```

**G. .kiro/steering/security.md**
```markdown
<!-- Inclusion Mode: Always -->

# Security Guidelines

## General Security Principles
- Principle of Least Privilege
- Defense in Depth
- Secure by Default

## Data Protection

### Sensitive Data Handling
- Never log sensitive data (passwords, tokens, personal info)
- Use secure storage (Keychain for iOS/watchOS, encrypted_shared_preferences for Flutter)
- Encrypt data in transit (HTTPS, TLS)

### Flutter Security
- Use `flutter_secure_storage` for sensitive data
- Validate all user input
- Sanitize data before display
- Use HTTPS for all network requests

### Swift/watchOS Security
- Use Keychain Services for credentials
- Enable Data Protection API
- Secure WatchConnectivity messages

## API Security

### Authentication
- OAuth 2.0 / OpenID Connect 推奨
- Token refresh mechanism
- Secure token storage

### Authorization
- Role-based access control
- Permission checks at multiple layers
- Validate permissions on both client and server

## Code Security

### Input Validation
- Validate all external inputs
- Sanitize user-generated content
- Use parameterized queries (if applicable)

### Dependency Management
- Regular dependency updates
- Vulnerability scanning
- Use only trusted packages

### Code Review Checklist
- [ ] 機密情報のハードコーディングなし
- [ ] 適切なエラーハンドリング
- [ ] 入力バリデーション実装済み
- [ ] セキュアな通信（HTTPS）
- [ ] 適切な認証・認可

## Privacy

### User Data
- Minimum data collection
- Clear privacy policy
- User consent for data collection
- Data deletion mechanism

### Location Data (for Navigation App)
- Request location permission appropriately
- Explain why location is needed
- Option to disable location tracking
- Clear location data when not needed

## Vulnerability Response

### Security Issues
- Immediate patching of critical vulnerabilities
- Security advisory creation
- User notification for critical issues
- Post-mortem and prevention measures
```

#### 4. Create Custom Steering Template Registry

**H. .kiro/steering/_registry.json**
```json
{
  "version": "1.0.0",
  "project_type": "hybrid|single",
  "platforms": ["flutter", "swift", "swiftui"],
  "active_steering": {
    "core": [
      {
        "file": "product.md",
        "mode": "always",
        "description": "Product context and objectives"
      },
      {
        "file": "tech.md",
        "mode": "always",
        "description": "Technology stack and architecture"
      },
      {
        "file": "structure.md",
        "mode": "always",
        "description": "Project structure and organization"
      },
      {
        "file": "security.md",
        "mode": "always",
        "description": "Security guidelines and best practices"
      }
    ],
    "platform_specific": [
      {
        "file": "flutter-best-practices.md",
        "mode": "conditional",
        "patterns": ["flutter_app/**/*", "lib/**/*", "*.dart"],
        "description": "Flutter and Dart best practices"
      },
      {
        "file": "swift-best-practices.md",
        "mode": "conditional",
        "patterns": ["watch_app/**/*", "*.swift"],
        "description": "Swift and SwiftUI best practices"
      }
    ],
    "domain_specific": [
      {
        "file": "data-sync.md",
        "mode": "always",
        "description": "Cross-platform data synchronization"
      }
    ]
  },
  "created_at": "[timestamp]",
  "updated_at": "[timestamp]"
}
```

#### 5. Update AGENTS.md

AGENTS.mdにプロジェクトタイプと初期化情報を追加:
```markdown
# Project Information

## Project Type
[Hybrid Multi-Platform | Single Platform]

## Platforms
- Android (Flutter)
- iOS (Flutter)
- watchOS (SwiftUI)

## Initialization Date
[date]

## Active Specifications
<!-- Managed by /kiro/init feature -->
```

### Output for Project Mode

1. プロジェクトタイプの検出結果
2. 作成されたステアリングファイルのリスト
3. Next Steps:
   ```
   ✅ プロジェクト初期化完了
   
   作成されたステアリングファイル:
   - .kiro/steering/product.md
   - .kiro/steering/tech.md
   - .kiro/steering/structure.md
   - .kiro/steering/security.md
   [混成プロジェクトの場合]
   - .kiro/steering/flutter-best-practices.md
   - .kiro/steering/swift-best-practices.md
   - .kiro/steering/data-sync.md
   
   次のステップ:
   `/kiro/init feature <機能説明>` で機能仕様の作成を開始
   または
   `/kiro/status` でプロジェクト状態を確認
   ```

---

## Feature Initialization Mode

### Task: Initialize Feature Specification

機能仕様の初期化（従来の spec-init と同等）

#### 1. Generate Feature Name
プロジェクト説明から簡潔な機能名を生成
既存の `.kiro/specs/` ディレクトリをチェックして重複を回避

#### 2. Create Spec Directory
`.kiro/specs/[feature-name]/` に以下を作成：
- `spec.json` - メタデータと承認トラッキング
- `requirements.md` - 要件テンプレート

#### 3. Initialize spec.json
```json
{
  "feature_name": "[generated-feature-name]",
  "created_at": "current_timestamp",
  "updated_at": "current_timestamp",
  "language": "ja",
  "platforms": ["flutter", "swift"],
  "phase": "initialized",
  "approvals": {
    "requirements": {
      "generated": false,
      "approved": false
    },
    "design": {
      "generated": false,
      "approved": false
    },
    "tasks": {
      "generated": false,
      "approved": false
    }
  },
  "dependencies": [],
  "affected_platforms": ["flutter", "swift"],
  "ready_for_implementation": false
}
```

#### 4. Create Requirements Template
```markdown
# Requirements Document

## Project Description
<機能説明>

## Target Platforms
- [ ] Flutter (Android/iOS)
- [ ] SwiftUI (watchOS)
- [ ] Shared (データ同期が必要)

## Requirements
<!-- `/kiro/spec requirements` で生成 -->
```

#### 5. Update AGENTS.md
アクティブな仕様リストに追加

### Output for Feature Mode

1. 生成された機能名と根拠
2. 簡潔なプロジェクトサマリー
3. 作成された spec.json のパス
4. Next Steps:
   ```
   ✅ 機能仕様を初期化しました: [feature-name]
   
   次のステップ:
   `/kiro/spec requirements [feature-name]` で要件定義を生成
   
   または統合コマンド:
   `/kiro/spec [feature-name]` で要件から実装まで対話的に進行
   ```

---

## Instructions

1. **引数をパース** - モード判定（project/feature）
2. **プロジェクトタイプ検出** - ファイルシステムとフラグから判定
3. **適切なモードを実行** - Project または Feature 初期化
4. **完了メッセージ** - 明確な次のステップを提示

---

## Error Handling

- ステアリングディレクトリが既に存在する場合: 確認プロンプト [o]verwrite/[m]erge/[c]ancel
- 機能名が重複する場合: 自動的にサフィックスを追加
- 必須引数が不足: 使用方法を表示

## Success Criteria

- すべての必要なファイルが作成された
- spec.json/registry.json が有効なJSONである
- AGENTS.md が更新された
- ユーザーに明確な次のステップが提示された

ultrathink
