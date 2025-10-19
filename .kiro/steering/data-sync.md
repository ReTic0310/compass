# Data Synchronization Strategy - Compass

## Overview
このドキュメントはCompassプロジェクトにおけるクロスプラットフォームデータ同期戦略を定義します。
iOS (Flutter) と watchOS (SwiftUI) 間のシームレスなデータ同期を実現します。

## Synchronization Architecture

### Sync Topology
```
┌─────────────────┐
│  Flutter App    │
│  (iOS/Android)  │
└────────┬────────┘
         │
         │ WatchConnectivity (iOS ↔ watchOS)
         │
┌────────▼────────┐
│   watchOS App   │
└─────────────────┘
```

**Phase 1 (MVP)**: iOS ↔ watchOS のみ  
**Phase 2**: Android ↔ watchOS (クラウド経由または Bluetooth)  
**Phase 3**: マルチデバイス同期（Cloud Sync）

## Data to Synchronize

### Priority Levels

#### P0: Critical (Immediate Sync Required)
1. **Navigation State**
   - 現在のナビゲーション状態（アクティブ/非アクティブ）
   - 現在のルート情報
   - 現在位置
   - 次の案内ポイント

2. **User Actions**
   - ナビゲーション開始/停止
   - ルート再計算リクエスト

**Sync Method**: Real-time message (WCSession.sendMessage)  
**Latency Target**: < 500ms

#### P1: High (Background Sync)
1. **Route History**
   - 過去のルート履歴（最新30日分）
   - ルート詳細情報

2. **Favorite Locations**
   - お気に入り地点
   - 保存された場所

3. **User Settings**
   - アプリ設定
   - 表示設定

**Sync Method**: Background transfer (WCSession.transferUserInfo)  
**Latency Target**: < 5 seconds

#### P2: Normal (Periodic Sync)
1. **App State**
   - 最後に表示した画面
   - UI状態

2. **Cache Data**
   - マップタイル（必要に応じて）

**Sync Method**: Application context (WCSession.updateApplicationContext)  
**Latency Target**: < 30 seconds

## Sync Protocols

### 1. Real-Time Sync (P0 Data)

#### Message Format
```json
{
  "type": "navigation_update",
  "timestamp": "2025-10-19T10:30:00Z",
  "version": "1.0",
  "payload": {
    "navigation_state": "active",
    "route": {
      "id": "route-123",
      "start": {
        "latitude": 35.681236,
        "longitude": 139.767125,
        "name": "Tokyo Station"
      },
      "end": {
        "latitude": 35.689487,
        "longitude": 139.691706,
        "name": "Shinjuku Station"
      },
      "points": [...],
      "distance": 10500.0,
      "estimated_duration_seconds": 900
    },
    "current_location": {
      "latitude": 35.681500,
      "longitude": 139.767500
    },
    "next_instruction": {
      "distance_to_next": 250.0,
      "instruction": "Turn left at the next intersection"
    }
  }
}
```

#### Flutter Implementation
```dart
// lib/features/sync/data/data_sources/watch_sync_data_source.dart
import 'package:flutter/services.dart';

class WatchSyncDataSource {
  static const platform = MethodChannel('com.compass.watch_sync');

  Future<void> sendNavigationUpdate({
    required String navigationState,
    required Route route,
    required Location currentLocation,
    required Instruction? nextInstruction,
  }) async {
    final message = {
      'type': 'navigation_update',
      'timestamp': DateTime.now().toIso8601String(),
      'version': '1.0',
      'payload': {
        'navigation_state': navigationState,
        'route': route.toJson(),
        'current_location': currentLocation.toJson(),
        'next_instruction': nextInstruction?.toJson(),
      },
    };

    try {
      await platform.invokeMethod('sendMessage', message);
    } on PlatformException catch (e) {
      throw SyncException('Failed to send navigation update: ${e.message}');
    }
  }

  Stream<Map<String, dynamic>> watchMessages() {
    return EventChannel('com.compass.watch_sync/messages')
        .receiveBroadcastStream()
        .map((event) => Map<String, dynamic>.from(event as Map));
  }
}
```

#### iOS Bridge (Method Channel)
```swift
// ios/Runner/WatchSyncBridge.swift
import Flutter
import WatchConnectivity

class WatchSyncBridge: NSObject, FlutterPlugin {
    private let channel: FlutterMethodChannel
    private let eventChannel: FlutterEventChannel
    private var eventSink: FlutterEventSink?
    
    static func register(with registrar: FlutterPluginRegistrar) {
        let channel = FlutterMethodChannel(
            name: "com.compass.watch_sync",
            binaryMessenger: registrar.messenger()
        )
        let eventChannel = FlutterEventChannel(
            name: "com.compass.watch_sync/messages",
            binaryMessenger: registrar.messenger()
        )
        
        let instance = WatchSyncBridge(channel: channel, eventChannel: eventChannel)
        registrar.addMethodCallDelegate(instance, channel: channel)
        eventChannel.setStreamHandler(instance)
        
        // Setup WatchConnectivity
        if WCSession.isSupported() {
            WCSession.default.delegate = instance
            WCSession.default.activate()
        }
    }
    
    init(channel: FlutterMethodChannel, eventChannel: FlutterEventChannel) {
        self.channel = channel
        self.eventChannel = eventChannel
    }
    
    func handle(_ call: FlutterMethodCall, result: @escaping FlutterResult) {
        switch call.method {
        case "sendMessage":
            guard let message = call.arguments as? [String: Any] else {
                result(FlutterError(code: "INVALID_ARGS", message: "Invalid arguments", details: nil))
                return
            }
            sendMessage(message, result: result)
            
        default:
            result(FlutterMethodNotImplemented)
        }
    }
    
    private func sendMessage(_ message: [String: Any], result: @escaping FlutterResult) {
        guard WCSession.default.isReachable else {
            // Fallback to background transfer
            WCSession.default.transferUserInfo(message)
            result(nil)
            return
        }
        
        WCSession.default.sendMessage(
            message,
            replyHandler: { _ in
                result(nil)
            },
            errorHandler: { error in
                result(FlutterError(
                    code: "SEND_FAILED",
                    message: "Failed to send message",
                    details: error.localizedDescription
                ))
            }
        )
    }
}

// MARK: - WCSessionDelegate

extension WatchSyncBridge: WCSessionDelegate {
    func session(_ session: WCSession, activationDidCompleteWith activationState: WCSessionActivationState, error: Error?) {
        // Handle activation
    }
    
    func sessionDidBecomeInactive(_ session: WCSession) {}
    func sessionDidDeactivate(_ session: WCSession) {}
    
    func session(_ session: WCSession, didReceiveMessage message: [String : Any]) {
        // Forward to Flutter
        DispatchQueue.main.async { [weak self] in
            self?.eventSink?(message)
        }
    }
}

// MARK: - FlutterStreamHandler

extension WatchSyncBridge: FlutterStreamHandler {
    func onListen(withArguments arguments: Any?, eventSink events: @escaping FlutterEventSink) -> FlutterError? {
        self.eventSink = events
        return nil
    }
    
    func onCancel(withArguments arguments: Any?) -> FlutterError? {
        self.eventSink = nil
        return nil
    }
}
```

#### watchOS Implementation
```swift
// watch_app/Features/Sync/Services/WatchConnectivityService.swift
import Foundation
import WatchConnectivity
import Combine

final class WatchConnectivityService: NSObject, ObservableObject {
    static let shared = WatchConnectivityService()
    
    @Published var isConnected = false
    @Published var lastReceivedRoute: Route?
    
    private let session: WCSession
    private let messageSubject = PassthroughSubject<[String: Any], Never>()
    
    var messagePublisher: AnyPublisher<[String: Any], Never> {
        messageSubject.eraseToAnyPublisher()
    }
    
    private override init() {
        guard WCSession.isSupported() else {
            fatalError("WatchConnectivity is not supported")
        }
        
        session = WCSession.default
        super.init()
        
        session.delegate = self
        session.activate()
    }
    
    func sendNavigationAction(_ action: NavigationAction) {
        let message: [String: Any] = [
            "type": "navigation_action",
            "timestamp": ISO8601DateFormatter().string(from: Date()),
            "action": action.rawValue,
        ]
        
        sendMessage(message)
    }
    
    private func sendMessage(_ message: [String: Any]) {
        guard session.isReachable else {
            // Store for later or use background transfer
            session.transferUserInfo(message)
            return
        }
        
        session.sendMessage(message, replyHandler: nil) { error in
            print("Error sending message: \(error.localizedDescription)")
        }
    }
}

// MARK: - WCSessionDelegate

extension WatchConnectivityService: WCSessionDelegate {
    func session(_ session: WCSession, activationDidCompleteWith activationState: WCSessionActivationState, error: Error?) {
        DispatchQueue.main.async {
            self.isConnected = activationState == .activated && session.isReachable
        }
    }
    
    func sessionReachabilityDidChange(_ session: WCSession) {
        DispatchQueue.main.async {
            self.isConnected = session.isReachable
        }
    }
    
    func session(_ session: WCSession, didReceiveMessage message: [String : Any]) {
        messageSubject.send(message)
        
        // Parse and handle specific message types
        if let type = message["type"] as? String {
            switch type {
            case "navigation_update":
                handleNavigationUpdate(message)
            default:
                break
            }
        }
    }
    
    func session(_ session: WCSession, didReceiveUserInfo userInfo: [String : Any] = [:]) {
        // Handle background-transferred data
        messageSubject.send(userInfo)
    }
    
    private func handleNavigationUpdate(_ message: [String: Any]) {
        guard let payload = message["payload"] as? [String: Any],
              let routeData = payload["route"] as? [String: Any] else {
            return
        }
        
        do {
            let jsonData = try JSONSerialization.data(withJSONObject: routeData)
            let route = try JSONDecoder().decode(Route.self, from: jsonData)
            
            DispatchQueue.main.async {
                self.lastReceivedRoute = route
            }
        } catch {
            print("Failed to decode route: \(error)")
        }
    }
}

enum NavigationAction: String {
    case start
    case stop
    case recalculate
}
```

### 2. Background Sync (P1 Data)

#### Flutter Implementation
```dart
class WatchSyncDataSource {
  Future<void> syncRouteHistory(List<Route> routes) async {
    final data = {
      'type': 'route_history',
      'timestamp': DateTime.now().toIso8601String(),
      'routes': routes.map((r) => r.toJson()).toList(),
    };

    try {
      await platform.invokeMethod('transferUserInfo', data);
    } on PlatformException catch (e) {
      throw SyncException('Failed to sync route history: ${e.message}');
    }
  }

  Future<void> syncFavorites(List<Location> favorites) async {
    final data = {
      'type': 'favorites',
      'timestamp': DateTime.now().toIso8601String(),
      'locations': favorites.map((l) => l.toJson()).toList(),
    };

    try {
      await platform.invokeMethod('transferUserInfo', data);
    } on PlatformException catch (e) {
      throw SyncException('Failed to sync favorites: ${e.message}');
    }
  }
}
```

### 3. Application Context (P2 Data)

#### Flutter Implementation
```dart
class WatchSyncDataSource {
  Future<void> updateAppContext({
    required String lastScreen,
    required Map<String, dynamic> uiState,
  }) async {
    final context = {
      'last_screen': lastScreen,
      'ui_state': uiState,
      'updated_at': DateTime.now().toIso8601String(),
    };

    try {
      await platform.invokeMethod('updateApplicationContext', context);
    } on PlatformException catch (e) {
      throw SyncException('Failed to update context: ${e.message}');
    }
  }
}
```

## Conflict Resolution

### Strategy: Last-Write-Wins with Priority

#### Conflict Detection
```dart
class SyncConflictResolver {
  SyncResolution resolveConflict({
    required SyncData local,
    required SyncData remote,
  }) {
    // 1. Compare timestamps
    if (remote.timestamp.isAfter(local.timestamp)) {
      return SyncResolution.useRemote(remote);
    }
    
    // 2. Check priority (user action > automatic update)
    if (remote.priority > local.priority) {
      return SyncResolution.useRemote(remote);
    }
    
    // 3. Check device type priority
    // Active navigation device has highest priority
    if (remote.deviceType == DeviceType.activeNavigation) {
      return SyncResolution.useRemote(remote);
    }
    
    // 4. Default: keep local
    return SyncResolution.keepLocal(local);
  }
}

class SyncData {
  final DateTime timestamp;
  final int priority; // 0: automatic, 1: user action
  final DeviceType deviceType;
  final dynamic data;

  SyncData({
    required this.timestamp,
    required this.priority,
    required this.deviceType,
    required this.data,
  });
}

enum DeviceType {
  phone,
  watch,
  activeNavigation, // Device currently running navigation
}

sealed class SyncResolution {
  factory SyncResolution.useRemote(SyncData remote) = _UseRemote;
  factory SyncResolution.keepLocal(SyncData local) = _KeepLocal;
  factory SyncResolution.merge(SyncData local, SyncData remote) = _Merge;
}
```

## Data Models (Shared Schema)

### Route Schema
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Route",
  "type": "object",
  "required": ["id", "start", "end", "points", "distance", "estimated_duration_seconds"],
  "properties": {
    "id": {
      "type": "string",
      "description": "Unique route identifier"
    },
    "start": {
      "$ref": "#/definitions/Location"
    },
    "end": {
      "$ref": "#/definitions/Location"
    },
    "points": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/RoutePoint"
      }
    },
    "distance": {
      "type": "number",
      "description": "Total distance in meters"
    },
    "estimated_duration_seconds": {
      "type": "integer",
      "description": "Estimated duration in seconds"
    },
    "created_at": {
      "type": "string",
      "format": "date-time"
    }
  },
  "definitions": {
    "Location": {
      "type": "object",
      "required": ["latitude", "longitude"],
      "properties": {
        "latitude": {
          "type": "number",
          "minimum": -90,
          "maximum": 90
        },
        "longitude": {
          "type": "number",
          "minimum": -180,
          "maximum": 180
        },
        "name": {
          "type": "string"
        }
      }
    },
    "RoutePoint": {
      "type": "object",
      "required": ["latitude", "longitude"],
      "properties": {
        "latitude": {
          "type": "number"
        },
        "longitude": {
          "type": "number"
        },
        "instruction": {
          "type": "string"
        }
      }
    }
  }
}
```

## Error Handling & Retry

### Retry Strategy
```dart
class SyncRetryPolicy {
  static const maxRetries = 3;
  static const initialDelay = Duration(seconds: 1);
  static const maxDelay = Duration(seconds: 10);

  Future<T> executeWithRetry<T>({
    required Future<T> Function() operation,
    int retries = maxRetries,
  }) async {
    int attempt = 0;
    Duration delay = initialDelay;

    while (attempt < retries) {
      try {
        return await operation();
      } catch (e) {
        attempt++;
        
        if (attempt >= retries) {
          rethrow;
        }

        // Exponential backoff
        await Future.delayed(delay);
        delay = Duration(
          milliseconds: min(delay.inMilliseconds * 2, maxDelay.inMilliseconds),
        );
      }
    }

    throw SyncException('Max retries exceeded');
  }
}
```

### Error Recovery
```dart
class SyncErrorHandler {
  void handleSyncError(SyncError error) {
    switch (error.type) {
      case SyncErrorType.networkUnavailable:
        // Queue for later
        _queueForRetry(error.data);
        break;
        
      case SyncErrorType.watchNotReachable:
        // Use background transfer
        _useBackgroundTransfer(error.data);
        break;
        
      case SyncErrorType.dataCorrupted:
        // Log and request full sync
        _logError(error);
        _requestFullSync();
        break;
        
      case SyncErrorType.versionMismatch:
        // Alert user to update app
        _showUpdateRequired();
        break;
    }
  }
}
```

## Sync Status Monitoring

### Sync State Management
```dart
@freezed
class SyncState with _$SyncState {
  const factory SyncState.idle() = _Idle;
  const factory SyncState.syncing(SyncProgress progress) = _Syncing;
  const factory SyncState.synced(DateTime lastSyncTime) = _Synced;
  const factory SyncState.error(String message) = _Error;
}

class SyncProgress {
  final int completed;
  final int total;
  final String currentItem;

  SyncProgress({
    required this.completed,
    required this.total,
    required this.currentItem,
  });

  double get percentage => total > 0 ? completed / total : 0.0;
}
```

### Sync Monitoring UI
```dart
class SyncStatusWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final syncState = ref.watch(syncStateProvider);

    return syncState.when(
      idle: () => Icon(Icons.cloud_off),
      syncing: (progress) => Row(
        children: [
          CircularProgressIndicator(value: progress.percentage),
          Text('Syncing... ${(progress.percentage * 100).toInt()}%'),
        ],
      ),
      synced: (lastSyncTime) => Row(
        children: [
          Icon(Icons.cloud_done),
          Text('Synced ${_formatTime(lastSyncTime)}'),
        ],
      ),
      error: (message) => Row(
        children: [
          Icon(Icons.cloud_off, color: Colors.red),
          Text('Sync failed: $message'),
        ],
      ),
    );
  }
}
```

## Testing Sync

### Unit Tests
```dart
void main() {
  group('WatchSyncService', () {
    late WatchSyncService service;
    late MockWatchSyncDataSource mockDataSource;

    setUp(() {
      mockDataSource = MockWatchSyncDataSource();
      service = WatchSyncService(mockDataSource);
    });

    test('should send navigation update successfully', () async {
      // Arrange
      final route = Route(/* ... */);
      when(() => mockDataSource.sendNavigationUpdate(
        navigationState: any(named: 'navigationState'),
        route: any(named: 'route'),
        currentLocation: any(named: 'currentLocation'),
        nextInstruction: any(named: 'nextInstruction'),
      )).thenAnswer((_) async => {});

      // Act
      await service.syncNavigationState(route);

      // Assert
      verify(() => mockDataSource.sendNavigationUpdate(
        navigationState: 'active',
        route: route,
        currentLocation: any(named: 'currentLocation'),
        nextInstruction: any(named: 'nextInstruction'),
      )).called(1);
    });
  });
}
```

## Performance Optimization

### Data Compression
```dart
class SyncDataCompressor {
  static List<int> compress(String json) {
    return GZipCodec().encode(utf8.encode(json));
  }

  static String decompress(List<int> compressed) {
    final decompressed = GZipCodec().decode(compressed);
    return utf8.decode(decompressed);
  }
}

// Usage
final json = jsonEncode(route.toJson());
final compressed = SyncDataCompressor.compress(json);
await platform.invokeMethod('sendCompressedMessage', compressed);
```

### Batch Sync
```dart
class BatchSyncService {
  static const batchSize = 10;
  static const batchInterval = Duration(seconds: 5);

  final List<SyncItem> _queue = [];
  Timer? _batchTimer;

  void enqueue(SyncItem item) {
    _queue.add(item);

    if (_queue.length >= batchSize) {
      _flush();
    } else {
      _scheduleBatchFlush();
    }
  }

  void _scheduleBatchFlush() {
    _batchTimer?.cancel();
    _batchTimer = Timer(batchInterval, _flush);
  }

  void _flush() {
    if (_queue.isEmpty) return;

    final batch = List<SyncItem>.from(_queue);
    _queue.clear();
    _batchTimer?.cancel();

    _sendBatch(batch);
  }

  Future<void> _sendBatch(List<SyncItem> batch) async {
    // Send all items in one message
  }
}
```

---

**Last Updated**: 2025-10-19  
**Auto-Loaded**: Always (for cross-platform features)  
**Review Cycle**: Monthly

