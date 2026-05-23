# Learning Timeline

> The progression from zero to a full iOS networking, persistence, iCloud sync, media processing, and MapKit stack — one project at a time.

---

<div class="timeline" markdown>

<div class="timeline-item" markdown>

## Project 1 — WordScramble
**Focus: Core SwiftUI primitives**

First real app. The goal was simple — build something functional — but it introduced the most foundational SwiftUI patterns: state driving UI, `List` with dynamic data, and bridging UIKit (UITextChecker) into a SwiftUI view.

**New concepts:** `@State`, `List`, `TextField`, `onSubmit`, `onAppear`, `Bundle` resource loading, `UITextChecker`

**The click:** Realizing that SwiftUI views are just functions of state — change `usedWords` and the `List` updates automatically. No reload needed.

</div>

<div class="timeline-item" markdown>

## Project 2 — AnimationTechnique
**Focus: Motion and interactivity**

A dedicated sandbox for SwiftUI's animation system. Built purely to develop muscle memory around `withAnimation`, `.animation(_:value:)`, and transitions before using them in real features.

**New concepts:** Implicit vs explicit animations, `withAnimation`, `.transition()`, `AnyTransition`, spring curves, `DragGesture`, `rotation3DEffect`

**The click:** The difference between implicit (always) and explicit (I decide) animation — and why implicit needs `value:` bound or it fires on everything.

</div>

<div class="timeline-item" markdown>

## Project 3 — iExpense
**Focus: Shared state and persistence**

First app with persistent data and a second screen. Introduced `@Observable` as the modern alternative to `@ObservableObject`/`@Published`, and `UserDefaults` + `Codable` as a persistence layer.

**New concepts:** `@Observable`, `didSet` persistence pattern, `UserDefaults`, `JSONEncoder`/`JSONDecoder`, `.sheet()`, `@Environment(\.dismiss)`, `onDelete`, `EditButton`

**The click:** Putting persistence logic inside the model (not the view) keeps views clean — they just read/write `expenses.items` without caring where it's saved.

</div>

<div class="timeline-item" markdown>

## Project 4 — Moonshot
**Focus: Complex data, grids, and multi-level navigation**

Most architecturally complex project so far. Required cross-referencing two JSON datasets (missions + astronauts) and navigating three levels deep. Introduced `LazyVGrid` and a reusable generic JSON decoder.

**New concepts:** `LazyVGrid`, `.adaptive` GridItem, `Bundle.decode<T>()` generic extension, `navigationDestination(for:)`, custom `Color` assets, multi-level `NavigationStack`, `ScrollView`

**The click:** The generic `Bundle.decode<T>()` extension — one function that handles any `Codable` type. Generics clicked as a concept here.

</div>

<div class="timeline-item" markdown>

## Project 5 — Cupcake Corner
**Focus: Networking and async/await**

First app to talk to the internet. Sending a real POST request with JSON, decoding the response, and handling it non-blocking with `async/await`. Also: multi-screen forms, validation, and `AsyncImage`.

**New concepts:** `URLSession.upload(for:from:)`, `URLRequest`, `async/await`, `Task {}`, `.task` modifier, `AsyncImage`, form validation with `disabled()`, shared `@Observable` across a nav stack

**The click:** `Task {}` inside a button action as the bridge from synchronous SwiftUI event handling to asynchronous network calls. Also: `.task` auto-cancels on view disappear — cleaner than `.onAppear`.

</div>

<div class="timeline-item" markdown>

## Project 6 — BookWorm
**Focus: SwiftData — real database persistence**

Replaced UserDefaults with SwiftData — Apple's modern database layer. Built a custom `@Binding`-driven star rating component and learned the full SwiftData lifecycle: `@Model`, `@Query`, `modelContext`.

**New concepts:** `SwiftData`, `@Model`, `@Query`, `modelContext.insert()`, `modelContext.delete()`, `.modelContainer()`, custom `@Binding` views, `Picker` with genre list

**The click:** `@Query` is live — no refresh logic needed. Add a book, the `List` updates instantly. SwiftData handles everything Core Data required 100 lines of boilerplate for.

</div>

<div class="timeline-item" markdown>

## Project 7 — SwiftDataProject
**Focus: SwiftData deep dive — relationships, dynamic queries, iCloud sync**

Pushed SwiftData further by introducing model relationships, runtime-configurable filtering and sorting, and iCloud sync via CloudKit. The key architectural lesson was separating query logic into a dedicated child view with a custom initializer so that filter/sort config can be injected from a parent.

**New concepts:** `@Relationship(deleteRule: .cascade)`, `#Predicate`, `SortDescriptor`, custom `@Query` `init` with `_users` underscore syntax, `@Bindable` for two-way bindings on `@Model` objects, programmatic `NavigationStack` with `path`, CloudKit capability setup, optional model properties for iCloud compatibility

**The click:** The `_users` underscore syntax — reaching into the property wrapper itself to construct the `@Query` dynamically at init time. Also: CloudKit sync requires zero networking code — just the right capability and optional properties on your models.

</div>

<div class="timeline-item" markdown>

## Project 8 — InstaFilter
**Focus: Core Image — photo processing pipeline**

First project to touch media processing. The main challenge was understanding that SwiftUI's `Image` is display-only — all the real work happens in Core Image's own type system before converting back to SwiftUI at the end. Also introduced `PhotosPicker`, `ShareLink`, and StoreKit review prompts.

**New concepts:** `CIFilter`, `CIContext`, `CIImage`, `UIImage` ↔ `CGImage` conversion, `PhotosPicker`, `kCIInputIntensityKey` / `kCIInputRadiusKey`, `ShareLink`, `SharePreview`, `@AppStorage`, `StoreKit` `requestReview`, `ContentUnavailableView`, `confirmationDialog`

**The click:** The image pipeline — `UIImage → CIImage → filter → CGImage → UIImage → SwiftUI Image`. Once that chain clicked, Core Image stopped feeling like magic and started feeling mechanical. Also: `CIContext` is expensive, so it lives at the struct level, not inside the processing function.

</div>

<div class="timeline-item" markdown>

## Project 9 — BucketList
**Focus: MapKit, location, and biometric authentication**

First project involving maps and real-world coordinates. Covered the full MapKit SwiftUI integration — rendering a map, placing custom annotation views, and converting screen taps to geographic coordinates. Added live Wikipedia data fetching and locked everything behind Face ID / Touch ID.

**New concepts:** `Map`, `MapReader`, `Annotation`, `CLLocationCoordinate2D`, `MKCoordinateRegion`, `MapCameraPosition`, `proxy.convert(_:from:)`, `LocalAuthentication`, `LAContext`, `sheet(item:)` with optional binding, `@Observable` MVVM via `ContentView` extension, `Comparable` conformance, custom `Equatable` `==`

**The click:** `sheet(item:)` with an optional — one property drives both whether the sheet is shown and what data it receives. SwiftUI unwraps the optional automatically inside the closure. Cleaner than a separate Bool + separate data property.

Also: long press gestures on `Map` annotation views conflict with the map's internal gesture recognizers on real devices (system gate timeout). Using `.onTapGesture` on the annotation view fixes it because the annotation intercepts the tap before the map sees it.

</div>

</div>

---

## Cumulative Skill Map

By the end of these 9 projects, the following areas are covered:

- [x] State management — `@State`, `@Binding`, `@Observable`, `@AppStorage`
- [x] Navigation — `NavigationStack`, multi-level drill-down, sheets
- [x] Layouts — stacks, `List`, `LazyVGrid`, `ScrollView`
- [x] Animations — implicit, explicit, transitions, gestures
- [x] Persistence — `UserDefaults`, `SwiftData`, `CloudKit`
- [x] Networking — `URLSession`, `async/await`, `Codable`
- [x] UIKit bridging — `UITextChecker`
- [x] Core Image — `CIFilter`, `CIContext`, photo processing pipeline
- [x] Location & Maps — `MapKit`, `CLLocationCoordinate2D`, `MapReader`, custom annotations
- [x] Biometric auth — `LocalAuthentication`, `LAContext`, Face ID / Touch ID
- [ ] Testing — unit tests, UI tests *(next)*
- [ ] Custom drawing — `Canvas`, `Path`, `Shape`
- [ ] Notifications — `UserNotifications`