# Maps & Location

> First introduced in **BucketList**. Covers the SwiftUI MapKit integration, coordinate handling, and biometric authentication via LocalAuthentication.

---

## Map + MapReader

`Map` renders an interactive MapKit map inside a SwiftUI view. `MapReader` wraps it to provide a proxy for converting screen-space tap positions into geographic coordinates.

```swift
MapReader { proxy in
    Map(initialPosition: startPosition) {
        // map content
    }
    .onTapGesture { position in
        if let coordinate = proxy.convert(position, from: .local) {
            // coordinate is a CLLocationCoordinate2D
        }
    }
}
```

`proxy.convert(_:from:)` translates a `CGPoint` (where the user tapped on screen) into a `CLLocationCoordinate2D` (latitude/longitude). Without `MapReader` you have no way to do this conversion.

---

## MapCameraPosition

Controls the initial viewport of the map. The most common approach is `.region`:

```swift
let startPosition = MapCameraPosition.region(
    MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 56, longitude: -3),
        span: MKCoordinateSpan(latitudeDelta: 10, longitudeDelta: 10)
    )
)

Map(initialPosition: startPosition) { ... }
```

`latitudeDelta` and `longitudeDelta` control zoom — smaller values = more zoomed in.

---

## Annotation

Places a custom SwiftUI view at a specific coordinate on the map. Unlike `Marker` (which gives you a fixed pin), `Annotation` accepts any view:

```swift
Annotation(location.name, coordinate: location.coordinate) {
    Image(systemName: "star.circle")
        .resizable()
        .foregroundStyle(.red)
        .frame(width: 44, height: 44)
        .background(.white)
        .clipShape(.circle)
        .onTapGesture {
            selectedPlace = location
        }
}
```

!!! warning "Long press vs tap on annotations"
    `.onLongPressGesture` on annotation views conflicts with the map's internal gesture recognizers on real devices, causing a system gesture gate timeout. Use `.onTapGesture` instead — annotation views intercept taps before the map sees them, so there is no conflict.

---

## Coordinate computed property pattern

Rather than building `CLLocationCoordinate2D` inline everywhere, add it as a computed property on your model:

```swift
struct Location: Identifiable, Codable {
    var latitude: Double
    var longitude: Double

    var coordinate: CLLocationCoordinate2D {
        CLLocationCoordinate2D(latitude: latitude, longitude: longitude)
    }
}
```

This keeps the view code clean — `location.coordinate` instead of `CLLocationCoordinate2D(latitude: location.latitude, longitude: location.longitude)` everywhere.

---

## sheet(item:) with optional binding

A clean pattern for presenting a sheet based on a selected item:

```swift
@State private var selectedPlace: Location?

// present
viewModel.selectedPlace = location  // sheet appears

// in body
.sheet(item: $selectedPlace) { place in
    EditView(location: place) { newLocation in
        viewModel.update(location: newLocation)
    }
}
```

Setting the optional to a value presents the sheet. SwiftUI unwraps it automatically inside the closure — `place` is always a real value. Dismissing the sheet resets it to `nil`.

---

## LocalAuthentication — Face ID / Touch ID

```swift
import LocalAuthentication

func authenticate() {
    let context = LAContext()
    var error: NSError?

    if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
        let reason = "Please authenticate yourself to unlock your places."
        context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics,
            localizedReason: reason) { success, _ in
            if success {
                self.isUnlocked = true
            }
        }
    }
}
```

`canEvaluatePolicy` checks if biometrics are available. `evaluatePolicy` shows the Face ID / Touch ID prompt. The callback fires on a background thread — update UI state via `self.` on an `@Observable` class, which handles thread-hopping automatically.

!!! tip "Info.plist requirement"
    Add `NSFaceIDUsageDescription` to `Info.plist` with a reason string, or the app will crash on Face ID devices.

!!! tip "Testing on simulator"
    **Features → Face ID → Enrolled** to enable, then **Features → Face ID → Matching Face** to simulate a successful auth.

---

## Projects Using This

| Project | Usage |
|---|---|
| [BucketList](../projects/bucketlist.md) | Full MapKit integration, custom annotations, Wikipedia fetch, Face ID lock |