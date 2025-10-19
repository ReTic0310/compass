# Shared Data Models - Compass

## Overview
このディレクトリには、Flutter App (iOS/Android) と watchOS App の間で共有されるデータモデルの定義が含まれています。

## Structure

```
shared_models/
├── schemas/            # JSONスキーマ定義
│   ├── route_schema.json
│   ├── location_schema.json
│   └── sync_message_schema.json
├── docs/              # データモデルのドキュメント
│   └── DATA_MODEL_SPEC.md
└── README.md (this file)
```

## Schemas

### route_schema.json
ナビゲーションルートのデータ構造を定義します。

**Key Fields**:
- `id`: ユニークなルート識別子 (UUID)
- `start`: 開始地点 (Location)
- `end`: 目的地 (Location)
- `points`: ルート上のウェイポイント配列
- `distance`: 総距離（メートル）
- `estimated_duration_seconds`: 推定所要時間（秒）

### location_schema.json
地理的位置情報のデータ構造を定義します。

**Key Fields**:
- `latitude`: 緯度 (-90 ~ 90)
- `longitude`: 経度 (-180 ~ 180)
- `altitude`: 高度（メートル）
- `accuracy`: 水平精度（メートル）
- `name`: 人間が読める地名
- `address`: 住所

### sync_message_schema.json
プラットフォーム間のデータ同期メッセージ形式を定義します。

**Message Types**:
- `navigation_update`: ナビゲーション状態の更新
- `navigation_action`: ナビゲーションアクション（開始/停止など）
- `route_history`: ルート履歴の同期
- `favorites`: お気に入り地点の同期
- `settings`: 設定の同期
- `location_update`: 位置情報の更新

## Usage

### Flutter (Dart)
```dart
// lib/shared/models/route_model.dart
import 'package:json_annotation/json_annotation.dart';

part 'route_model.g.dart';

@JsonSerializable()
class RouteModel {
  final String id;
  final LocationModel start;
  final LocationModel end;
  final List<RoutePointModel> points;
  final double distance;
  @JsonKey(name: 'estimated_duration_seconds')
  final int estimatedDurationSeconds;
  @JsonKey(name: 'created_at')
  final DateTime? createdAt;

  RouteModel({
    required this.id,
    required this.start,
    required this.end,
    required this.points,
    required this.distance,
    required this.estimatedDurationSeconds,
    this.createdAt,
  });

  factory RouteModel.fromJson(Map<String, dynamic> json) => 
      _$RouteModelFromJson(json);
  
  Map<String, dynamic> toJson() => _$RouteModelToJson(this);
}
```

### watchOS (Swift)
```swift
// watch_app/Shared/Models/Route.swift
import Foundation

struct Route: Codable, Identifiable, Equatable {
    let id: String
    let start: Location
    let end: Location
    let points: [RoutePoint]
    let distance: Double
    let estimatedDurationSeconds: Int
    let createdAt: Date?
    
    enum CodingKeys: String, CodingKey {
        case id, start, end, points, distance
        case estimatedDurationSeconds = "estimated_duration_seconds"
        case createdAt = "created_at"
    }
}

struct Location: Codable, Equatable {
    let latitude: Double
    let longitude: Double
    let altitude: Double?
    let accuracy: Double?
    let name: String?
    let address: String?
    let timestamp: Date?
}

struct RoutePoint: Codable, Equatable {
    let latitude: Double
    let longitude: Double
    let instruction: String?
    let distanceFromStart: Double?
    
    enum CodingKeys: String, CodingKey {
        case latitude, longitude, instruction
        case distanceFromStart = "distance_from_start"
    }
}
```

## Schema Validation

### Online Validation
JSONスキーマのバリデーションは以下のサイトで確認できます：
- [JSONSchema.net](https://www.jsonschema.net/)
- [JSON Schema Validator](https://www.jsonschemavalidator.net/)

### Dart Validation
```dart
import 'package:json_schema/json_schema.dart';

final schema = JsonSchema.create(schemaJson);
final validation = schema.validate(instanceJson);

if (!validation.isValid) {
  print('Validation errors: ${validation.errors}');
}
```

### Swift Validation
```swift
// Using JSONDecoder for validation
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601

do {
    let route = try decoder.decode(Route.self, from: jsonData)
    print("Valid route: \(route)")
} catch {
    print("Validation error: \(error)")
}
```

## Version Compatibility

### Schema Versioning
- **Major version**: Breaking changes (e.g., required field removal)
- **Minor version**: Backward-compatible additions
- **Current version**: 1.0

### Migration Strategy
新しいスキーマバージョンへの移行時：
1. 新しいスキーマファイルを追加（例: `route_schema_v2.json`）
2. 両方のバージョンをサポート（2バージョン並行期間）
3. データ移行ロジックを実装
4. 古いスキーマを非推奨化
5. 2リリース後に古いスキーマを削除

## Best Practices

### 1. Always Use Schemas
- スキーマ定義に従ってモデルクラスを生成
- コード生成ツールを活用（`json_serializable`, `Codable`）

### 2. Validate at Boundaries
- データ受信時に必ずバリデーション
- APIレスポンスのパース時にスキーマ検証

### 3. Handle Missing Fields Gracefully
- Optional フィールドには適切なデフォルト値
- 必須フィールドが欠けている場合はエラー

### 4. Keep Schemas Synchronized
- スキーマ変更時は両プラットフォームのモデルを更新
- CI/CDでスキーマの一貫性をテスト

### 5. Document Changes
- `CHANGELOG.md` にスキーマ変更を記録
- Breaking changes は明示的にマーク

## Testing

### Schema Validation Tests
```dart
// test/shared/models/route_model_test.dart
void main() {
  group('RouteModel', () {
    test('should deserialize from valid JSON', () {
      final json = {
        'id': 'route-123',
        'start': {'latitude': 35.0, 'longitude': 139.0},
        'end': {'latitude': 36.0, 'longitude': 140.0},
        'points': [],
        'distance': 10000.0,
        'estimated_duration_seconds': 600,
      };

      final route = RouteModel.fromJson(json);

      expect(route.id, 'route-123');
      expect(route.distance, 10000.0);
    });

    test('should throw on missing required field', () {
      final json = {
        'id': 'route-123',
        // Missing required fields
      };

      expect(
        () => RouteModel.fromJson(json),
        throwsA(isA<Exception>()),
      );
    });
  });
}
```

## Tools

### Schema Generation
- **From JSON**: [quicktype.io](https://quicktype.io/)
- **From Code**: [json_schema](https://pub.dev/packages/json_schema)

### Code Generation
- **Dart**: `build_runner` + `json_serializable`
- **Swift**: Native `Codable` protocol

## Support
スキーマに関する質問や提案は、プロジェクトのIssueトラッカーに投稿してください。

---

**Last Updated**: 2025-10-19  
**Schema Version**: 1.0  
**Maintainer**: Compass Development Team

