# Technical Steering - Compass

## Technology Stack

### Frontend

#### Flutter (Android & iOS)
- **Version**: Flutter 3.24+ (Stable Channel)
- **Dart Version**: 3.5+
- **Target Platforms**: Android 7.0+, iOS 14.0+
- **UI Framework**: Material Design 3 & Cupertino

**Key Dependencies**:
```yaml
dependencies:
  flutter:
    sdk: flutter
  
  # State Management
  riverpod: ^2.5.0
  flutter_riverpod: ^2.5.0
  
  # Navigation & Maps
  google_maps_flutter: ^2.8.0
  geolocator: ^12.0.0
  geocoding: ^3.0.0
  
  # Data Sync & Storage
  isar: ^3.1.0
  isar_flutter_libs: ^3.1.0
  
  # Network
  dio: ^5.7.0
  connectivity_plus: ^6.0.5
  
  # Platform Integration
  platform: ^3.1.5
  device_info_plus: ^10.1.2
```

#### watchOS (Apple Watch)
- **Version**: watchOS 9.0+
- **Language**: Swift 5.9+
- **UI Framework**: SwiftUI
- **Deployment Target**: watchOS 9.0

**Key Frameworks**:
- **WatchKit**: Apple Watch アプリのベース
- **HealthKit**: 健康データ統合（フィットネストラッキング用）
- **CoreLocation**: 位置情報サービス
- **MapKit**: マップ表示
- **WatchConnectivity**: iPhoneとの通信

### Backend & Services

#### Data Synchronization
- **Strategy**: デバイスローカル優先 + Cloud Backup
- **Primary Sync**: iOS <-> watchOS (WatchConnectivity)
- **Secondary Sync**: Android <-> iOS (Future: Firebase)

#### Map Services
- **Primary**: Apple MapKit (iOS/watchOS)
- **Secondary**: Google Maps API (Android)
- **Routing Engine**: Platform Native APIs

### Development Tools

#### Build & CI/CD
- **Flutter Build**: `flutter build apk/ipa`
- **watchOS Build**: Xcode 15+
- **Version Control**: Git
- **CI/CD**: (Future) GitHub Actions

#### Code Quality
- **Linter**: 
  - Dart: `flutter_lints` ^4.0.0
  - Swift: SwiftLint
- **Formatter**: 
  - Dart: `dart format`
  - Swift: `swift-format`
- **Static Analysis**: `dart analyze`

#### Testing
- **Flutter Testing**:
  - Unit Tests: `test` package
  - Widget Tests: `flutter_test`
  - Integration Tests: `integration_test`
  - Coverage Goal: 80%+

- **watchOS Testing**:
  - Unit Tests: XCTest
  - UI Tests: XCUITest
  - Coverage Goal: 70%+

## Architecture Decisions

### Overall Architecture Pattern
**Clean Architecture + Feature-First Structure**

```
┌─────────────────────────────────────────┐
│         Presentation Layer              │
│  (UI, ViewModels, State Management)     │
├─────────────────────────────────────────┤
│         Domain Layer                    │
│  (Business Logic, Entities, Use Cases)  │
├─────────────────────────────────────────┤
│         Data Layer                      │
│  (Repositories, Data Sources, Models)   │
└─────────────────────────────────────────┘
```

### Flutter Architecture

#### State Management: Riverpod
**Rationale**: 
- コンパイル時安全性
- テストが容易
- パフォーマンスが優れている
- Flutter 3.x との親和性

**Pattern**:
```dart
// Provider定義
final navigationServiceProvider = Provider<NavigationService>((ref) {
  return NavigationService(ref.read(locationRepositoryProvider));
});

// State Notifier
final routeStateProvider = StateNotifierProvider<RouteNotifier, RouteState>((ref) {
  return RouteNotifier(ref.read(navigationServiceProvider));
});
```

#### Directory Structure
```
flutter_app/
├── lib/
│   ├── core/
│   │   ├── constants/
│   │   ├── theme/
│   │   ├── utils/
│   │   └── network/
│   ├── features/
│   │   ├── navigation/
│   │   │   ├── data/
│   │   │   │   ├── models/
│   │   │   │   ├── repositories/
│   │   │   │   └── data_sources/
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   ├── repositories/
│   │   │   │   └── use_cases/
│   │   │   └── presentation/
│   │   │       ├── providers/
│   │   │       ├── screens/
│   │   │       └── widgets/
│   │   ├── sync/
│   │   └── settings/
│   ├── shared/
│   │   ├── models/
│   │   └── widgets/
│   └── main.dart
├── test/
├── integration_test/
└── pubspec.yaml
```

### watchOS Architecture

#### Architecture Pattern: MVVM + Combine
**Rationale**:
- SwiftUIとの自然な統合
- リアクティブプログラミング
- テスト可能性

**Structure**:
```
watch_app/
├── CompassWatch WatchKit Extension/
│   ├── Core/
│   │   ├── Constants/
│   │   ├── Extensions/
│   │   └── Utilities/
│   ├── Features/
│   │   ├── Navigation/
│   │   │   ├── Models/
│   │   │   ├── ViewModels/
│   │   │   ├── Views/
│   │   │   └── Services/
│   │   └── Sync/
│   ├── Shared/
│   │   ├── Models/
│   │   └── Components/
│   └── App/
│       └── CompassWatchApp.swift
└── CompassWatch WatchKit App/
    └── Assets.xcassets/
```

**Example ViewModel**:
```swift
@MainActor
class NavigationViewModel: ObservableObject {
    @Published var currentRoute: Route?
    @Published var isNavigating: Bool = false
    
    private let navigationService: NavigationService
    private let syncService: SyncService
    
    init(navigationService: NavigationService, syncService: SyncService) {
        self.navigationService = navigationService
        self.syncService = syncService
    }
    
    func startNavigation(to destination: Location) async {
        // Implementation
    }
}
```

### Data Synchronization Architecture

#### iOS <-> watchOS Sync
**Protocol**: WatchConnectivity Framework

**Sync Types**:
1. **Application Context**: 永続的な設定データ
2. **User Info Transfer**: バックグラウンド転送
3. **Message**: リアルタイムメッセージ（アクティブ時）

**Data Flow**:
```
Flutter App (iOS)
    ↕ (WatchConnectivity)
watchOS App
```

**Sync Strategy**:
- **Immediate Sync**: ナビゲーション状態、現在位置
- **Background Sync**: ルート履歴、お気に入り
- **On-Demand Sync**: 大容量データ（マップタイル）

#### Conflict Resolution
- **Last-Write-Wins**: タイムスタンプベース
- **Client Priority**: デバイスタイプ別の優先度
  - Active Navigation: 最も高い優先度
  - User Input: 中程度の優先度
  - Background Update: 最も低い優先度

### Data Storage

#### Flutter (Local Storage)
**Primary Database**: Isar
- **Rationale**: 
  - 高速（SQLiteより3-10倍高速）
  - 型安全
  - 自動マイグレーション
  - Flutter Web対応（将来的）

**Schema Example**:
```dart
@collection
class RouteEntity {
  Id id = Isar.autoIncrement;
  
  late String name;
  late DateTime startTime;
  late DateTime? endTime;
  
  late double startLat;
  late double startLng;
  late double endLat;
  late double endLng;
  
  List<RoutePoint> points = [];
  
  @Index()
  late DateTime createdAt;
}
```

#### watchOS (Local Storage)
**Primary Storage**: UserDefaults + File System
- **UserDefaults**: 軽量な設定とキャッシュ
- **File System**: ルートデータ（JSON）

**Rationale**: 
- watchOSのストレージ制約
- シンプルな読み書き
- WatchConnectivityとの統合が容易

### Network Layer

#### Flutter Network Architecture
**HTTP Client**: Dio
- **Rationale**:
  - Interceptorサポート
  - エラーハンドリング
  - タイムアウト管理
  - リトライロジック

**Structure**:
```dart
class ApiClient {
  final Dio _dio;
  
  ApiClient({required String baseUrl}) : _dio = Dio(
    BaseOptions(
      baseUrl: baseUrl,
      connectTimeout: Duration(seconds: 10),
      receiveTimeout: Duration(seconds: 10),
    ),
  ) {
    _dio.interceptors.add(LoggingInterceptor());
    _dio.interceptors.add(AuthInterceptor());
    _dio.interceptors.add(ErrorInterceptor());
  }
}
```

#### watchOS Network
**URLSession** with async/await
- シンプルなHTTPリクエスト
- バックグラウンドダウンロード対応

### Location Services

#### Flutter Location Strategy
**Primary**: `geolocator` package
- **Accuracy Levels**:
  - Navigation Mode: `LocationAccuracy.best` (GPS)
  - Background: `LocationAccuracy.balanced`
  - Battery Saver: `LocationAccuracy.low`

**Permission Handling**:
```dart
class LocationService {
  Future<bool> requestPermission() async {
    LocationPermission permission = await Geolocator.checkPermission();
    if (permission == LocationPermission.denied) {
      permission = await Geolocator.requestPermission();
    }
    return permission == LocationPermission.always ||
           permission == LocationPermission.whileInUse;
  }
}
```

#### watchOS Location Strategy
**CoreLocation Framework**
- **Workout Mode**: 継続的な位置情報取得（ナビゲーション中）
- **Significant Location Change**: バックグラウンド位置更新

### Performance Optimization

#### Flutter Optimization
1. **Lazy Loading**: 画面遷移時の遅延読み込み
2. **Image Caching**: `cached_network_image`
3. **List Performance**: `ListView.builder` for long lists
4. **Map Optimization**: 
   - 可視領域のみマーカー表示
   - マーカークラスタリング

#### watchOS Optimization
1. **Background Task Minimization**: 必要最小限のバックグラウンド処理
2. **ComplicationUpdate**: 効率的なコンプリケーション更新
3. **Battery Optimization**:
   - 位置情報取得間隔の調整
   - 不要なアニメーション削減

### Security Architecture

#### Data Encryption
- **At Rest**: 
  - Flutter: Isar encryption (AES-256)
  - watchOS: iOS Keychain for sensitive data
- **In Transit**: TLS 1.3+

#### Authentication & Authorization
- **Phase 1 (MVP)**: Device-local only (認証なし)
- **Phase 2**: OAuth 2.0 / Apple Sign In

#### Privacy
- **Location Data**: ローカルのみ保存、サーバーに送信しない（MVP）
- **Anonymization**: 将来的な分析データは匿名化

### Testing Strategy

#### Flutter Testing Pyramid
```
        E2E Tests (10%)
           ↑
      Integration Tests (20%)
           ↑
       Widget Tests (30%)
           ↑
       Unit Tests (40%)
```

#### watchOS Testing Strategy
```
      UI Tests (20%)
          ↑
    Integration Tests (30%)
          ↑
      Unit Tests (50%)
```

### Error Handling & Logging

#### Error Handling Pattern
```dart
sealed class Result<T> {
  const Result();
}

class Success<T> extends Result<T> {
  final T data;
  const Success(this.data);
}

class Failure<T> extends Result<T> {
  final String message;
  final Exception? exception;
  const Failure(this.message, {this.exception});
}
```

#### Logging Strategy
- **Development**: Console logging (verbose)
- **Production**: Error logging only
- **Future**: Crash reporting (Firebase Crashlytics)

### Dependency Injection

#### Flutter DI
**Riverpod Providers** as DI container
- Type-safe
- Compile-time safety
- Easy testing

#### watchOS DI
**Protocol-based Dependency Injection**
```swift
protocol NavigationServiceProtocol {
    func calculateRoute(to destination: Location) async throws -> Route
}

class NavigationService: NavigationServiceProtocol {
    // Implementation
}

// Injection
class NavigationViewModel {
    private let navigationService: NavigationServiceProtocol
    
    init(navigationService: NavigationServiceProtocol) {
        self.navigationService = navigationService
    }
}
```

### Build Configuration

#### Flutter Build Variants
- **Debug**: Development builds with debugging enabled
- **Profile**: Performance profiling
- **Release**: Production builds (optimized)

**Environment Configuration**:
```dart
enum Environment {
  development,
  staging,
  production,
}

class Config {
  static Environment get environment {
    // Determined at build time
  }
}
```

#### watchOS Build Configuration
- **Debug**: Simulator + Device testing
- **Release**: App Store distribution

### Version Management

#### Semantic Versioning
Format: `MAJOR.MINOR.PATCH`
- **MAJOR**: Breaking changes
- **MINOR**: New features (backward compatible)
- **PATCH**: Bug fixes

#### Build Number
- **Format**: `YYYYMMDD.HH` (e.g., 20251019.01)
- **Auto-increment**: On each CI build

### Migration Strategy

#### Database Migration
- **Isar**: Automatic schema migration
- **watchOS**: Manual migration with version checks

#### API Versioning
- **Header-based versioning**: `API-Version: 1.0`
- **Backward compatibility**: Maintain N-1 version support

### Monitoring & Analytics (Future)

#### Performance Monitoring
- App startup time
- Screen rendering time
- Network request latency
- Battery usage

#### Usage Analytics
- Feature usage
- User flow
- Error rates
- Crash reports

### Technology Debt Management

#### Code Review Requirements
- [ ] Follows architecture patterns
- [ ] Includes tests (80%+ coverage for Flutter, 70%+ for watchOS)
- [ ] No linter warnings
- [ ] Performance considerations documented

#### Refactoring Policy
- Monthly tech debt sprint
- Document known issues in `TECH_DEBT.md`
- Prioritize based on impact and effort

### Platform-Specific Considerations

#### Flutter Platform Differences
- **Material Design** on Android
- **Cupertino** on iOS
- Platform-adaptive widgets where appropriate

#### watchOS Limitations
- Limited background execution
- Small screen size constraints
- Limited local storage
- Battery constraints

---

**Last Updated**: 2025-10-19  
**Document Owner**: Tech Lead  
**Review Cycle**: Monthly

