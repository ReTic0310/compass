<meta>
description: Generate comprehensive test strategies and test code with TDD/BDD support
argument-hint: [feature-name] [test-type]
</meta>

# Test Strategy & Generation

包括的なテスト戦略の生成とTDD/BDD統合をサポートします。

Tool policy: Use Cursor file tools (read_file/list_dir/glob_file_search/search_replace/write); no shell.

## Usage

```bash
# テスト戦略全体を生成
/kiro/test [feature-name] strategy

# ユニットテスト生成
/kiro/test [feature-name] unit

# 統合テスト生成
/kiro/test [feature-name] integration

# E2Eテスト生成（Flutter Widget tests / watchOS UI tests）
/kiro/test [feature-name] e2e

# すべてのテストを生成
/kiro/test [feature-name] all

# TDDモード（テストファースト）
/kiro/test [feature-name] unit --tdd

# テストカバレッジレポート
/kiro/test [feature-name] coverage
```

---

## Test Strategy Generation

### Task: Create Comprehensive Test Strategy Document

**Context Loading**:
- `.kiro/specs/[feature-name]/requirements.md`
- `.kiro/specs/[feature-name]/design.md`
- `.kiro/specs/[feature-name]/tasks.md`
- `.kiro/steering/flutter-best-practices.md` (if Flutter)
- `.kiro/steering/swift-best-practices.md` (if Swift)
- `.kiro/steering/data-sync.md` (if cross-platform)

**Generate**: `.kiro/specs/[feature-name]/test-strategy.md`

**Document Structure**:

```markdown
# Test Strategy: [Feature Name]

## Overview

### Testing Goals
- 要件の完全なカバレッジ
- エッジケースと境界条件の検証
- クロスプラットフォーム動作の保証
- リグレッション防止

### Coverage Targets
- **Flutter**: 80%以上
- **watchOS**: 70%以上
- **同期レイヤー**: 90%以上（クリティカル）

## Testing Pyramid

\`\`\`mermaid
graph TB
    E2E[E2E Tests<br/>15%]
    Integration[Integration Tests<br/>25%]
    Unit[Unit Tests<br/>60%]
    
    Unit --> Integration
    Integration --> E2E
\`\`\`

### Test Distribution
- **Unit Tests (60%)**: ビジネスロジック、ユーティリティ、モデル
- **Integration Tests (25%)**: コンポーネント間連携、API、データフロー
- **E2E Tests (15%)**: ユーザーシナリオ、クリティカルパス

---

## Unit Tests

### Flutter Unit Tests

#### Test Targets
| Component | Test File | Priority | Test Cases |
|-----------|-----------|----------|-----------|
| [Component1] | [component1]_test.dart | High | 8 |
| [Component2] | [component2]_test.dart | High | 6 |
| [Service1] | [service1]_test.dart | High | 10 |

#### Example Test Structure
\`\`\`dart
// test/features/[feature]/domain/[component]_test.dart

import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

void main() {
  group('[ComponentName]', () {
    late ComponentName sut; // System Under Test
    late MockDependency mockDep;

    setUp(() {
      mockDep = MockDependency();
      sut = ComponentName(dependency: mockDep);
    });

    group('methodName', () {
      test('should return success when valid input provided', () {
        // Arrange
        final input = ValidInput();
        when(mockDep.method(any)).thenReturn(expectedValue);

        // Act
        final result = sut.methodName(input);

        // Assert
        expect(result.isSuccess, true);
        expect(result.value, expectedValue);
        verify(mockDep.method(any)).called(1);
      });

      test('should return error when invalid input provided', () {
        // Arrange
        final input = InvalidInput();

        // Act
        final result = sut.methodName(input);

        // Assert
        expect(result.isFailure, true);
        expect(result.error, isA<ValidationError>());
      });

      test('should handle edge case: empty input', () {
        // Arrange
        final input = EmptyInput();

        // Act
        final result = sut.methodName(input);

        // Assert
        expect(result.isFailure, true);
      });
    });
  });
}
\`\`\`

### watchOS Unit Tests

#### Test Targets
| Component | Test File | Priority | Test Cases |
|-----------|-----------|----------|-----------|
| [ViewModel1] | [ViewModel1]Tests.swift | High | 7 |
| [Service1] | [Service1]Tests.swift | High | 8 |
| [Model1] | [Model1]Tests.swift | Medium | 5 |

#### Example Test Structure
\`\`\`swift
// WatchAppTests/[ComponentName]Tests.swift

import XCTest
@testable import WatchApp

final class ComponentNameTests: XCTestCase {
    var sut: ComponentName!
    var mockDependency: MockDependency!
    
    override func setUp() {
        super.setUp()
        mockDependency = MockDependency()
        sut = ComponentName(dependency: mockDependency)
    }
    
    override func tearDown() {
        sut = nil
        mockDependency = nil
        super.tearDown()
    }
    
    func testMethodName_withValidInput_shouldReturnSuccess() {
        // Given
        let input = ValidInput()
        mockDependency.stub = expectedValue
        
        // When
        let result = sut.methodName(input: input)
        
        // Then
        XCTAssertTrue(result.isSuccess)
        XCTAssertEqual(result.value, expectedValue)
        XCTAssertEqual(mockDependency.callCount, 1)
    }
    
    func testMethodName_withInvalidInput_shouldReturnError() {
        // Given
        let input = InvalidInput()
        
        // When
        let result = sut.methodName(input: input)
        
        // Then
        XCTAssertFalse(result.isSuccess)
        XCTAssertTrue(result.error is ValidationError)
    }
    
    func testMethodName_withEdgeCase_shouldHandleGracefully() {
        // Given
        let input = EdgeCaseInput()
        
        // When & Then
        XCTAssertNoThrow(try sut.methodName(input: input))
    }
}
\`\`\`

---

## Integration Tests

### Flutter Integration Tests

#### Test Scenarios
| Scenario | Description | Components | Priority |
|----------|-------------|------------|----------|
| [Scenario1] | [Description] | A → B → C | High |
| [Scenario2] | [Description] | A → Sync | High |

#### Example: Data Flow Integration
\`\`\`dart
// test/integration/[feature]_data_flow_test.dart

import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('[Feature] Data Flow Integration', () {
    testWidgets('should sync data from repository to UI', (tester) async {
      // Arrange
      final repository = TestRepository();
      final bloc = FeatureBloc(repository: repository);
      
      await tester.pumpWidget(
        TestApp(child: FeaturePage(bloc: bloc)),
      );

      // Act
      repository.emitData(testData);
      await tester.pump();

      // Assert
      expect(find.text(testData.title), findsOneWidget);
      expect(bloc.state, isA<DataLoadedState>());
    });
  });
}
\`\`\`

### watchOS Integration Tests

#### Test Scenarios
| Scenario | Description | Components | Priority |
|----------|-------------|------------|----------|
| [Scenario1] | [Description] | ViewModel → Service | High |
| [Scenario2] | [Description] | Service → WatchConnectivity | High |

#### Example: ViewModel-Service Integration
\`\`\`swift
// WatchAppTests/Integration/[Feature]IntegrationTests.swift

import XCTest
@testable import WatchApp

final class FeatureIntegrationTests: XCTestCase {
    func testViewModelServiceIntegration_whenDataUpdates_shouldReflectInViewModel() async throws {
        // Given
        let service = RealService()
        let viewModel = FeatureViewModel(service: service)
        
        // When
        await service.updateData(newData)
        
        // Then
        let expectation = self.expectation(description: "ViewModel updated")
        
        viewModel.$state.sink { state in
            if case .loaded(let data) = state {
                XCTAssertEqual(data, newData)
                expectation.fulfill()
            }
        }.store(in: &cancellables)
        
        await fulfillment(of: [expectation], timeout: 2.0)
    }
}
\`\`\`

### Cross-Platform Sync Integration

#### Critical Sync Tests
| Test Case | Description | Priority |
|-----------|-------------|----------|
| Flutter → watchOS | データ送信と受信 | Critical |
| watchOS → Flutter | イベント送信と受信 | Critical |
| Conflict Resolution | 競合時の解決 | Critical |
| Offline Sync | オフライン時のキューイング | High |
| Data Validation | スキーマ検証 | High |

#### Example: Mock WatchConnectivity Test
\`\`\`dart
// test/integration/sync/flutter_to_watch_test.dart

void main() {
  group('Flutter to watchOS Sync', () {
    late MockWatchConnectivity mockConnectivity;
    late SyncService syncService;

    setUp(() {
      mockConnectivity = MockWatchConnectivity();
      syncService = SyncService(connectivity: mockConnectivity);
    });

    test('should send data to watchOS successfully', () async {
      // Arrange
      final data = NavigationData(route: testRoute);
      when(mockConnectivity.sendMessage(any))
          .thenAnswer((_) async => {'status': 'success'});

      // Act
      final result = await syncService.syncToWatch(data);

      // Assert
      expect(result.isSuccess, true);
      verify(mockConnectivity.sendMessage({
        'type': 'navigation',
        'data': data.toJson(),
      })).called(1);
    });

    test('should handle connection failure gracefully', () async {
      // Arrange
      final data = NavigationData(route: testRoute);
      when(mockConnectivity.sendMessage(any))
          .thenThrow(ConnectionException());

      // Act
      final result = await syncService.syncToWatch(data);

      // Assert
      expect(result.isFailure, true);
      expect(result.error, isA<SyncError>());
      // Verify data is queued for retry
      expect(syncService.pendingQueue.length, 1);
    });

    test('should validate data before sending', () async {
      // Arrange
      final invalidData = NavigationData(route: null);

      // Act
      final result = await syncService.syncToWatch(invalidData);

      // Assert
      expect(result.isFailure, true);
      expect(result.error, isA<ValidationError>());
      verifyNever(mockConnectivity.sendMessage(any));
    });
  });
}
\`\`\`

---

## E2E Tests

### Flutter Widget/E2E Tests

#### User Scenarios
| Scenario | Steps | Expected Outcome | Priority |
|----------|-------|------------------|----------|
| [Scenario1] | 1. Open app<br/>2. Navigate to feature<br/>3. Interact | [Expected] | Critical |
| [Scenario2] | [Steps] | [Expected] | High |

#### Example: Widget Test
\`\`\`dart
// test/features/[feature]/presentation/widgets/[widget]_test.dart

void main() {
  group('[WidgetName] Widget', () {
    testWidgets('should display data when loaded', (tester) async {
      // Arrange
      final mockBloc = MockFeatureBloc();
      when(mockBloc.stream).thenAnswer(
        (_) => Stream.value(LoadedState(data: testData)),
      );
      when(mockBloc.state).thenReturn(LoadedState(data: testData));

      // Act
      await tester.pumpWidget(
        TestApp(
          child: BlocProvider.value(
            value: mockBloc,
            child: FeatureWidget(),
          ),
        ),
      );
      await tester.pumpAndSettle();

      // Assert
      expect(find.text(testData.title), findsOneWidget);
      expect(find.byType(LoadingIndicator), findsNothing);
    });

    testWidgets('should handle user interaction', (tester) async {
      // Arrange
      final mockBloc = MockFeatureBloc();
      await tester.pumpWidget(
        TestApp(child: BlocProvider.value(
          value: mockBloc,
          child: FeatureWidget(),
        )),
      );

      // Act
      await tester.tap(find.byKey(Key('action-button')));
      await tester.pumpAndSettle();

      // Assert
      verify(mockBloc.add(ActionTriggered())).called(1);
    });
  });
}
\`\`\`

### watchOS UI Tests

#### User Scenarios
| Scenario | Steps | Expected Outcome | Priority |
|----------|-------|------------------|----------|
| [Scenario1] | 1. Launch watch app<br/>2. Navigate<br/>3. Verify | [Expected] | Critical |

#### Example: SwiftUI View Test
\`\`\`swift
// WatchAppTests/UI/[View]Tests.swift

import XCTest
import SwiftUI
import ViewInspector
@testable import WatchApp

final class FeatureViewTests: XCTestCase {
    func testView_whenDataLoaded_shouldDisplayContent() throws {
        // Given
        let viewModel = FeatureViewModel()
        viewModel.state = .loaded(testData)
        let view = FeatureView(viewModel: viewModel)
        
        // When
        let text = try view.inspect().find(text: testData.title)
        
        // Then
        XCTAssertNotNil(text)
    }
    
    func testView_whenButtonTapped_shouldTriggerAction() throws {
        // Given
        let viewModel = FeatureViewModel()
        let view = FeatureView(viewModel: viewModel)
        
        // When
        try view.inspect().find(button: "Action").tap()
        
        // Then
        XCTAssertEqual(viewModel.actionCallCount, 1)
    }
}
\`\`\`

---

## BDD Support (Behavior-Driven Development)

### Gherkin-Style Test Structure

#### Flutter BDD with `flutter_gherkin`

**Feature File**: `test_driver/features/[feature].feature`
\`\`\`gherkin
Feature: [Feature Name]
  As a [role]
  I want to [action]
  So that [benefit]

  Scenario: [Scenario Name]
    Given the app is launched
    And I am on the [page] screen
    When I tap the "[button]" button
    And I enter "[text]" into the "[field]" field
    Then I should see "[expected text]"
    And the "[element]" should be visible
\`\`\`

**Step Definitions**:
\`\`\`dart
// test_driver/steps/[feature]_steps.dart

import 'package:flutter_gherkin/flutter_gherkin.dart';
import 'package:gherkin/gherkin.dart';

class GivenAppLaunched extends Given1<String> {
  @override
  Future<void> executeStep(String input1) async {
    // Launch app logic
  }

  @override
  RegExp get pattern => RegExp(r"the app is launched");
}

class WhenTapButton extends When1<String> {
  @override
  Future<void> executeStep(String buttonLabel) async {
    final button = find.text(buttonLabel);
    await FlutterDriverUtils.tap(world, button);
  }

  @override
  RegExp get pattern => RegExp(r'I tap the {string} button');
}

class ThenSeeText extends Then1<String> {
  @override
  Future<void> executeStep(String expectedText) async {
    final finder = find.text(expectedText);
    await FlutterDriverUtils.isPresent(world, finder);
  }

  @override
  RegExp get pattern => RegExp(r'I should see {string}');
}
\`\`\`

#### watchOS BDD with XCTest

\`\`\`swift
// WatchAppTests/BDD/[Feature]BDDTests.swift

final class FeatureBDDTests: XCTestCase {
    func testScenario_userNavigatesToFeature_shouldDisplayContent() {
        // Given the watch app is launched
        let app = XCUIApplication()
        app.launch()
        
        // And I am on the home screen
        XCTAssertTrue(app.staticTexts["Home"].exists)
        
        // When I tap the "Feature" button
        app.buttons["Feature"].tap()
        
        // Then I should see "Feature Content"
        XCTAssertTrue(app.staticTexts["Feature Content"].waitForExistence(timeout: 2))
        
        // And the "Action" button should be visible
        XCTAssertTrue(app.buttons["Action"].exists)
    }
}
\`\`\`

---

## TDD Workflow

### Red-Green-Refactor Cycle

\`\`\`mermaid
graph LR
    Red[🔴 RED<br/>Write Failing Test] --> Green[🟢 GREEN<br/>Make Test Pass]
    Green --> Refactor[🔵 REFACTOR<br/>Improve Code]
    Refactor --> Red
\`\`\`

### TDD Mode Implementation

When using `--tdd` flag:

1. **Generate Test First** (RED)
   - Create test file with failing tests
   - Define expected behavior
   - Run tests (should fail)

2. **Prompt for Implementation** (GREEN)
   - Guide user to implement minimal code
   - Run tests until they pass

3. **Suggest Refactoring** (REFACTOR)
   - Identify code smells
   - Suggest improvements
   - Ensure tests still pass

#### Example TDD Session

\`\`\`bash
$ /kiro/test feature-navigation unit --tdd

🔴 RED Phase: テストを生成しました
📄 test/features/navigation/domain/route_calculator_test.dart

実行してください:
  flutter test test/features/navigation/domain/route_calculator_test.dart

期待される結果: すべてのテストが失敗する ✅

---

🟢 GREEN Phase: 実装を開始してください

最小限の実装で以下のテストを通過させてください:
  1. test: should calculate shortest route between two points
  2. test: should return error for invalid coordinates

ヒント:
  - RouteCalculator class を作成
  - calculateRoute() メソッドを実装
  - 基本的なバリデーションを追加

実装完了後、再度テストを実行してください:
  flutter test test/features/navigation/domain/route_calculator_test.dart

期待される結果: すべてのテストが成功する ✅

---

🔵 REFACTOR Phase: コードを改善しましょう

リファクタリング提案:
  1. 座標バリデーションロジックを別クラスに抽出
  2. エラーハンドリングをResult型で統一
  3. 定数をconstantsファイルに移動

リファクタリング後も、テストが通ることを確認:
  flutter test test/features/navigation/domain/route_calculator_test.dart
\`\`\`

---

## Test Coverage Analysis

### Task: Generate Coverage Report

**Generate**: `.kiro/specs/[feature-name]/test-coverage.md`

\`\`\`markdown
# Test Coverage Report: [Feature Name]

Generated: [timestamp]

## Overall Coverage

| Platform | Coverage | Target | Status |
|----------|----------|--------|--------|
| Flutter | 85% | 80% | ✅ |
| watchOS | 72% | 70% | ✅ |
| Sync Layer | 92% | 90% | ✅ |

## Flutter Coverage Breakdown

### By Layer
| Layer | Coverage | Files | Lines Covered |
|-------|----------|-------|---------------|
| Domain | 95% | 12 | 450/473 |
| Data | 80% | 8 | 320/400 |
| Presentation | 75% | 15 | 600/800 |

### Uncovered Critical Paths
1. `lib/features/[feature]/data/datasources/remote_datasource.dart:45-52`
   - Error handling for network timeout
   - **Priority**: High
   - **Action**: Add integration test for timeout scenario

2. `lib/features/[feature]/domain/usecases/calculate_route.dart:78-85`
   - Edge case: empty waypoints list
   - **Priority**: Medium
   - **Action**: Add unit test for empty input

## watchOS Coverage Breakdown

### By Component
| Component | Coverage | Files | Lines Covered |
|-----------|----------|-------|---------------|
| ViewModels | 85% | 5 | 170/200 |
| Services | 70% | 4 | 140/200 |
| Models | 60% | 3 | 90/150 |

### Uncovered Critical Paths
1. `WatchApp/Services/WatchConnectivityService.swift:120-135`
   - Background transfer handling
   - **Priority**: Critical
   - **Action**: Add integration test for background scenarios

## Recommendations

### Immediate Actions (Critical)
1. watchOS background transfer tests (priority: Critical)
2. Flutter network timeout handling (priority: High)

### Short Term (1-2 days)
1. Increase Model coverage to 70%
2. Add edge case tests for domain layer

### Long Term
1. Maintain 80%+ coverage for all new code
2. Quarterly coverage review
\`\`\`

---

## Test Generation

### Auto-Generate Tests

Based on requirements and design documents:

1. **Parse Requirements**: Extract EARS format acceptance criteria
2. **Map to Components**: Identify components from design
3. **Generate Test Cases**: Create test scenarios for each criterion
4. **Generate Test Code**: Scaffold test files with TODOs

#### Example: Auto-Generated Test from Requirement

**Requirement**:
\`\`\`markdown
### Requirement 1.1: Route Calculation
WHEN user selects destination THEN system SHALL calculate optimal route
\`\`\`

**Auto-Generated Test**:
\`\`\`dart
// test/features/navigation/domain/usecases/calculate_route_test.dart
// Auto-generated from Requirement 1.1

void main() {
  group('CalculateRoute', () {
    late CalculateRoute usecase;
    late MockRouteRepository mockRepository;

    setUp(() {
      mockRepository = MockRouteRepository();
      usecase = CalculateRoute(repository: mockRepository);
    });

    // From EARS: WHEN user selects destination 
    // THEN system SHALL calculate optimal route
    test('should calculate optimal route when user selects destination', () async {
      // TODO: Implement test based on design specification
      //
      // 1. Arrange: Set up test data (origin, destination)
      // 2. Act: Call usecase.execute()
      // 3. Assert: Verify optimal route is returned
      //
      // Design reference: RouteCalculator component (design.md line XX)
      // Related requirements: 1.1, 1.2
      
      fail('Test implementation pending');
    });

    // Edge case: invalid destination
    test('should return error when destination is invalid', () async {
      // TODO: Implement edge case test
      fail('Test implementation pending');
    });

    // Edge case: same origin and destination
    test('should handle same origin and destination gracefully', () async {
      // TODO: Implement edge case test
      fail('Test implementation pending');
    });
  });
}
\`\`\`

---

## Mock Generation

### Auto-Generate Mocks

For each interface/protocol in design:

**Flutter (with `mockito`)**:
\`\`\`dart
// test/mocks/mocks.dart
// Auto-generated mocks

import 'package:mockito/annotations.dart';

// Import interfaces from design
import 'package:app/features/[feature]/domain/repositories/route_repository.dart';
import 'package:app/features/[feature]/data/datasources/location_datasource.dart';

@GenerateMocks([
  RouteRepository,
  LocationDatasource,
])
void main() {}

// Run: flutter pub run build_runner build
\`\`\`

**watchOS (manual protocol mocks)**:
\`\`\`swift
// WatchAppTests/Mocks/MockRouteService.swift
// Auto-generated mock

@testable import WatchApp

final class MockRouteService: RouteServiceProtocol {
    var calculateRouteCallCount = 0
    var calculateRouteStub: Route?
    var calculateRouteShouldThrow = false
    
    func calculateRoute(from origin: Coordinate, to destination: Coordinate) async throws -> Route {
        calculateRouteCallCount += 1
        
        if calculateRouteShouldThrow {
            throw RouteError.calculationFailed
        }
        
        return calculateRouteStub ?? Route.mock
    }
}
\`\`\`

---

## Test Execution Scripts

### Flutter Test Runner

**Generate**: `.kiro/scripts/run_flutter_tests.sh`
\`\`\`bash
#!/bin/bash
# Auto-generated Flutter test runner

set -e

echo "🧪 Running Flutter Tests for [feature-name]..."

# Unit tests
echo "📦 Unit Tests..."
flutter test test/features/[feature]/domain/ \\
  test/features/[feature]/data/

# Widget tests
echo "🎨 Widget Tests..."
flutter test test/features/[feature]/presentation/

# Integration tests
echo "🔗 Integration Tests..."
flutter test test/integration/[feature]/

# Coverage
echo "📊 Generating Coverage..."
flutter test --coverage
genhtml coverage/lcov.info -o coverage/html

echo "✅ Tests Complete!"
echo "📈 Coverage report: coverage/html/index.html"
\`\`\`

### watchOS Test Runner

**Generate**: `.kiro/scripts/run_watch_tests.sh`
\`\`\`bash
#!/bin/bash
# Auto-generated watchOS test runner

set -e

echo "🧪 Running watchOS Tests for [feature-name]..."

xcodebuild test \\
  -workspace WatchApp.xcworkspace \\
  -scheme WatchApp \\
  -destination 'platform=watchOS Simulator,name=Apple Watch Series 9 (45mm)' \\
  -enableCodeCoverage YES \\
  | xcpretty

echo "✅ Tests Complete!"
\`\`\`

---

## Integration with Spec Workflow

### During Requirements Phase
- Generate test scenarios from EARS criteria
- Document testability of each requirement

### During Design Phase
- Identify testable contracts and interfaces
- Plan test doubles and mocks

### During Tasks Phase
- Include test tasks in implementation plan
- Map tests to requirements

### During Implementation
- Follow TDD: Write test → Implement → Refactor
- Update test-strategy.md with actual test results

---

## Success Criteria

- [ ] Test strategy document generated
- [ ] Test scaffolds created for all components
- [ ] Mocks generated for all dependencies
- [ ] Coverage targets defined and tracked
- [ ] TDD/BDD workflow integrated
- [ ] Platform-specific test patterns followed
- [ ] Cross-platform sync tests included
- [ ] Test execution scripts generated

ultrathink
