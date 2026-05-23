# BucketList

> A map-based location bookmarking app. Drop pins anywhere on the map, tap to name and describe them, browse nearby Wikipedia articles fetched live, and lock the whole thing behind Face ID or Touch ID.

**Branch:** `bucketlist` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui/tree/bucketlist)

---

## What It Does

- Shows an interactive map centered on Scotland via `MapKit`
- Tap anywhere on the map to drop a custom star annotation
- Tap an existing annotation to open an edit sheet — rename and describe the location
- Fetches nearby Wikipedia articles live from the Wikipedia API and displays them inside the edit sheet
- Locks all content behind Face ID / Touch ID via `LocalAuthentication` — a single button unlocks the app
- Persists nothing to disk at this stage — state lives in the ViewModel

---

## Concepts Covered

| Concept | Used For |
|---|---|
| `Map` + `MapReader` | Displaying the map and converting tap positions to coordinates |
| `Annotation` | Custom SwiftUI views pinned to map coordinates |
| `CLLocationCoordinate2D` / `MKCoordinateRegion` / `MapCameraPosition` | Setting and working with map positions |
| `@Observable` ViewModel | Separating state and logic from the view via a class extension |
| `sheet(item:)` with optional binding | Presenting an edit sheet only when a location is selected |
| `URLSession` + `async/await` | Fetching nearby Wikipedia articles asynchronously |
| `Codable` with nested structs | Decoding nested Wikipedia API JSON |
| `Comparable` conformance | Sorting `Page` results by title |
| `Equatable` with custom `==` | Comparing locations by `id` only, not every property |
| `Identifiable` | Driving `ForEach` and `sheet(item:)` |
| `LocalAuthentication` | Face ID / Touch ID via `LAContext` |
| `@escaping` closures | Passing a save callback from `ContentView` into `EditView` |
| `@Environment(\.dismiss)` | Dismissing the edit sheet from inside it |

---

## Architecture

The app uses MVVM with the `ViewModel` defined as an inner class via a `ContentView` extension — a pattern that keeps the model logic in a separate file without polluting the global namespace.

```
ContentView
  └── ViewModel (ContentView+ViewModel.swift)
        ├── locations: [Location]
        ├── selectedPlace: Location?
        ├── isUnlocked: Bool
        ├── addLocation(at:)
        ├── update(location:)
        └── authenticate()
```

`ContentView` holds a single `@State private var viewModel = ViewModel()` and reads everything from it. The view never manipulates the data arrays directly.

---

## Code Snippets

### The Location model

```swift
struct Location: Codable, Equatable, Identifiable {
    var id: UUID
    var name: String
    var description: String
    var latitude: Double
    var longitude: Double

    var coordinate: CLLocationCoordinate2D {
        CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
    }

    static let example = Location(id: UUID(), name: "Buckingham Palace",
        description: "Lit by over 40,000 lightbulbs.", latitude: 51.501, longitude: -0.141)

    static func ==(lhs: Location, rhs: Location) -> Bool {
        lhs.id == rhs.id
    }
}
```

!!! tip "Custom `==` for efficiency"
    Swift auto-synthesizes `Equatable` by comparing every property. Since each location already has a unique `id`, comparing only `id` is both correct and faster. The custom `==` opts out of the default behaviour.

---

### ViewModel — state + logic in one place

```swift
extension ContentView {
    @Observable
    class ViewModel {
        var locations = [Location]()
        var selectedPlace: Location?
        var isUnlocked = false

        func addLocation(at point: CLLocationCoordinate2D) {
            let newLocation = Location(id: UUID(), name: "New Location",
                description: "", latitude: point.latitude, longitude: point.longitude)
            locations.append(newLocation)
        }

        func update(location: Location) {
            guard let selectedPlace else { return }
            if let index = locations.firstIndex(of: selectedPlace) {
                locations[index] = location
            }
        }

        func authenticate() {
            let context = LAContext()
            var error: NSError?
            if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
                context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics,
                    localizedReason: "Please authenticate yourself to unlock your places.") { success, _ in
                    if success { self.isUnlocked = true }
                }
            }
        }
    }
}
```

---

### Map with tap-to-drop and sheet

```swift
MapReader { proxy in
    Map(initialPosition: startPosition) {
        ForEach(viewModel.locations) { location in
            Annotation(location.name, coordinate: location.coordinate) {
                Image(systemName: "star.circle")
                    .resizable()
                    .foregroundStyle(.red)
                    .frame(width: 44, height: 44)
                    .background(.white)
                    .clipShape(.circle)
                    .onTapGesture {
                        viewModel.selectedPlace = location
                    }
            }
        }
    }
    .onTapGesture { position in
        if let coordinate = proxy.convert(position, from: .local) {
            viewModel.addLocation(at: coordinate)
        }
    }
    .sheet(item: $viewModel.selectedPlace) { place in
        EditView(location: place) { newLocation in
            viewModel.update(location: newLocation)
        }
    }
}
```

!!! warning "Long press vs tap on map annotations"
    The book recommends `.onLongPressGesture` to distinguish annotation taps from map taps. In practice on real devices, the map's gesture system causes long press to time out (`Gesture: System gesture gate timed out`). Using `.onTapGesture` on the annotation view works reliably because the annotation intercepts the tap before the map sees it — no conflict.

---

### Wikipedia fetch

```swift
func fetchNearbyPlaces() async {
    let urlString = "https://en.wikipedia.org/w/api.php?ggscoord=\(location.latitude)%7C\(location.longitude)&action=query&prop=coordinates%7Cpageimages%7Cpageterms&colimit=50&piprop=thumbnail&pithumbsize=500&pilimit=50&wbptterms=description&generator=geosearch&ggsradius=10000&ggslimit=50&format=json"

    guard let url = URL(string: urlString) else { return }

    do {
        let (data, _) = try await URLSession.shared.data(from: url)
        let items = try JSONDecoder().decode(WikipediaResult.self, from: data)
        pages = items.query.pages.values.sorted()
        loadingState = .loaded
    } catch {
        loadingState = .failed
    }
}
```

The result is decoded into a custom `WikipediaResult` struct (named to avoid conflict with Swift's built-in `Result` type):

```swift
struct WikipediaResult: Codable {
    let query: Query

    struct Query: Codable {
        let pages: [Int: Page]
    }

    struct Page: Codable, Comparable {
        let pageid: Int
        let title: String
        let terms: [String: [String]]?

        static func <(lhs: Page, rhs: Page) -> Bool { lhs.title < rhs.title }

        var description: String {
            terms?["description"]?.first ?? "No further information"
        }
    }
}
```

!!! warning "Name clash with Swift's `Result`"
    Naming your decode struct `Result` conflicts with Swift's built-in `Result<Success, Failure>` type. The compiler resolves it unpredictably. Always use a unique name like `WikipediaResult` for API response models.

---

### Face ID / Touch ID lock screen

```swift
var body: some View {
    if viewModel.isUnlocked {
        MapReader { ... }
    } else {
        Button("Unlock Places", action: viewModel.authenticate)
            .padding()
            .background(.blue)
            .foregroundStyle(.white)
            .clipShape(.capsule)
    }
}
```

!!! tip "Testing Face ID on the simulator"
    Go to **Features → Face ID → Enrolled** on first run, then use **Features → Face ID → Matching Face** to simulate a successful auth. On a real device it just works.

---

### `sheet(item:)` — optional binding pattern

```swift
@State private var selectedPlace: Location?

.sheet(item: $selectedPlace) { place in
    EditView(location: place) { ... }
}
```

Setting `selectedPlace` to a `Location` presents the sheet. SwiftUI automatically unwraps the optional inside the closure — `place` is always a real `Location`, no optional handling needed. Dismissing the sheet sets `selectedPlace` back to `nil` automatically.

---

## Where I Needed Clarification

**Why did `.onLongPressGesture` time out on a real device?**

The map's internal gesture recognizers compete with custom gestures attached to annotation views. Long press hit the system gesture gate timeout before firing. Switching to `.onTapGesture` on the annotation view fixed it — annotation views intercept taps before the map's recognizer sees them, so there's no conflict.

---

**Why did naming my model `Result` cause strange compiler errors?**

Swift's standard library has a built-in `Result<Success, Failure>` type. Using the same name for a `Codable` struct causes the compiler to confuse the two — errors like "cannot call value of non-function type" and "generic parameter could not be inferred" appear on lines that look correct. The fix is a unique name: `WikipediaResult`.

---

**Why is `===` wrong for a custom Equatable implementation?**

`===` is the identity operator for reference types — it checks if two objects are the same instance in memory, not whether they're equal. For a `struct`, you want `==`. Defining `===` instead of `==` means your custom equality is never registered — Swift still uses the synthesized version that compares every property.

---

**How does `sheet(item:)` know when to show and hide?**

It watches the binding — when the bound optional goes from `nil` to a value, the sheet appears. When the sheet is dismissed (swipe down or `dismiss()`), SwiftUI sets the binding back to `nil`. The value inside the closure is the unwrapped optional, so you never need to guard-let inside the sheet.

---

## What I Learned

!!! success "Key Takeaways"
    - `MapReader` wraps `Map` and provides a proxy for converting screen points to map coordinates — the tap-to-drop pattern relies on `proxy.convert(_:from:)`
    - `sheet(item:)` with an optional is cleaner than a separate Bool — one property drives both the show/hide and the data passed into the sheet
    - `@Observable` MVVM via a `ContentView` extension keeps state and logic out of the view without polluting the global namespace
    - Long press gestures on `Map` annotation views conflict with the map's own recognizers on real devices — use tap instead
    - Never name a `Codable` model `Result` — it clashes with Swift's built-in `Result` type in ways that produce cryptic compiler errors
    - `===` vs `==`: always use `==` for value-type equality; `===` is identity for reference types and does nothing useful on a struct