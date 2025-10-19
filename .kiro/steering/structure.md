# Project Structure Steering - Compass

## Repository Structure

```
compass/
├── .kiro/                          # Kiro SDD Framework
│   ├── steering/                   # Project-wide guidelines
│   │   ├── product.md
│   │   ├── tech.md
│   │   ├── structure.md (this file)
│   │   ├── security.md
│   │   ├── flutter-best-practices.md
│   │   ├── swift-best-practices.md
│   │   ├── data-sync.md
│   │   └── figma-integration.md
│   └── specs/                      # Feature specifications
│       └── [feature-name]/
│           ├── spec.json
│           ├── requirements.md
│           ├── design.md
│           ├── tasks.md
│           └── test-strategy.md
│
├── flutter_app/                    # Flutter Application
│   ├── android/                    # Android native code
│   ├── ios/                        # iOS native code
│   ├── lib/
│   │   ├── core/
│   │   │   ├── constants/
│   │   │   │   ├── app_constants.dart
│   │   │   │   ├── route_constants.dart
│   │   │   │   └── api_constants.dart
│   │   │   ├── theme/
│   │   │   │   ├── app_theme.dart
│   │   │   │   ├── colors.dart
│   │   │   │   └── text_styles.dart
│   │   │   ├── utils/
│   │   │   │   ├── logger.dart
│   │   │   │   ├── date_utils.dart
│   │   │   │   └── validators.dart
│   │   │   ├── network/
│   │   │   │   ├── api_client.dart
│   │   │   │   ├── network_info.dart
│   │   │   │   └── interceptors/
│   │   │   └── errors/
│   │   │       ├── failures.dart
│   │   │       └── exceptions.dart
│   │   │
│   │   ├── features/
│   │   │   ├── navigation/         # Navigation feature
│   │   │   │   ├── data/
│   │   │   │   │   ├── models/
│   │   │   │   │   │   ├── route_model.dart
│   │   │   │   │   │   └── location_model.dart
│   │   │   │   │   ├── repositories/
│   │   │   │   │   │   └── navigation_repository_impl.dart
│   │   │   │   │   └── data_sources/
│   │   │   │   │       ├── navigation_local_data_source.dart
│   │   │   │   │       └── navigation_remote_data_source.dart
│   │   │   │   ├── domain/
│   │   │   │   │   ├── entities/
│   │   │   │   │   │   ├── route.dart
│   │   │   │   │   │   └── location.dart
│   │   │   │   │   ├── repositories/
│   │   │   │   │   │   └── navigation_repository.dart
│   │   │   │   │   └── use_cases/
│   │   │   │   │       ├── start_navigation.dart
│   │   │   │   │       ├── calculate_route.dart
│   │   │   │   │       └── get_current_location.dart
│   │   │   │   └── presentation/
│   │   │   │       ├── providers/
│   │   │   │       │   ├── navigation_providers.dart
│   │   │   │       │   └── route_state_provider.dart
│   │   │   │       ├── screens/
│   │   │   │       │   ├── map_screen.dart
│   │   │   │       │   └── navigation_screen.dart
│   │   │   │       └── widgets/
│   │   │   │           ├── map_widget.dart
│   │   │   │           └── route_info_card.dart
│   │   │   │
│   │   │   ├── sync/                # Data synchronization feature
│   │   │   │   ├── data/
│   │   │   │   ├── domain/
│   │   │   │   └── presentation/
│   │   │   │
│   │   │   └── settings/            # Settings feature
│   │   │       ├── data/
│   │   │       ├── domain/
│   │   │       └── presentation/
│   │   │
│   │   ├── shared/
│   │   │   ├── models/              # Cross-platform shared models
│   │   │   │   └── sync_data_model.dart
│   │   │   └── widgets/             # Reusable widgets
│   │   │       ├── loading_indicator.dart
│   │   │       └── error_view.dart
│   │   │
│   │   └── main.dart                # Entry point
│   │
│   ├── test/                        # Unit & Widget tests
│   │   ├── core/
│   │   ├── features/
│   │   │   └── navigation/
│   │   │       ├── data/
│   │   │       ├── domain/
│   │   │       └── presentation/
│   │   └── test_helpers/
│   │       ├── mock_data.dart
│   │       └── test_utils.dart
│   │
│   ├── integration_test/            # Integration tests
│   │   └── app_test.dart
│   │
│   ├── assets/                      # Static assets
│   │   ├── images/
│   │   ├── icons/
│   │   └── fonts/
│   │
│   ├── pubspec.yaml                 # Flutter dependencies
│   └── README.md
│
├── watch_app/                       # watchOS Application
│   ├── CompassWatch.xcodeproj       # Xcode project
│   │
│   ├── CompassWatch WatchKit App/   # WatchKit app bundle
│   │   ├── Assets.xcassets/
│   │   ├── Info.plist
│   │   └── CompassWatch.entitlements
│   │
│   ├── CompassWatch WatchKit Extension/  # Watch app code
│   │   ├── Core/
│   │   │   ├── Constants/
│   │   │   │   ├── AppConstants.swift
│   │   │   │   └── ColorConstants.swift
│   │   │   ├── Extensions/
│   │   │   │   ├── View+Extensions.swift
│   │   │   │   └── Date+Extensions.swift
│   │   │   └── Utilities/
│   │   │       ├── Logger.swift
│   │   │       └── DateFormatter.swift
│   │   │
│   │   ├── Features/
│   │   │   ├── Navigation/
│   │   │   │   ├── Models/
│   │   │   │   │   ├── Route.swift
│   │   │   │   │   └── Location.swift
│   │   │   │   ├── ViewModels/
│   │   │   │   │   └── NavigationViewModel.swift
│   │   │   │   ├── Views/
│   │   │   │   │   ├── NavigationView.swift
│   │   │   │   │   └── RouteDetailView.swift
│   │   │   │   └── Services/
│   │   │   │       └── NavigationService.swift
│   │   │   │
│   │   │   └── Sync/
│   │   │       ├── Models/
│   │   │       ├── ViewModels/
│   │   │       ├── Views/
│   │   │       └── Services/
│   │   │           └── WatchConnectivityService.swift
│   │   │
│   │   ├── Shared/
│   │   │   ├── Models/              # Cross-platform models
│   │   │   │   └── SyncDataModel.swift
│   │   │   └── Components/          # Reusable SwiftUI components
│   │   │       ├── LoadingView.swift
│   │   │       └── ErrorView.swift
│   │   │
│   │   ├── App/
│   │   │   └── CompassWatchApp.swift  # App entry point
│   │   │
│   │   ├── Info.plist
│   │   └── CompassWatch_WatchKit_Extension.entitlements
│   │
│   ├── CompassWatchTests/           # Unit tests
│   │   ├── Features/
│   │   │   └── Navigation/
│   │   └── TestHelpers/
│   │       └── MockData.swift
│   │
│   ├── CompassWatchUITests/         # UI tests
│   │   └── NavigationUITests.swift
│   │
│   └── README.md
│
├── shared_models/                   # Shared data model definitions
│   ├── schemas/
│   │   ├── route_schema.json        # Route data schema
│   │   ├── location_schema.json     # Location data schema
│   │   └── sync_message_schema.json # Sync message schema
│   ├── docs/
│   │   └── DATA_MODEL_SPEC.md       # Data model documentation
│   └── README.md
│
├── docs/                            # Project documentation
│   ├── architecture/
│   │   ├── ARCHITECTURE.md          # Overall architecture
│   │   ├── DATA_FLOW.md             # Data flow diagrams
│   │   └── SYNC_STRATEGY.md         # Synchronization strategy
│   ├── api/
│   │   └── API_SPEC.md              # API specifications (future)
│   ├── setup/
│   │   ├── FLUTTER_SETUP.md
│   │   ├── WATCHOS_SETUP.md
│   │   └── DEVELOPMENT_SETUP.md
│   └── guides/
│       ├── CONTRIBUTING.md
│       └── CODE_REVIEW.md
│
├── scripts/                         # Utility scripts
│   ├── setup/
│   │   ├── setup_flutter.sh
│   │   └── setup_watchos.sh
│   ├── build/
│   │   ├── build_flutter.sh
│   │   └── build_watchos.sh
│   ├── test/
│   │   ├── run_flutter_tests.sh
│   │   └── run_watchos_tests.sh
│   └── sync/
│       └── sync_shared_models.sh    # Sync shared models between platforms
│
├── .cursor/                         # Cursor IDE configuration
│   └── commands/
│       └── kiro/                    # Kiro commands
│           └── commands.sh
│
├── .github/                         # GitHub configuration (future)
│   ├── workflows/
│   │   ├── flutter_ci.yml
│   │   └── watchos_ci.yml
│   └── ISSUE_TEMPLATE/
│
├── .gitignore                       # Git ignore patterns
├── .gitattributes                   # Git attributes
├── README.md                        # Project README
├── CHANGELOG.md                     # Version changelog
└── LICENSE                          # Project license
```

## File Naming Conventions

### Flutter (Dart)
- **Files**: `snake_case.dart`
- **Classes**: `PascalCase`
- **Variables/Functions**: `camelCase`
- **Constants**: `SCREAMING_SNAKE_CASE` or `kPascalCase`
- **Private members**: Prefix with `_`

**Examples**:
```dart
// File: navigation_service.dart
class NavigationService {
  static const int kMaxRetries = 3;
  
  final ApiClient _apiClient;
  String baseUrl = '';
  
  Future<void> calculateRoute() async { }
}
```

### watchOS (Swift)
- **Files**: `PascalCase.swift`
- **Classes/Structs/Enums**: `PascalCase`
- **Variables/Functions**: `camelCase`
- **Constants**: `camelCase` or `kPascalCase` (static let)
- **Private members**: Prefix with `private`

**Examples**:
```swift
// File: NavigationService.swift
class NavigationService {
    static let maxRetries = 3
    
    private let apiClient: APIClient
    var baseURL: String = ""
    
    func calculateRoute() async throws { }
}
```

### Shared Models
- **Schema files**: `snake_case.json`
- **Documentation**: `SCREAMING_SNAKE_CASE.md`

## Directory Organization Principles

### 1. Feature-First Structure
各機能は独立したディレクトリに配置し、その中でレイヤー（data/domain/presentation）を分離。

**Rationale**:
- 機能の追加・削除が容易
- チーム間の競合が減少
- 機能ごとの依存関係が明確

### 2. Layer Separation
Clean Architectureに基づく3層構造:
- **Presentation**: UI、State Management
- **Domain**: Business Logic、Entities
- **Data**: データ取得・永続化

**Dependency Rule**: 外側から内側への依存のみ（Domain層は他に依存しない）

### 3. Shared Components
複数の機能で使用されるコードは`shared/`または`core/`に配置。

**Guidelines**:
- 3回以上使用されるコードを共有化
- プラットフォーム間で共有可能なモデルは`shared_models/`に配置

### 4. Test Mirroring
テストディレクトリは実装ディレクトリの構造を反映。

**Example**:
```
lib/features/navigation/data/repositories/navigation_repository_impl.dart
test/features/navigation/data/repositories/navigation_repository_impl_test.dart
```

## Module Boundaries

### Flutter Modules

#### Core Module
**Responsibility**: アプリケーション全体で使用される基盤機能
- Constants
- Theme
- Utils
- Network
- Error handling

**Dependencies**: なし（他のモジュールに依存しない）

#### Feature Modules
**Responsibility**: 特定の機能を実装
- Navigation
- Sync
- Settings

**Dependencies**: Core Module、Domain entities

**Rule**: Feature間の直接依存は禁止（Shared経由で通信）

### watchOS Modules

#### Core Module
**Responsibility**: アプリケーション全体で使用される基盤機能
- Constants
- Extensions
- Utilities

#### Feature Modules
**Responsibility**: 特定の機能を実装
- Navigation
- Sync

**Rule**: Flutter同様、Feature間の直接依存は禁止

### Shared Models Module
**Responsibility**: プラットフォーム間でのデータ構造定義
- JSONスキーマ
- データモデル仕様書

**Consumers**: Flutter App、watchOS App

## Import Organization

### Flutter Import Order
```dart
// 1. Dart SDK imports
import 'dart:async';
import 'dart:io';

// 2. Flutter imports
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

// 3. Third-party package imports
import 'package:riverpod/riverpod.dart';
import 'package:geolocator/geolocator.dart';

// 4. Local imports - feature
import 'package:compass/features/navigation/domain/entities/route.dart';

// 5. Local imports - core
import 'package:compass/core/constants/app_constants.dart';

// 6. Relative imports (same feature)
import '../models/route_model.dart';
import '../../domain/repositories/navigation_repository.dart';
```

### Swift Import Order
```swift
// 1. System frameworks
import Foundation
import SwiftUI
import MapKit

// 2. Third-party frameworks
// (if any)

// 3. Local imports (grouped by feature)
import NavigationService
import SyncService
```

## Asset Organization

### Flutter Assets
```
assets/
├── images/
│   ├── common/
│   │   ├── logo.png
│   │   └── splash.png
│   └── navigation/
│       ├── marker.png
│       └── compass.png
├── icons/
│   └── app_icon.png
└── fonts/
    └── Roboto/
        ├── Roboto-Regular.ttf
        ├── Roboto-Bold.ttf
        └── Roboto-Italic.ttf
```

**pubspec.yaml**:
```yaml
flutter:
  assets:
    - assets/images/common/
    - assets/images/navigation/
    - assets/icons/
  fonts:
    - family: Roboto
      fonts:
        - asset: assets/fonts/Roboto/Roboto-Regular.ttf
        - asset: assets/fonts/Roboto/Roboto-Bold.ttf
          weight: 700
```

### watchOS Assets
```
Assets.xcassets/
├── AppIcon.appiconset/
├── Colors/
│   ├── PrimaryColor.colorset/
│   └── AccentColor.colorset/
└── Images/
    ├── marker.imageset/
    └── compass.imageset/
```

## Configuration Files

### Flutter Configuration
- **pubspec.yaml**: Dependencies, assets, Flutter settings
- **analysis_options.yaml**: Linter rules
- **flutter_app/android/app/build.gradle**: Android config
- **flutter_app/ios/Runner/Info.plist**: iOS config

### watchOS Configuration
- **Info.plist**: App metadata, permissions
- **CompassWatch.entitlements**: Capabilities (location, HealthKit)
- **.xcconfig files**: Build configuration

### Kiro Configuration
- **.kiro/specs/[feature]/spec.json**: Feature metadata
  ```json
  {
    "name": "navigation",
    "description": "Core navigation feature",
    "status": "in_progress",
    "phase": "design",
    "platforms": ["flutter", "watchos"],
    "language": "ja",
    "created_at": "2025-10-19T10:00:00Z",
    "updated_at": "2025-10-19T12:00:00Z"
  }
  ```

## Documentation Structure

### Code Documentation

#### Flutter (Dart)
```dart
/// NavigationServiceはルート計算とナビゲーション機能を提供します。
///
/// このサービスは位置情報を取得し、目的地までの最適なルートを計算します。
/// リアルタイムのナビゲーション案内も提供します。
///
/// Example:
/// ```dart
/// final service = NavigationService();
/// final route = await service.calculateRoute(destination);
/// ```
class NavigationService {
  /// 目的地までのルートを計算します。
  ///
  /// [destination] は目的地の位置情報です。
  /// 
  /// Returns 計算されたルートオブジェクト。
  /// Throws [NetworkException] ネットワークエラーが発生した場合。
  Future<Route> calculateRoute(Location destination) async {
    // Implementation
  }
}
```

#### watchOS (Swift)
```swift
/// NavigationServiceはルート計算とナビゲーション機能を提供します。
///
/// このサービスは位置情報を取得し、目的地までの最適なルートを計算します。
/// リアルタイムのナビゲーション案内も提供します。
///
/// Example:
/// ```swift
/// let service = NavigationService()
/// let route = try await service.calculateRoute(to: destination)
/// ```
class NavigationService {
    /// 目的地までのルートを計算します。
    ///
    /// - Parameter destination: 目的地の位置情報
    /// - Returns: 計算されたルートオブジェクト
    /// - Throws: `NetworkError` ネットワークエラーが発生した場合
    func calculateRoute(to destination: Location) async throws -> Route {
        // Implementation
    }
}
```

### README Structure
各モジュールに`README.md`を配置:
```markdown
# [Module Name]

## Overview
Brief description

## Structure
Directory structure explanation

## Usage
How to use this module

## Dependencies
List of dependencies

## Testing
How to run tests
```

## Version Control Strategy

### Git Workflow
- **Main Branch**: `main` - 常に安定したコード
- **Feature Branches**: `feature/[feature-name]` - 機能開発
- **Hotfix Branches**: `hotfix/[issue-number]` - 緊急修正

### Commit Message Convention
```
type(scope): subject

body (optional)

footer (optional)
```

**Types**:
- `feat`: 新機能
- `fix`: バグ修正
- `docs`: ドキュメント変更
- `style`: コードフォーマット
- `refactor`: リファクタリング
- `test`: テスト追加・修正
- `chore`: ビルドプロセス・補助ツール変更

**Example**:
```
feat(navigation): Add route calculation with real-time traffic

Implement route calculation that considers real-time traffic data.
Uses Google Maps API for Android and MapKit for iOS.

Closes #123
```

### .gitignore Strategy
プラットフォーム別の.gitignoreパターン:
- Flutter生成ファイル: `.dart_tool/`, `build/`
- Xcode生成ファイル: `*.xcworkspace/`, `DerivedData/`
- IDE設定: `.idea/`, `.vscode/` (ただし`.cursor/`は含める)
- 環境ファイル: `.env`、`*.env.local`

## Build Artifacts

### Flutter Build Outputs
```
flutter_app/build/
├── app/
│   ├── outputs/
│   │   └── flutter-apk/
│   │       └── app-release.apk
├── ios/
│   └── iphoneos/
│       └── Runner.app
```

### watchOS Build Outputs
```
watch_app/build/
└── Build/
    └── Products/
        ├── Debug-watchos/
        └── Release-watchos/
            └── CompassWatch.app
```

**Note**: Build artifactsは.gitignoreに含める

## Migration & Evolution

### Adding New Features
1. `/kiro/init feature` でspec初期化
2. `.kiro/specs/[feature-name]/` にドキュメント生成
3. `flutter_app/lib/features/` または `watch_app/Features/` に実装
4. テストを追加
5. READMEを更新

### Deprecating Features
1. コードに`@Deprecated`アノテーション追加
2. CHANGELOGに記載
3. ドキュメント更新
4. 2バージョン後に削除

### Platform-Specific File Patterns

#### Conditional Steering Loading
- **Flutter files**: `**/*.dart`, `flutter_app/**/*`
  - Auto-load: `flutter-best-practices.md`
- **Swift files**: `**/*.swift`, `watch_app/**/*`
  - Auto-load: `swift-best-practices.md`
- **Sync-related**: `**/sync/**/*`
  - Auto-load: `data-sync.md`

---

**Last Updated**: 2025-10-19  
**Document Owner**: Tech Lead  
**Review Cycle**: Quarterly or on major structural changes

