# Security Steering - Compass

## Security Principles

### Core Security Tenets
1. **Defense in Depth**: 多層防御戦略
2. **Least Privilege**: 最小権限の原則
3. **Secure by Default**: デフォルトで安全な設定
4. **Privacy First**: ユーザープライバシーの最優先
5. **Fail Securely**: エラー時も安全な状態を維持

## Data Security

### Sensitive Data Classification

#### Level 1: Public
- アプリバージョン情報
- 公開マップデータ
- 一般的な設定値

**Storage**: 暗号化不要
**Transmission**: HTTPS推奨

#### Level 2: Internal
- ユーザー設定（非個人）
- アプリ使用統計（匿名化済み）
- キャッシュデータ

**Storage**: デバイスストレージ（OS標準保護）
**Transmission**: HTTPS必須

#### Level 3: Confidential
- 位置情報履歴
- ルート履歴
- お気に入り地点

**Storage**: 暗号化必須
**Transmission**: HTTPS + Certificate Pinning（将来）

#### Level 4: Highly Confidential
- 認証トークン（将来）
- APIキー
- ユーザー認証情報（将来）

**Storage**: Secure Storage (Keychain/Keystore)
**Transmission**: HTTPS + Certificate Pinning + Token Expiration

### Data Encryption

#### At Rest Encryption

**Flutter**:
```dart
// Isar Database Encryption (AES-256)
final isar = await Isar.open(
  [RouteEntitySchema],
  directory: dir.path,
  encryption: EncryptionKey.fromSecureRandom(), // Store key in secure storage
);

// Secure Storage for sensitive data
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

const storage = FlutterSecureStorage();

// Store
await storage.write(key: 'api_key', value: apiKey);

// Retrieve
final apiKey = await storage.read(key: 'api_key');
```

**watchOS**:
```swift
// Keychain for sensitive data
import Security

class KeychainService {
    static func save(key: String, data: Data) -> Bool {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecValueData as String: data,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]
        
        SecItemDelete(query as CFDictionary)
        let status = SecItemAdd(query as CFDictionary, nil)
        return status == errSecSuccess
    }
    
    static func load(key: String) -> Data? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        
        return status == errSecSuccess ? result as? Data : nil
    }
}
```

#### In Transit Encryption

**Requirements**:
- TLS 1.3+ only
- Strong cipher suites only
- Certificate validation必須
- Certificate Pinning（Phase 2以降）

**Flutter Implementation**:
```dart
class SecureApiClient {
  late final Dio _dio;
  
  SecureApiClient() {
    _dio = Dio(BaseOptions(
      baseUrl: 'https://api.compass.app',
      connectTimeout: Duration(seconds: 10),
      receiveTimeout: Duration(seconds: 10),
    ));
    
    // Certificate Pinning (Phase 2)
    // (_dio.httpClientAdapter as DefaultHttpClientAdapter).onHttpClientCreate = (client) {
    //   client.badCertificateCallback = (cert, host, port) {
    //     return verifyCertificate(cert, host);
    //   };
    //   return client;
    // };
  }
}
```

### Data Retention & Deletion

#### Retention Policy
- **位置情報履歴**: 30日間（ユーザー設定可能）
- **ルート履歴**: 90日間（ユーザー設定可能）
- **キャッシュデータ**: 7日間
- **ログデータ**: 開発時のみ、本番では即時削除

#### Secure Deletion
```dart
// Flutter
class SecureDataDeletion {
  Future<void> deleteUserData() async {
    // 1. Database deletion
    final isar = await Isar.getInstance();
    await isar?.writeTxn(() async {
      await isar.routeEntitys.clear();
      await isar.locationEntitys.clear();
    });
    
    // 2. Secure storage deletion
    const storage = FlutterSecureStorage();
    await storage.deleteAll();
    
    // 3. Shared Preferences deletion
    final prefs = await SharedPreferences.getInstance();
    await prefs.clear();
    
    // 4. Cache deletion
    await DefaultCacheManager().emptyCache();
  }
}
```

```swift
// watchOS
class SecureDataDeletion {
    static func deleteAllUserData() {
        // 1. UserDefaults deletion
        if let bundleID = Bundle.main.bundleIdentifier {
            UserDefaults.standard.removePersistentDomain(forName: bundleID)
        }
        
        // 2. Keychain deletion
        let secItemClasses = [
            kSecClassGenericPassword,
            kSecClassInternetPassword,
            kSecClassCertificate,
            kSecClassKey,
            kSecClassIdentity
        ]
        
        for secItemClass in secItemClasses {
            let query: [String: Any] = [kSecClass as String: secItemClass]
            SecItemDelete(query as CFDictionary)
        }
        
        // 3. File system deletion
        let fileManager = FileManager.default
        if let documentsDirectory = fileManager.urls(for: .documentDirectory, in: .userDomainMask).first {
            try? fileManager.removeItem(at: documentsDirectory)
        }
    }
}
```

## Privacy Protection

### Location Privacy

#### Minimal Data Collection
- 位置情報はナビゲーション中のみ取得
- バックグラウンドでの位置情報取得は最小限
- 正確な位置情報が不要な場合は低精度モードを使用

#### Location Permission Handling

**Flutter**:
```dart
class LocationPermissionService {
  Future<bool> requestLocationPermission() async {
    // 1. Check current permission
    LocationPermission permission = await Geolocator.checkPermission();
    
    // 2. Show rationale if denied
    if (permission == LocationPermission.denied) {
      // Show explanation to user
      final shouldRequest = await _showPermissionRationale();
      
      if (!shouldRequest) return false;
      
      permission = await Geolocator.requestPermission();
    }
    
    // 3. Handle permanent denial
    if (permission == LocationPermission.deniedForever) {
      await _showSettingsDialog();
      return false;
    }
    
    return permission == LocationPermission.whileInUse ||
           permission == LocationPermission.always;
  }
  
  Future<bool> _showPermissionRationale() async {
    // Show dialog explaining why location is needed
    return true; // User's choice
  }
  
  Future<void> _showSettingsDialog() async {
    // Guide user to app settings
  }
}
```

**watchOS**:
```swift
import CoreLocation

class LocationPermissionService: NSObject, CLLocationManagerDelegate {
    private let locationManager = CLLocationManager()
    
    func requestLocationPermission() async -> Bool {
        locationManager.delegate = self
        
        let status = locationManager.authorizationStatus
        
        switch status {
        case .notDetermined:
            locationManager.requestWhenInUseAuthorization()
            // Wait for delegate callback
            return await withCheckedContinuation { continuation in
                // Handle async result
            }
            
        case .restricted, .denied:
            // Show settings alert
            return false
            
        case .authorizedWhenInUse, .authorizedAlways:
            return true
            
        @unknown default:
            return false
        }
    }
    
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        // Handle authorization change
    }
}
```

#### Location Data Anonymization
- ユーザー分析用のデータは位置情報を匿名化
- 正確な座標の代わりにグリッドベースの位置を使用（将来）
- 個人を特定できる情報を削除

### User Consent

#### Transparent Data Usage
- アプリ初回起動時にプライバシーポリシーを表示
- データ収集の目的を明確に説明
- ユーザーがデータ収集をオプトアウト可能

#### Consent Management
```dart
class ConsentManager {
  static const String _analyticsConsentKey = 'analytics_consent';
  static const String _locationHistoryConsentKey = 'location_history_consent';
  
  Future<bool> hasAnalyticsConsent() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getBool(_analyticsConsentKey) ?? false;
  }
  
  Future<void> setAnalyticsConsent(bool consent) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setBool(_analyticsConsentKey, consent);
  }
  
  Future<bool> hasLocationHistoryConsent() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getBool(_locationHistoryConsentKey) ?? true; // Default: enabled
  }
  
  Future<void> setLocationHistoryConsent(bool consent) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setBool(_locationHistoryConsentKey, consent);
    
    if (!consent) {
      // Delete existing location history
      await _deleteLocationHistory();
    }
  }
  
  Future<void> _deleteLocationHistory() async {
    // Implement secure deletion
  }
}
```

## Authentication & Authorization (Phase 2)

### Authentication Strategy

#### Planned Authentication Methods
1. **Apple Sign In** (iOS/watchOS)
2. **Google Sign In** (Android)
3. **Email/Password** (All platforms)

#### Token Management
```dart
class AuthTokenManager {
  static const String _accessTokenKey = 'access_token';
  static const String _refreshTokenKey = 'refresh_token';
  static const String _tokenExpiryKey = 'token_expiry';
  
  final FlutterSecureStorage _storage = const FlutterSecureStorage();
  
  Future<void> saveTokens({
    required String accessToken,
    required String refreshToken,
    required DateTime expiresAt,
  }) async {
    await _storage.write(key: _accessTokenKey, value: accessToken);
    await _storage.write(key: _refreshTokenKey, value: refreshToken);
    await _storage.write(key: _tokenExpiryKey, value: expiresAt.toIso8601String());
  }
  
  Future<String?> getAccessToken() async {
    final token = await _storage.read(key: _accessTokenKey);
    final expiryStr = await _storage.read(key: _tokenExpiryKey);
    
    if (token == null || expiryStr == null) return null;
    
    final expiry = DateTime.parse(expiryStr);
    if (DateTime.now().isAfter(expiry)) {
      // Token expired, refresh it
      return await _refreshAccessToken();
    }
    
    return token;
  }
  
  Future<String?> _refreshAccessToken() async {
    // Implement token refresh logic
    return null;
  }
  
  Future<void> clearTokens() async {
    await _storage.delete(key: _accessTokenKey);
    await _storage.delete(key: _refreshTokenKey);
    await _storage.delete(key: _tokenExpiryKey);
  }
}
```

## Input Validation & Sanitization

### User Input Validation

#### Location Search
```dart
class LocationInputValidator {
  static const int maxSearchLength = 200;
  static final RegExp allowedChars = RegExp(r'^[\p{L}\p{N}\s,.-]+$', unicode: true);
  
  static ValidationResult validate(String input) {
    // 1. Length check
    if (input.isEmpty) {
      return ValidationResult.error('検索語を入力してください');
    }
    
    if (input.length > maxSearchLength) {
      return ValidationResult.error('検索語が長すぎます（最大${maxSearchLength}文字）');
    }
    
    // 2. Character validation
    if (!allowedChars.hasMatch(input)) {
      return ValidationResult.error('使用できない文字が含まれています');
    }
    
    // 3. SQL injection prevention (if using SQL)
    final sanitized = _sanitizeInput(input);
    
    return ValidationResult.success(sanitized);
  }
  
  static String _sanitizeInput(String input) {
    // Remove potentially dangerous characters
    return input
        .replaceAll(RegExp(r'[<>"\';]'), '')
        .trim();
  }
}

class ValidationResult {
  final bool isValid;
  final String? message;
  final String? sanitizedValue;
  
  const ValidationResult.success(this.sanitizedValue)
      : isValid = true,
        message = null;
  
  const ValidationResult.error(this.message)
      : isValid = false,
        sanitizedValue = null;
}
```

### API Response Validation
```dart
class ApiResponseValidator {
  static T validateAndParse<T>({
    required dynamic response,
    required T Function(Map<String, dynamic>) fromJson,
  }) {
    // 1. Type check
    if (response is! Map<String, dynamic>) {
      throw ValidationException('Invalid response format');
    }
    
    // 2. Required fields check
    if (!_hasRequiredFields(response)) {
      throw ValidationException('Missing required fields');
    }
    
    // 3. Value range check
    if (!_validateValueRanges(response)) {
      throw ValidationException('Invalid value ranges');
    }
    
    // 4. Parse
    try {
      return fromJson(response);
    } catch (e) {
      throw ValidationException('Failed to parse response: $e');
    }
  }
  
  static bool _hasRequiredFields(Map<String, dynamic> response) {
    // Implement field validation
    return true;
  }
  
  static bool _validateValueRanges(Map<String, dynamic> response) {
    // Implement range validation (e.g., latitude: -90 to 90)
    return true;
  }
}
```

## Secure Communication

### API Security

#### Request Security
```dart
class SecureApiInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    // 1. Add security headers
    options.headers['X-API-Version'] = '1.0';
    options.headers['X-Device-ID'] = _getDeviceId();
    options.headers['X-Request-ID'] = _generateRequestId();
    
    // 2. Add authentication token (Phase 2)
    // final token = await _getAccessToken();
    // if (token != null) {
    //   options.headers['Authorization'] = 'Bearer $token';
    // }
    
    // 3. Validate request data
    if (options.data != null) {
      _validateRequestData(options.data);
    }
    
    super.onRequest(options, handler);
  }
  
  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    // Don't expose internal error details
    final safeError = _sanitizeError(err);
    super.onError(safeError, handler);
  }
  
  String _getDeviceId() {
    // Get secure device identifier
    return 'device_id';
  }
  
  String _generateRequestId() {
    return const Uuid().v4();
  }
  
  void _validateRequestData(dynamic data) {
    // Implement validation
  }
  
  DioException _sanitizeError(DioException error) {
    // Remove sensitive information from error messages
    return error;
  }
}
```

### Rate Limiting & Throttling

#### Client-Side Rate Limiting
```dart
class RateLimiter {
  final int maxRequests;
  final Duration window;
  final List<DateTime> _requestTimestamps = [];
  
  RateLimiter({
    required this.maxRequests,
    required this.window,
  });
  
  Future<bool> tryAcquire() async {
    final now = DateTime.now();
    final windowStart = now.subtract(window);
    
    // Remove old timestamps
    _requestTimestamps.removeWhere((t) => t.isBefore(windowStart));
    
    // Check limit
    if (_requestTimestamps.length >= maxRequests) {
      return false; // Rate limit exceeded
    }
    
    _requestTimestamps.add(now);
    return true;
  }
}

// Usage
final rateLimiter = RateLimiter(
  maxRequests: 10,
  window: Duration(minutes: 1),
);

Future<void> makeApiCall() async {
  if (!await rateLimiter.tryAcquire()) {
    throw RateLimitException('リクエストが多すぎます。しばらく待ってから再試行してください。');
  }
  
  // Proceed with API call
}
```

## Code Security

### Secure Coding Practices

#### No Hardcoded Secrets
```dart
// ❌ BAD: Hardcoded API key
class ApiClient {
  static const String apiKey = 'sk-1234567890abcdef';
}

// ✅ GOOD: Load from secure storage or environment
class ApiClient {
  late final String apiKey;
  
  Future<void> initialize() async {
    const storage = FlutterSecureStorage();
    apiKey = await storage.read(key: 'api_key') ?? '';
    
    if (apiKey.isEmpty) {
      throw SecurityException('API key not found');
    }
  }
}
```

#### Avoid Logging Sensitive Data
```dart
class SecureLogger {
  static void log(String message, {Map<String, dynamic>? data}) {
    // Filter sensitive data
    final sanitizedData = data != null ? _sanitizeData(data) : null;
    
    if (kDebugMode) {
      print('[$_timestamp] $message ${sanitizedData ?? ''}');
    }
  }
  
  static Map<String, dynamic> _sanitizeData(Map<String, dynamic> data) {
    final sanitized = Map<String, dynamic>.from(data);
    
    // Remove sensitive keys
    const sensitiveKeys = ['password', 'token', 'api_key', 'location'];
    for (final key in sensitiveKeys) {
      if (sanitized.containsKey(key)) {
        sanitized[key] = '[REDACTED]';
      }
    }
    
    return sanitized;
  }
  
  static String get _timestamp => DateTime.now().toIso8601String();
}
```

### Dependency Security

#### Regular Dependency Updates
```bash
# Flutter
flutter pub outdated
flutter pub upgrade

# watchOS
# Check for updates in Xcode or via Swift Package Manager
```

#### Dependency Vulnerability Scanning
- 定期的な脆弱性チェック（月次）
- 重大な脆弱性は即時対応
- `dart pub audit` の活用（将来）

### Code Obfuscation (Release Builds)

#### Flutter Obfuscation
```bash
# Build with obfuscation
flutter build apk --obfuscate --split-debug-info=build/debug-info
flutter build ios --obfuscate --split-debug-info=build/debug-info
```

**`pubspec.yaml`**:
```yaml
flutter:
  # Enable code shrinking
  uses-material-design: true
```

## Incident Response

### Security Incident Classification

#### Severity Levels
- **Critical (P0)**: データ漏洩、認証バイパス
- **High (P1)**: 重要機能の脆弱性
- **Medium (P2)**: 軽微な脆弱性
- **Low (P3)**: セキュリティ改善提案

### Incident Response Plan

#### Immediate Actions (0-2 hours)
1. インシデントの確認と記録
2. 影響範囲の特定
3. ステークホルダーへの通知
4. 緊急パッチの準備開始

#### Short-term Actions (2-24 hours)
1. パッチのテストとデプロイ
2. ユーザーへの通知（必要な場合）
3. 詳細な調査

#### Long-term Actions (1-7 days)
1. 根本原因の分析
2. 再発防止策の実装
3. セキュリティレビューの実施
4. ポストモーテムの作成

### Vulnerability Disclosure Policy

#### Reporting Channel
- Email: security@compass.app (将来)
- PGP Key for encrypted communication

#### Response Timeline
- 初回応答: 24時間以内
- 詳細調査: 7日以内
- パッチリリース: 30日以内（重大度による）

## Compliance & Standards

### Platform Compliance

#### Apple App Store
- App Transport Security (ATS) 準拠
- プライバシーラベルの正確な記載
- サードパーティSDKの開示

#### Google Play Store
- データセーフティセクションの記入
- 権限の正当な理由の説明
- 対象年齢の設定

### Privacy Regulations

#### GDPR Compliance (将来の国際展開)
- ユーザーデータへのアクセス権
- データポータビリティ
- 削除権（Right to be forgotten）
- 同意の管理

#### Local Regulations
- 各国の位置情報規制の遵守
- 地図表示規制（国境線など）

## Security Testing

### Security Test Checklist

#### Static Analysis
- [ ] Linter ルールの適用
- [ ] ハードコードされたシークレットのスキャン
- [ ] 依存関係の脆弱性チェック

#### Dynamic Analysis
- [ ] 認証・認可のテスト
- [ ] 入力バリデーションのテスト
- [ ] エラーハンドリングのテスト

#### Penetration Testing
- [ ] API エンドポイントのテスト
- [ ] データストレージのテスト
- [ ] 通信の暗号化テスト

### Security Audit Schedule
- **コードレビュー**: すべてのPR
- **自動セキュリティスキャン**: 毎日（CI/CD）
- **手動セキュリティレビュー**: 月次
- **外部セキュリティ監査**: 年次（将来）

---

**Last Updated**: 2025-10-19  
**Document Owner**: Security Team  
**Review Cycle**: Quarterly or after security incidents

