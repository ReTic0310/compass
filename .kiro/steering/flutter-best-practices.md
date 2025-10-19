# Flutter Best Practices - Compass

## Overview
このドキュメントはCompassプロジェクトにおけるFlutter開発のベストプラクティスを定義します。
このガイドラインは、Flutter/Dartファイルを編集する際に自動的に適用されます。

## Code Style & Formatting

### Dart Style Guide
公式の[Effective Dart](https://dart.dev/guides/language/effective-dart)に準拠します。

#### Naming Conventions
```dart
// Classes, enums, typedefs, type parameters: PascalCase
class NavigationService {}
enum RouteType {}
typedef RouteCallback = void Function(Route route);

// Libraries, packages, directories, source files: snake_case
// lib/features/navigation/data/repositories/navigation_repository_impl.dart

// Variables, constants, parameters, named parameters: camelCase
String userName = 'John';
void calculateRoute({required Location destination}) {}

// Constant values: lowerCamelCase with 'k' prefix (optional) or SCREAMING_SNAKE_CASE
const kMaxRetries = 3;
const Duration kAnimationDuration = Duration(milliseconds: 300);
static const String API_BASE_URL = 'https://api.compass.app';

// Private members: prefix with underscore
class NavigationService {
  final String _apiKey;
  void _calculateInternalRoute() {}
}
```

### Code Formatting
```bash
# Auto-format before commit
dart format .

# Check formatting
dart format --output=none --set-exit-if-changed .
```

**Editor Configuration** (`.editorconfig`):
```ini
[*.dart]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```

### Import Organization
```dart
// 1. Dart SDK
import 'dart:async';
import 'dart:io';

// 2. Flutter
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

// 3. Third-party packages (alphabetically)
import 'package:dio/dio.dart';
import 'package:riverpod/riverpod.dart';

// 4. Local project imports (alphabetically)
import 'package:compass/core/constants/app_constants.dart';
import 'package:compass/features/navigation/domain/entities/route.dart';

// 5. Relative imports (within same feature)
import '../models/route_model.dart';
import '../../domain/repositories/navigation_repository.dart';

// 6. Export statements last
export 'navigation_service.dart';
```

## Architecture Patterns

### Clean Architecture Structure

```
feature/
├── data/
│   ├── models/              # Data Transfer Objects
│   ├── repositories/        # Repository implementations
│   └── data_sources/        # Local & Remote data sources
├── domain/
│   ├── entities/            # Business objects
│   ├── repositories/        # Repository interfaces
│   └── use_cases/           # Business logic
└── presentation/
    ├── providers/           # Riverpod providers
    ├── screens/             # Full-screen widgets
    └── widgets/             # Reusable components
```

### Dependency Rule
依存関係は常に外側から内側へ:
```
Presentation → Domain ← Data
```

**Domain層はフレームワークに依存しない**（Pure Dart）

### Example: Feature Implementation

#### 1. Entity (Domain Layer)
```dart
// lib/features/navigation/domain/entities/route.dart
import 'package:equatable/equatable.dart';

class Route extends Equatable {
  final String id;
  final Location start;
  final Location end;
  final List<RoutePoint> points;
  final double distance;
  final Duration estimatedDuration;

  const Route({
    required this.id,
    required this.start,
    required this.end,
    required this.points,
    required this.distance,
    required this.estimatedDuration,
  });

  @override
  List<Object?> get props => [id, start, end, points, distance, estimatedDuration];
}
```

#### 2. Repository Interface (Domain Layer)
```dart
// lib/features/navigation/domain/repositories/navigation_repository.dart
import 'package:dartz/dartz.dart';
import '../entities/route.dart';
import '../../../../core/errors/failures.dart';

abstract class NavigationRepository {
  Future<Either<Failure, Route>> calculateRoute({
    required Location start,
    required Location end,
  });
  
  Future<Either<Failure, Location>> getCurrentLocation();
  
  Stream<Location> watchCurrentLocation();
}
```

#### 3. Use Case (Domain Layer)
```dart
// lib/features/navigation/domain/use_cases/calculate_route.dart
import 'package:dartz/dartz.dart';
import '../entities/route.dart';
import '../repositories/navigation_repository.dart';
import '../../../../core/errors/failures.dart';
import '../../../../core/use_cases/use_case.dart';

class CalculateRoute implements UseCase<Route, CalculateRouteParams> {
  final NavigationRepository repository;

  CalculateRoute(this.repository);

  @override
  Future<Either<Failure, Route>> call(CalculateRouteParams params) async {
    return await repository.calculateRoute(
      start: params.start,
      end: params.end,
    );
  }
}

class CalculateRouteParams {
  final Location start;
  final Location end;

  CalculateRouteParams({required this.start, required this.end});
}
```

#### 4. Model (Data Layer)
```dart
// lib/features/navigation/data/models/route_model.dart
import '../../domain/entities/route.dart';
import 'location_model.dart';
import 'route_point_model.dart';

class RouteModel extends Route {
  const RouteModel({
    required super.id,
    required super.start,
    required super.end,
    required super.points,
    required super.distance,
    required super.estimatedDuration,
  });

  factory RouteModel.fromJson(Map<String, dynamic> json) {
    return RouteModel(
      id: json['id'] as String,
      start: LocationModel.fromJson(json['start'] as Map<String, dynamic>),
      end: LocationModel.fromJson(json['end'] as Map<String, dynamic>),
      points: (json['points'] as List)
          .map((e) => RoutePointModel.fromJson(e as Map<String, dynamic>))
          .toList(),
      distance: (json['distance'] as num).toDouble(),
      estimatedDuration: Duration(seconds: json['estimated_duration_seconds'] as int),
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'start': (start as LocationModel).toJson(),
      'end': (end as LocationModel).toJson(),
      'points': points.map((e) => (e as RoutePointModel).toJson()).toList(),
      'distance': distance,
      'estimated_duration_seconds': estimatedDuration.inSeconds,
    };
  }
  
  Route toEntity() => this;
}
```

#### 5. Repository Implementation (Data Layer)
```dart
// lib/features/navigation/data/repositories/navigation_repository_impl.dart
import 'package:dartz/dartz.dart';
import '../../domain/entities/route.dart';
import '../../domain/repositories/navigation_repository.dart';
import '../../../../core/errors/exceptions.dart';
import '../../../../core/errors/failures.dart';
import '../../../../core/network/network_info.dart';
import '../data_sources/navigation_local_data_source.dart';
import '../data_sources/navigation_remote_data_source.dart';

class NavigationRepositoryImpl implements NavigationRepository {
  final NavigationRemoteDataSource remoteDataSource;
  final NavigationLocalDataSource localDataSource;
  final NetworkInfo networkInfo;

  NavigationRepositoryImpl({
    required this.remoteDataSource,
    required this.localDataSource,
    required this.networkInfo,
  });

  @override
  Future<Either<Failure, Route>> calculateRoute({
    required Location start,
    required Location end,
  }) async {
    if (await networkInfo.isConnected) {
      try {
        final routeModel = await remoteDataSource.calculateRoute(
          start: start,
          end: end,
        );
        await localDataSource.cacheRoute(routeModel);
        return Right(routeModel.toEntity());
      } on ServerException {
        return Left(ServerFailure());
      }
    } else {
      try {
        final cachedRoute = await localDataSource.getLastRoute();
        return Right(cachedRoute.toEntity());
      } on CacheException {
        return Left(CacheFailure());
      }
    }
  }

  @override
  Future<Either<Failure, Location>> getCurrentLocation() async {
    try {
      final location = await localDataSource.getCurrentLocation();
      return Right(location);
    } on LocationException {
      return Left(LocationFailure());
    }
  }

  @override
  Stream<Location> watchCurrentLocation() {
    return localDataSource.watchCurrentLocation();
  }
}
```

## State Management with Riverpod

### Provider Types & Usage

#### 1. Provider (Read-only, cached)
```dart
// For services, repositories
final navigationRepositoryProvider = Provider<NavigationRepository>((ref) {
  return NavigationRepositoryImpl(
    remoteDataSource: ref.read(navigationRemoteDataSourceProvider),
    localDataSource: ref.read(navigationLocalDataSourceProvider),
    networkInfo: ref.read(networkInfoProvider),
  );
});
```

#### 2. StateProvider (Simple mutable state)
```dart
// For simple state like toggles, counters
final mapZoomLevelProvider = StateProvider<double>((ref) => 15.0);

// Usage in widget
Consumer(
  builder: (context, ref, child) {
    final zoomLevel = ref.watch(mapZoomLevelProvider);
    return Text('Zoom: $zoomLevel');
  },
)

// Update
ref.read(mapZoomLevelProvider.notifier).state = 18.0;
```

#### 3. StateNotifierProvider (Complex mutable state)
```dart
// State class
@freezed
class RouteState with _$RouteState {
  const factory RouteState.initial() = _Initial;
  const factory RouteState.loading() = _Loading;
  const factory RouteState.loaded(Route route) = _Loaded;
  const factory RouteState.error(String message) = _Error;
}

// StateNotifier
class RouteNotifier extends StateNotifier<RouteState> {
  final CalculateRoute calculateRouteUseCase;

  RouteNotifier(this.calculateRouteUseCase) : super(const RouteState.initial());

  Future<void> calculateRoute(Location start, Location end) async {
    state = const RouteState.loading();
    
    final result = await calculateRouteUseCase(
      CalculateRouteParams(start: start, end: end),
    );

    result.fold(
      (failure) => state = RouteState.error(_mapFailureToMessage(failure)),
      (route) => state = RouteState.loaded(route),
    );
  }

  String _mapFailureToMessage(Failure failure) {
    switch (failure.runtimeType) {
      case ServerFailure:
        return 'サーバーエラーが発生しました';
      case NetworkFailure:
        return 'ネットワークに接続できません';
      default:
        return '予期しないエラーが発生しました';
    }
  }
}

// Provider
final routeNotifierProvider = StateNotifierProvider<RouteNotifier, RouteState>((ref) {
  return RouteNotifier(ref.read(calculateRouteUseCaseProvider));
});
```

#### 4. FutureProvider (Async data loading)
```dart
// For async operations that don't need to be triggered manually
final currentLocationProvider = FutureProvider<Location>((ref) async {
  final repository = ref.read(navigationRepositoryProvider);
  final result = await repository.getCurrentLocation();
  return result.fold(
    (failure) => throw Exception('Failed to get location'),
    (location) => location,
  );
});

// Usage
Consumer(
  builder: (context, ref, child) {
    final locationAsync = ref.watch(currentLocationProvider);
    
    return locationAsync.when(
      data: (location) => Text('Lat: ${location.latitude}'),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  },
)
```

#### 5. StreamProvider (Reactive data stream)
```dart
// For streams like location updates
final locationStreamProvider = StreamProvider<Location>((ref) {
  final repository = ref.read(navigationRepositoryProvider);
  return repository.watchCurrentLocation();
});

// Usage
Consumer(
  builder: (context, ref, child) {
    final locationAsync = ref.watch(locationStreamProvider);
    
    return locationAsync.when(
      data: (location) => MapWidget(center: location),
      loading: () => LoadingIndicator(),
      error: (error, stack) => ErrorView(message: error.toString()),
    );
  },
)
```

### Provider Best Practices

#### ✅ DO: Use family for parameterized providers
```dart
final routeDetailProvider = FutureProvider.family<Route, String>((ref, routeId) async {
  final repository = ref.read(navigationRepositoryProvider);
  return repository.getRouteById(routeId);
});

// Usage
ref.watch(routeDetailProvider('route-123'));
```

#### ✅ DO: Dispose resources properly
```dart
final timerProvider = StateNotifierProvider<TimerNotifier, int>((ref) {
  final notifier = TimerNotifier();
  ref.onDispose(() {
    notifier.dispose();
  });
  return notifier;
});
```

#### ❌ DON'T: Call providers inside build method without ref.watch
```dart
// ❌ BAD
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final value = ref.read(myProvider); // Won't rebuild
    return Text('$value');
  }
}

// ✅ GOOD
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final value = ref.watch(myProvider); // Will rebuild
    return Text('$value');
  }
}
```

## Widget Best Practices

### Widget Types

#### 1. StatelessWidget (Immutable UI)
```dart
class RouteInfoCard extends StatelessWidget {
  final Route route;

  const RouteInfoCard({
    super.key,
    required this.route,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Distance: ${route.distance} km'),
            Text('Duration: ${route.estimatedDuration.inMinutes} min'),
          ],
        ),
      ),
    );
  }
}
```

#### 2. ConsumerWidget (Riverpod-aware widget)
```dart
class NavigationScreen extends ConsumerWidget {
  const NavigationScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final routeState = ref.watch(routeNotifierProvider);

    return routeState.when(
      initial: () => const Center(child: Text('Enter destination')),
      loading: () => const Center(child: CircularProgressIndicator()),
      loaded: (route) => MapWidget(route: route),
      error: (message) => ErrorView(message: message),
    );
  }
}
```

#### 3. ConsumerStatefulWidget (Stateful + Riverpod)
```dart
class MapWidget extends ConsumerStatefulWidget {
  final Route route;

  const MapWidget({super.key, required this.route});

  @override
  ConsumerState<MapWidget> createState() => _MapWidgetState();
}

class _MapWidgetState extends ConsumerState<MapWidget> {
  late GoogleMapController _mapController;

  @override
  void initState() {
    super.initState();
    _initializeMap();
  }

  @override
  void dispose() {
    _mapController.dispose();
    super.dispose();
  }

  void _initializeMap() {
    // Initialization logic
  }

  @override
  Widget build(BuildContext context) {
    final currentLocation = ref.watch(locationStreamProvider);

    return currentLocation.when(
      data: (location) => GoogleMap(
        onMapCreated: (controller) => _mapController = controller,
        initialCameraPosition: CameraPosition(
          target: LatLng(location.latitude, location.longitude),
          zoom: 15,
        ),
      ),
      loading: () => const LoadingIndicator(),
      error: (error, _) => ErrorView(message: error.toString()),
    );
  }
}
```

### Widget Composition

#### ✅ DO: Extract widgets for reusability
```dart
// ✅ GOOD: Extracted widget
class _RouteInfoSection extends StatelessWidget {
  final Route route;

  const _RouteInfoSection({required this.route});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Distance: ${route.distance}'),
        Text('Duration: ${route.estimatedDuration}'),
      ],
    );
  }
}

class NavigationScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        _RouteInfoSection(route: route),
        MapWidget(route: route),
      ],
    );
  }
}
```

#### ❌ DON'T: Build complex widgets inline
```dart
// ❌ BAD: Inline complex widget
class NavigationScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Column(
          children: [
            Text('Distance: ${route.distance}'),
            Text('Duration: ${route.estimatedDuration}'),
            // ... many more widgets
          ],
        ),
        // ...
      ],
    );
  }
}
```

### const Constructors

#### ✅ DO: Use const wherever possible
```dart
// ✅ GOOD
const SizedBox(height: 16)
const EdgeInsets.all(8.0)
const Text('Hello')

class MyWidget extends StatelessWidget {
  const MyWidget({super.key}); // const constructor
  
  @override
  Widget build(BuildContext context) {
    return const Text('Constant widget');
  }
}
```

**Benefits**: Reduces rebuilds, improves performance

## Error Handling

### Result Pattern with Either
```dart
import 'package:dartz/dartz.dart';

// Failure classes
abstract class Failure {
  final String message;
  const Failure(this.message);
}

class ServerFailure extends Failure {
  const ServerFailure([super.message = 'サーバーエラーが発生しました']);
}

class NetworkFailure extends Failure {
  const NetworkFailure([super.message = 'ネットワークに接続できません']);
}

// Usage in repository
Future<Either<Failure, Route>> calculateRoute() async {
  try {
    final route = await _apiClient.calculateRoute();
    return Right(route);
  } on DioException catch (e) {
    if (e.type == DioExceptionType.connectionTimeout) {
      return Left(NetworkFailure());
    } else {
      return Left(ServerFailure());
    }
  } catch (e) {
    return Left(ServerFailure('予期しないエラー: $e'));
  }
}
```

### Exception Handling
```dart
// Custom exceptions
class ServerException implements Exception {
  final String message;
  const ServerException([this.message = 'Server error']);
}

class CacheException implements Exception {
  final String message;
  const CacheException([this.message = 'Cache error']);
}

// Exception handling in data source
Future<RouteModel> calculateRoute() async {
  try {
    final response = await dio.post('/calculate-route');
    
    if (response.statusCode == 200) {
      return RouteModel.fromJson(response.data);
    } else {
      throw ServerException('Invalid status code: ${response.statusCode}');
    }
  } on DioException {
    throw ServerException();
  }
}
```

## Testing

### Test Structure
```dart
void main() {
  group('NavigationService', () {
    late NavigationService service;
    late MockNavigationRepository mockRepository;

    setUp(() {
      mockRepository = MockNavigationRepository();
      service = NavigationService(mockRepository);
    });

    test('should return route when calculation succeeds', () async {
      // Arrange
      final tRoute = Route(/* ... */);
      when(() => mockRepository.calculateRoute(
        start: any(named: 'start'),
        end: any(named: 'end'),
      )).thenAnswer((_) async => Right(tRoute));

      // Act
      final result = await service.calculateRoute(start, end);

      // Assert
      expect(result, Right(tRoute));
      verify(() => mockRepository.calculateRoute(start: start, end: end)).called(1);
    });

    test('should return failure when calculation fails', () async {
      // Arrange
      when(() => mockRepository.calculateRoute(
        start: any(named: 'start'),
        end: any(named: 'end'),
      )).thenAnswer((_) async => Left(ServerFailure()));

      // Act
      final result = await service.calculateRoute(start, end);

      // Assert
      expect(result, Left(ServerFailure()));
    });
  });
}
```

### Widget Testing
```dart
void main() {
  testWidgets('RouteInfoCard displays route information', (tester) async {
    // Arrange
    final route = Route(
      id: '1',
      distance: 10.5,
      estimatedDuration: Duration(minutes: 15),
      // ...
    );

    // Act
    await tester.pumpWidget(
      MaterialApp(
        home: Scaffold(
          body: RouteInfoCard(route: route),
        ),
      ),
    );

    // Assert
    expect(find.text('Distance: 10.5 km'), findsOneWidget);
    expect(find.text('Duration: 15 min'), findsOneWidget);
  });
}
```

### Integration Testing
```dart
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('Complete navigation flow', (tester) async {
    await tester.pumpWidget(const MyApp());

    // Enter destination
    await tester.enterText(find.byKey(Key('destination-input')), 'Tokyo Station');
    await tester.tap(find.byKey(Key('search-button')));
    await tester.pumpAndSettle();

    // Verify route is displayed
    expect(find.byType(MapWidget), findsOneWidget);
    expect(find.byType(RouteInfoCard), findsOneWidget);

    // Start navigation
    await tester.tap(find.byKey(Key('start-navigation-button')));
    await tester.pumpAndSettle();

    // Verify navigation started
    expect(find.text('Navigating...'), findsOneWidget);
  });
}
```

## Performance Optimization

### 1. Use const Widgets
```dart
// ✅ GOOD
const Text('Hello')
const Icon(Icons.navigation)
```

### 2. Lazy Loading Lists
```dart
// ✅ GOOD: ListView.builder
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(title: Text(items[index]));
  },
)

// ❌ BAD: ListView with all children
ListView(
  children: items.map((item) => ListTile(title: Text(item))).toList(),
)
```

### 3. Image Optimization
```dart
// Use cached_network_image for network images
CachedNetworkImage(
  imageUrl: imageUrl,
  placeholder: (context, url) => CircularProgressIndicator(),
  errorWidget: (context, url, error) => Icon(Icons.error),
)

// Specify dimensions to avoid layout shifts
Image.asset(
  'assets/images/logo.png',
  width: 100,
  height: 100,
)
```

### 4. Avoid Rebuilds
```dart
// ✅ GOOD: Extract static widgets
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final data = ref.watch(dataProvider);
    
    return Column(
      children: [
        const _StaticHeader(), // Won't rebuild
        _DynamicContent(data: data), // Only this rebuilds
      ],
    );
  }
}
```

## Accessibility

### 1. Semantic Labels
```dart
IconButton(
  icon: Icon(Icons.navigation),
  onPressed: () {},
  tooltip: 'Start navigation',
  // For screen readers
  semanticsLabel: 'Start navigation button',
)
```

### 2. Minimum Touch Target Size
```dart
// Ensure at least 48x48 logical pixels
SizedBox(
  width: 48,
  height: 48,
  child: IconButton(
    icon: Icon(Icons.close),
    onPressed: () {},
  ),
)
```

### 3. Color Contrast
```dart
// Ensure sufficient contrast (WCAG AA: 4.5:1 for normal text)
Text(
  'Navigation',
  style: TextStyle(
    color: Colors.black, // Good contrast on white background
  ),
)
```

## Platform-Specific Code

### 1. Platform Detection
```dart
import 'dart:io' show Platform;
import 'package:flutter/foundation.dart' show kIsWeb;

if (Platform.isAndroid) {
  // Android-specific code
} else if (Platform.isIOS) {
  // iOS-specific code
}

if (kIsWeb) {
  // Web-specific code
}
```

### 2. Platform-Adaptive Widgets
```dart
// Use Cupertino on iOS, Material on Android
Widget buildButton() {
  if (Platform.isIOS) {
    return CupertinoButton(
      child: Text('Continue'),
      onPressed: () {},
    );
  } else {
    return ElevatedButton(
      child: Text('Continue'),
      onPressed: () {},
    );
  }
}

// Or use adaptive widgets
PlatformWidget(
  material: (context) => ElevatedButton(...),
  cupertino: (context) => CupertinoButton(...),
)
```

## Linting

### analysis_options.yaml
```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    # Style rules
    - always_declare_return_types
    - always_put_required_named_parameters_first
    - always_use_package_imports
    - avoid_print
    - avoid_unnecessary_containers
    - prefer_const_constructors
    - prefer_const_declarations
    - prefer_final_fields
    - prefer_single_quotes
    - require_trailing_commas
    - sort_child_properties_last
    - sort_constructors_first
    - unawaited_futures
    
    # Error prevention
    - avoid_dynamic_calls
    - avoid_slow_async_io
    - cancel_subscriptions
    - close_sinks
    - no_adjacent_strings_in_list
    - use_build_context_synchronously

analyzer:
  exclude:
    - '**/*.g.dart'
    - '**/*.freezed.dart'
  errors:
    invalid_annotation_target: ignore
```

---

**Last Updated**: 2025-10-19  
**Auto-Loaded**: When working with Flutter/Dart files  
**Review Cycle**: Quarterly

