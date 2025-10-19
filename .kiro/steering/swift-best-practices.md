# Swift Best Practices - Compass watchOS

## Overview
このドキュメントはCompassプロジェクトにおけるwatchOS (Swift + SwiftUI) 開発のベストプラクティスを定義します。
このガイドラインは、Swift/SwiftUIファイルを編集する際に自動的に適用されます。

## Code Style & Formatting

### Swift Style Guide
Apple公式の[Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)に準拠します。

#### Naming Conventions
```swift
// Types (classes, structs, enums, protocols): PascalCase
class NavigationService {}
struct Route {}
enum RouteType {}
protocol NavigationServiceProtocol {}

// Functions, variables, constants: camelCase
func calculateRoute() {}
var userName: String = ""
let maxRetries = 3

// Type-associated constants: camelCase with static let
struct Constants {
    static let animationDuration: TimeInterval = 0.3
    static let maxZoomLevel: Double = 20.0
}

// Private members: camelCase (no underscore prefix)
class NavigationService {
    private let apiKey: String
    private func calculateInternalRoute() {}
}

// Boolean variables: use "is", "has", "should" prefix
var isNavigating: Bool = false
var hasActiveRoute: Bool = false
var shouldShowAlert: Bool = true

// Delegate protocols: end with "Delegate"
protocol NavigationServiceDelegate: AnyObject {
    func navigationService(_ service: NavigationService, didUpdateRoute route: Route)
}
```

### Code Formatting
```swift
// Indentation: 4 spaces
func calculateRoute(from start: Location, to end: Location) -> Route? {
    guard let route = routeCalculator.calculate(start: start, end: end) else {
        return nil
    }
    return route
}

// Vertical spacing
class NavigationService {
    // MARK: - Properties
    
    private let locationManager: CLLocationManager
    private var currentRoute: Route?
    
    // MARK: - Initialization
    
    init(locationManager: CLLocationManager) {
        self.locationManager = locationManager
    }
    
    // MARK: - Public Methods
    
    func startNavigation() {
        // Implementation
    }
    
    // MARK: - Private Methods
    
    private func updateRoute() {
        // Implementation
    }
}
```

### SwiftLint Configuration
`.swiftlint.yml`:
```yaml
disabled_rules:
  - trailing_whitespace
  
opt_in_rules:
  - empty_count
  - empty_string
  - explicit_init
  - first_where
  - sorted_imports
  
included:
  - watch_app
  
excluded:
  - watch_app/Pods
  - watch_app/build
  
line_length:
  warning: 120
  error: 200
  
type_body_length:
  warning: 300
  error: 400
  
file_length:
  warning: 500
  error: 1000
  
function_body_length:
  warning: 50
  error: 100
  
identifier_name:
  min_length:
    warning: 2
  max_length:
    warning: 40
```

### Import Organization
```swift
// 1. System frameworks (alphabetically)
import Combine
import CoreLocation
import Foundation
import MapKit
import SwiftUI
import WatchKit

// 2. Third-party frameworks (alphabetically)
// import ThirdPartyFramework

// 3. Internal modules (alphabetically)
import NavigationService
import SyncService
```

## Architecture Patterns

### MVVM Architecture

```
Feature/
├── Models/              # Data models
├── ViewModels/          # Business logic & state
├── Views/               # SwiftUI views
└── Services/            # API, data sources
```

### Example: Feature Implementation

#### 1. Model
```swift
// Models/Route.swift
import Foundation

struct Route: Identifiable, Codable, Equatable {
    let id: String
    let start: Location
    let end: Location
    let points: [RoutePoint]
    let distance: Double
    let estimatedDuration: TimeInterval
    
    var formattedDistance: String {
        String(format: "%.1f km", distance / 1000)
    }
    
    var formattedDuration: String {
        let minutes = Int(estimatedDuration / 60)
        return "\(minutes) min"
    }
}

struct Location: Codable, Equatable {
    let latitude: Double
    let longitude: Double
    let name: String?
    
    var coordinate: CLLocationCoordinate2D {
        CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
    }
}

struct RoutePoint: Codable, Equatable {
    let latitude: Double
    let longitude: Double
    let instruction: String?
}
```

#### 2. Service (Data Layer)
```swift
// Services/NavigationService.swift
import Foundation
import CoreLocation
import Combine

protocol NavigationServiceProtocol {
    func calculateRoute(from start: Location, to end: Location) async throws -> Route
    func getCurrentLocation() async throws -> Location
    func startLocationUpdates() -> AnyPublisher<Location, Error>
}

final class NavigationService: NavigationServiceProtocol {
    // MARK: - Properties
    
    private let locationManager: CLLocationManager
    private let apiClient: APIClient
    
    // MARK: - Initialization
    
    init(locationManager: CLLocationManager = CLLocationManager(), 
         apiClient: APIClient = APIClient()) {
        self.locationManager = locationManager
        self.apiClient = apiClient
    }
    
    // MARK: - Public Methods
    
    func calculateRoute(from start: Location, to end: Location) async throws -> Route {
        let request = RouteRequest(start: start, end: end)
        
        do {
            let response = try await apiClient.post(endpoint: "/calculate-route", body: request)
            return try JSONDecoder().decode(Route.self, from: response)
        } catch {
            throw NavigationError.calculationFailed(error)
        }
    }
    
    func getCurrentLocation() async throws -> Location {
        guard locationManager.authorizationStatus == .authorizedWhenInUse ||
              locationManager.authorizationStatus == .authorizedAlways else {
            throw NavigationError.locationPermissionDenied
        }
        
        // Get current location
        let location = try await withCheckedThrowingContinuation { continuation in
            // Implementation using CLLocationManager delegate
        }
        
        return Location(
            latitude: location.coordinate.latitude,
            longitude: location.coordinate.longitude,
            name: nil
        )
    }
    
    func startLocationUpdates() -> AnyPublisher<Location, Error> {
        // Implementation using Combine
        return locationManager.locationPublisher
            .map { clLocation in
                Location(
                    latitude: clLocation.coordinate.latitude,
                    longitude: clLocation.coordinate.longitude,
                    name: nil
                )
            }
            .mapError { $0 as Error }
            .eraseToAnyPublisher()
    }
}

// MARK: - Errors

enum NavigationError: LocalizedError {
    case calculationFailed(Error)
    case locationPermissionDenied
    case networkError
    
    var errorDescription: String? {
        switch self {
        case .calculationFailed(let error):
            return "ルート計算に失敗しました: \(error.localizedDescription)"
        case .locationPermissionDenied:
            return "位置情報へのアクセスが許可されていません"
        case .networkError:
            return "ネットワークエラーが発生しました"
        }
    }
}
```

#### 3. ViewModel
```swift
// ViewModels/NavigationViewModel.swift
import Foundation
import Combine
import SwiftUI

@MainActor
final class NavigationViewModel: ObservableObject {
    // MARK: - Published Properties
    
    @Published var currentRoute: Route?
    @Published var currentLocation: Location?
    @Published var isNavigating: Bool = false
    @Published var errorMessage: String?
    @Published var isLoading: Bool = false
    
    // MARK: - Private Properties
    
    private let navigationService: NavigationServiceProtocol
    private let syncService: SyncServiceProtocol
    private var cancellables = Set<AnyCancellable>()
    
    // MARK: - Initialization
    
    init(navigationService: NavigationServiceProtocol, 
         syncService: SyncServiceProtocol) {
        self.navigationService = navigationService
        self.syncService = syncService
        
        setupLocationUpdates()
        setupSyncListener()
    }
    
    // MARK: - Public Methods
    
    func startNavigation(to destination: Location) async {
        isLoading = true
        errorMessage = nil
        
        do {
            guard let currentLocation = currentLocation else {
                throw NavigationError.locationPermissionDenied
            }
            
            let route = try await navigationService.calculateRoute(
                from: currentLocation,
                to: destination
            )
            
            self.currentRoute = route
            self.isNavigating = true
            
            // Sync to iPhone
            try await syncService.syncRoute(route)
            
        } catch {
            errorMessage = error.localizedDescription
        }
        
        isLoading = false
    }
    
    func stopNavigation() {
        isNavigating = false
        currentRoute = nil
    }
    
    // MARK: - Private Methods
    
    private func setupLocationUpdates() {
        navigationService.startLocationUpdates()
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { [weak self] completion in
                    if case .failure(let error) = completion {
                        self?.errorMessage = error.localizedDescription
                    }
                },
                receiveValue: { [weak self] location in
                    self?.currentLocation = location
                }
            )
            .store(in: &cancellables)
    }
    
    private func setupSyncListener() {
        syncService.routePublisher
            .receive(on: DispatchQueue.main)
            .sink { [weak self] route in
                self?.currentRoute = route
                self?.isNavigating = true
            }
            .store(in: &cancellables)
    }
}
```

#### 4. View (SwiftUI)
```swift
// Views/NavigationView.swift
import SwiftUI
import MapKit

struct NavigationView: View {
    // MARK: - Properties
    
    @StateObject private var viewModel: NavigationViewModel
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 35.6812, longitude: 139.7671),
        span: MKCoordinateSpan(latitudeDelta: 0.05, longitudeDelta: 0.05)
    )
    
    // MARK: - Initialization
    
    init(viewModel: NavigationViewModel) {
        _viewModel = StateObject(wrappedValue: viewModel)
    }
    
    // MARK: - Body
    
    var body: some View {
        ZStack {
            // Map
            mapView
            
            // Route Info Overlay
            if let route = viewModel.currentRoute {
                VStack {
                    Spacer()
                    RouteInfoCard(route: route)
                        .padding()
                }
            }
            
            // Loading Overlay
            if viewModel.isLoading {
                LoadingView()
            }
        }
        .alert("エラー", isPresented: .constant(viewModel.errorMessage != nil)) {
            Button("OK") {
                viewModel.errorMessage = nil
            }
        } message: {
            if let errorMessage = viewModel.errorMessage {
                Text(errorMessage)
            }
        }
    }
    
    // MARK: - Private Views
    
    private var mapView: some View {
        Map(coordinateRegion: $region, annotationItems: annotations) { annotation in
            MapPin(coordinate: annotation.coordinate, tint: .blue)
        }
        .ignoresSafeArea()
    }
    
    private var annotations: [MapAnnotation] {
        guard let route = viewModel.currentRoute else { return [] }
        
        return [
            MapAnnotation(id: "start", coordinate: route.start.coordinate),
            MapAnnotation(id: "end", coordinate: route.end.coordinate),
        ]
    }
}

// MARK: - Supporting Types

struct MapAnnotation: Identifiable {
    let id: String
    let coordinate: CLLocationCoordinate2D
}

// MARK: - Preview

struct NavigationView_Previews: PreviewProvider {
    static var previews: some View {
        NavigationView(
            viewModel: NavigationViewModel(
                navigationService: MockNavigationService(),
                syncService: MockSyncService()
            )
        )
    }
}
```

## SwiftUI Best Practices

### 1. View Composition
```swift
// ✅ GOOD: Break down complex views
struct RouteInfoCard: View {
    let route: Route
    
    var body: some View {
        VStack(spacing: 8) {
            RouteDistanceRow(distance: route.distance)
            RouteDurationRow(duration: route.estimatedDuration)
            RouteActionsRow(onStart: {}, onStop: {})
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
    }
}

// ❌ BAD: Monolithic view
struct RouteInfoCard: View {
    let route: Route
    
    var body: some View {
        VStack(spacing: 8) {
            HStack {
                Image(systemName: "arrow.left.arrow.right")
                Text("Distance: \(route.formattedDistance)")
            }
            HStack {
                Image(systemName: "clock")
                Text("Duration: \(route.formattedDuration)")
            }
            HStack {
                Button("Start") { /* ... */ }
                Button("Stop") { /* ... */ }
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
    }
}
```

### 2. State Management

#### @State (View-local state)
```swift
struct CounterView: View {
    @State private var count = 0
    
    var body: some View {
        VStack {
            Text("Count: \(count)")
            Button("+") { count += 1 }
        }
    }
}
```

#### @StateObject (View-owned object)
```swift
struct NavigationView: View {
    @StateObject private var viewModel = NavigationViewModel()
    
    var body: some View {
        // View uses viewModel
    }
}
```

#### @ObservedObject (Passed object)
```swift
struct RouteDetailView: View {
    @ObservedObject var viewModel: RouteDetailViewModel
    
    var body: some View {
        // View observes passed viewModel
    }
}
```

#### @EnvironmentObject (Dependency injection)
```swift
@main
struct CompassWatchApp: App {
    @StateObject private var appState = AppState()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
        }
    }
}

struct ContentView: View {
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        // Access appState
    }
}
```

### 3. View Modifiers

#### Custom View Modifiers
```swift
struct CardStyle: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(Color(.systemBackground))
            .cornerRadius(12)
            .shadow(radius: 2)
    }
}

extension View {
    func cardStyle() -> some View {
        modifier(CardStyle())
    }
}

// Usage
Text("Hello")
    .cardStyle()
```

#### Conditional Modifiers
```swift
extension View {
    @ViewBuilder
    func `if`<Transform: View>(_ condition: Bool, transform: (Self) -> Transform) -> some View {
        if condition {
            transform(self)
        } else {
            self
        }
    }
}

// Usage
Text("Hello")
    .if(isHighlighted) { view in
        view.foregroundColor(.red)
    }
```

### 4. Performance Optimization

#### Lazy Stacks
```swift
// ✅ GOOD: Use LazyVStack for long lists
ScrollView {
    LazyVStack {
        ForEach(items) { item in
            ItemRow(item: item)
        }
    }
}

// ❌ BAD: Regular VStack loads all views
ScrollView {
    VStack {
        ForEach(items) { item in
            ItemRow(item: item)
        }
    }
}
```

#### Equatable Views
```swift
struct RouteRow: View, Equatable {
    let route: Route
    
    static func == (lhs: RouteRow, rhs: RouteRow) -> Bool {
        lhs.route.id == rhs.route.id
    }
    
    var body: some View {
        Text(route.name)
    }
}

// Usage with .equatable()
ForEach(routes) { route in
    RouteRow(route: route)
        .equatable()
}
```

## Concurrency (async/await)

### Modern Async Pattern
```swift
// ✅ GOOD: async/await
func fetchRoute() async throws -> Route {
    let url = URL(string: "https://api.compass.app/route")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(Route.self, from: data)
}

// Usage in view
Button("Fetch") {
    Task {
        do {
            let route = try await fetchRoute()
            self.route = route
        } catch {
            self.errorMessage = error.localizedDescription
        }
    }
}
```

### Actor for Thread Safety
```swift
actor RouteCache {
    private var cache: [String: Route] = [:]
    
    func getRoute(for id: String) -> Route? {
        cache[id]
    }
    
    func setRoute(_ route: Route, for id: String) {
        cache[id] = route
    }
    
    func clear() {
        cache.removeAll()
    }
}

// Usage
let cache = RouteCache()

Task {
    await cache.setRoute(route, for: "route-1")
    let cachedRoute = await cache.getRoute(for: "route-1")
}
```

## Error Handling

### Result Type
```swift
func calculateRoute() -> Result<Route, NavigationError> {
    do {
        let route = try performCalculation()
        return .success(route)
    } catch {
        return .failure(.calculationFailed(error))
    }
}

// Usage
let result = calculateRoute()
switch result {
case .success(let route):
    print("Route: \(route)")
case .failure(let error):
    print("Error: \(error.localizedDescription)")
}
```

### Throwing Functions
```swift
enum NetworkError: Error {
    case invalidURL
    case noData
    case decodingFailed
}

func fetchData() async throws -> Route {
    guard let url = URL(string: apiEndpoint) else {
        throw NetworkError.invalidURL
    }
    
    let (data, _) = try await URLSession.shared.data(from: url)
    
    guard !data.isEmpty else {
        throw NetworkError.noData
    }
    
    do {
        return try JSONDecoder().decode(Route.self, from: data)
    } catch {
        throw NetworkError.decodingFailed
    }
}
```

## Testing

### Unit Testing
```swift
import XCTest
@testable import CompassWatch

final class NavigationServiceTests: XCTestCase {
    var sut: NavigationService!
    var mockAPIClient: MockAPIClient!
    var mockLocationManager: MockLocationManager!
    
    override func setUp() {
        super.setUp()
        mockAPIClient = MockAPIClient()
        mockLocationManager = MockLocationManager()
        sut = NavigationService(
            locationManager: mockLocationManager,
            apiClient: mockAPIClient
        )
    }
    
    override func tearDown() {
        sut = nil
        mockAPIClient = nil
        mockLocationManager = nil
        super.tearDown()
    }
    
    func testCalculateRoute_Success() async throws {
        // Arrange
        let start = Location(latitude: 35.0, longitude: 139.0, name: "Start")
        let end = Location(latitude: 36.0, longitude: 140.0, name: "End")
        let expectedRoute = Route(id: "1", start: start, end: end, points: [], distance: 1000, estimatedDuration: 600)
        
        mockAPIClient.mockResponse = expectedRoute
        
        // Act
        let result = try await sut.calculateRoute(from: start, to: end)
        
        // Assert
        XCTAssertEqual(result.id, expectedRoute.id)
        XCTAssertEqual(result.distance, expectedRoute.distance)
    }
    
    func testCalculateRoute_Failure() async {
        // Arrange
        let start = Location(latitude: 35.0, longitude: 139.0, name: "Start")
        let end = Location(latitude: 36.0, longitude: 140.0, name: "End")
        
        mockAPIClient.shouldFail = true
        
        // Act & Assert
        do {
            _ = try await sut.calculateRoute(from: start, to: end)
            XCTFail("Should throw error")
        } catch {
            XCTAssertTrue(error is NavigationError)
        }
    }
}
```

### UI Testing
```swift
import XCTest

final class NavigationUITests: XCTestCase {
    var app: XCUIApplication!
    
    override func setUp() {
        super.setUp()
        continueAfterFailure = false
        app = XCUIApplication()
        app.launch()
    }
    
    func testNavigationFlow() {
        // Tap destination input
        let destinationField = app.textFields["destination-input"]
        XCTAssertTrue(destinationField.exists)
        destinationField.tap()
        destinationField.typeText("Tokyo Station")
        
        // Tap search button
        let searchButton = app.buttons["search-button"]
        searchButton.tap()
        
        // Wait for route to load
        let routeCard = app.otherElements["route-info-card"]
        XCTAssertTrue(routeCard.waitForExistence(timeout: 5))
        
        // Start navigation
        let startButton = app.buttons["start-navigation-button"]
        startButton.tap()
        
        // Verify navigation started
        let navigationIndicator = app.staticTexts["Navigating..."]
        XCTAssertTrue(navigationIndicator.exists)
    }
}
```

## WatchConnectivity Best Practices

### Setup WatchConnectivity
```swift
import WatchConnectivity

final class WatchConnectivityService: NSObject, ObservableObject {
    static let shared = WatchConnectivityService()
    
    @Published var isReachable = false
    @Published var receivedRoute: Route?
    
    private let session: WCSession
    
    private override init() {
        session = WCSession.default
        super.init()
        
        if WCSession.isSupported() {
            session.delegate = self
            session.activate()
        }
    }
    
    func sendRoute(_ route: Route) {
        guard session.isReachable else {
            // Use background transfer
            sendRouteInBackground(route)
            return
        }
        
        do {
            let data = try JSONEncoder().encode(route)
            let message: [String: Any] = ["route": data]
            
            session.sendMessage(message, replyHandler: { reply in
                print("Message sent successfully")
            }, errorHandler: { error in
                print("Error sending message: \(error)")
            })
        } catch {
            print("Encoding error: \(error)")
        }
    }
    
    private func sendRouteInBackground(_ route: Route) {
        do {
            let data = try JSONEncoder().encode(route)
            let userInfo: [String: Any] = ["route": data]
            
            session.transferUserInfo(userInfo)
        } catch {
            print("Encoding error: \(error)")
        }
    }
}

// MARK: - WCSessionDelegate

extension WatchConnectivityService: WCSessionDelegate {
    func session(_ session: WCSession, activationDidCompleteWith activationState: WCSessionActivationState, error: Error?) {
        DispatchQueue.main.async {
            self.isReachable = session.isReachable
        }
    }
    
    func sessionReachabilityDidChange(_ session: WCSession) {
        DispatchQueue.main.async {
            self.isReachable = session.isReachable
        }
    }
    
    func session(_ session: WCSession, didReceiveMessage message: [String : Any]) {
        guard let routeData = message["route"] as? Data else { return }
        
        do {
            let route = try JSONDecoder().decode(Route.self, from: routeData)
            DispatchQueue.main.async {
                self.receivedRoute = route
            }
        } catch {
            print("Decoding error: \(error)")
        }
    }
}
```

## Memory Management

### Weak References in Closures
```swift
// ✅ GOOD: Use [weak self]
class NavigationService {
    func startLocationUpdates() {
        locationManager.startUpdatingLocation { [weak self] location in
            self?.handleLocation(location)
        }
    }
}

// ❌ BAD: Strong reference cycle
class NavigationService {
    func startLocationUpdates() {
        locationManager.startUpdatingLocation { location in
            self.handleLocation(location) // Captures self strongly
        }
    }
}
```

### Unowned vs Weak
```swift
// Use [unowned self] when self will definitely exist
class NavigationService {
    private var timer: Timer?
    
    func startTimer() {
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [unowned self] _ in
            self.update() // Safe because timer is invalidated in deinit
        }
    }
    
    deinit {
        timer?.invalidate()
    }
}
```

## Accessibility

### VoiceOver Support
```swift
Text("Distance: 10.5 km")
    .accessibility(label: Text("Distance: ten point five kilometers"))

Button(action: startNavigation) {
    Image(systemName: "play.fill")
}
.accessibility(label: Text("Start navigation"))
.accessibility(hint: Text("Double tap to start navigation"))
```

### Dynamic Type Support
```swift
Text("Navigation")
    .font(.title)
    .dynamicTypeSize(...DynamicTypeSize.xxxLarge) // Limit maximum size
```

---

**Last Updated**: 2025-10-19  
**Auto-Loaded**: When working with Swift/watchOS files  
**Review Cycle**: Quarterly

