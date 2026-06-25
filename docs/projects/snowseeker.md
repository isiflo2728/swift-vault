# SnowSeeker

> An adaptive ski-resort browser that runs as a sidebar + detail layout on iPad and a push stack on iPhone — from a single `NavigationSplitView`. Search a list of resorts, favorite the ones you like, and drill into a detail screen that rearranges itself based on size class and Dynamic Type.

**Branch:** `snowseeker` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui/tree/snowseeker)

---

## What It Does

- `NavigationSplitView` master/detail — a resort `List` on the left, a `WelcomeView` placeholder on the right. On iPhone it automatically collapses to a normal push stack
- Each row shows a country flag (an asset named after the country), the resort name, and its run count, with a red `heart.fill` appended when the resort is a favorite
- `.searchable` filtering — type in the search bar to filter the list by name using `localizedStandardContains` (diacritic- and case-insensitive)
- A `Favorites` store — an `@Observable` class wrapping a `Set<String>` of resort ids, injected once into the environment and read from any view below
- `ResortView` detail screen — a decorative hero image, price/size/elevation/snow stats, a description, and a row of tappable facility icons that each open an explanatory `.alert`
- Adaptive stats layout — the detail view stacks its stat blocks vertically when the screen is compact *and* the user has cranked up Dynamic Type, otherwise lays them side by side
- Resort data decoded from `resorts.json` via the generic `Bundle.decode<T>()` extension first written in Moonshot

---

## Concepts Covered

| Concept | Used For |
|---|---|
| `NavigationSplitView` | One container that is a sidebar + detail on iPad and a push stack on iPhone — no per-device branching |
| `.searchable(text:prompt:)` | Adds the system search bar; a computed `filteredResorts` array does the actual filtering |
| `localizedStandardContains` | Case- and diacritic-insensitive substring match — the "right" way to filter user-facing text |
| `navigationDestination(for:)` | Type-safe push from a `NavigationLink(value:)` row to `ResortView` |
| `@Observable` (`Favorites`) | Shared favorites state that any row or detail view can read and mutate |
| `.environment(_:)` + `@Environment(Favorites.self)` | Inject the `Favorites` instance once at the split view; read it by type anywhere below |
| `@Environment(\.horizontalSizeClass)` | Detect compact vs regular width to choose the stat layout |
| `@Environment(\.dynamicTypeSize)` | Respect the user's text-size setting; combined with size class to avoid overflow |
| `List` with custom rows | Flag + name + run count + conditional favorite heart in an `HStack` |
| `Codable` + `Bundle.decode<T>()` | Decode `resorts.json` into `[Resort]` with the reusable generic extension |
| `Image(decorative:)` | Mark the hero photo as non-semantic so VoiceOver skips it |
| `.accessibilityLabel` | Give the icon-only facility buttons and the favorite heart a spoken identity |
| `.alert(_:isPresented:presenting:)` | Show facility detail using the optional `selectedFacility` as the presenting value |

---

## Key Code

### One container, two layouts — `NavigationSplitView`

The whole app is a single split view. On iPad it renders as a sidebar with a detail pane; on iPhone SwiftUI collapses it to a push stack automatically. `Favorites` is injected once at the bottom so every row and the detail view can reach it.

```swift
NavigationSplitView {
    List(filteredResorts) { resort in
        NavigationLink(value: resort) {
            HStack {
                Image(resort.country)               // asset named after the country
                    .resizable()
                    .scaledToFit()
                    .frame(width: 40, height: 25)
                    .clipShape(.rect(cornerRadius: 5))
                    .overlay(
                        RoundedRectangle(cornerRadius: 5)
                            .stroke(.black, lineWidth: 1)
                    )

                VStack(alignment: .leading) {
                    Text(resort.name).font(.headline)
                    Text("\(resort.runs) runs").foregroundStyle(.secondary)
                }

                if favorites.contains(resort) {
                    Spacer()
                    Image(systemName: "heart.fill")
                        .accessibilityLabel("This is a favorite resort")
                        .foregroundStyle(.red)
                }
            }
        }
    }
    .navigationTitle("Resorts")
    .navigationDestination(for: Resort.self) { resort in
        ResortView(resort: resort)
    }
    .searchable(text: $searchText, prompt: "Search for a resort")
} detail: {
    WelcomeView()
}
.environment(favorites)
```

### Search is just a filter

`.searchable` only supplies the bar and binds `searchText`. The list itself is driven by a computed property — change the text, the array recomputes, the `List` updates.

```swift
@State private var searchText = ""

var filteredResorts: [Resort] {
    if searchText.isEmpty {
        resorts
    } else {
        resorts.filter { $0.name.localizedStandardContains(searchText) }
    }
}
```

### `Favorites` — an `@Observable` shared through the environment

A tiny model that stores favorite resort **ids** (not whole `Resort` values) in a `Set` for O(1) membership and stable identity.

```swift
@Observable
class Favorites {
    private var resorts: Set<String>
    private let key = "favorites"

    init() {
        resorts = []          // load saved data here
    }

    func contains(_ resort: Resort) -> Bool { resorts.contains(resort.id) }

    func add(_ resort: Resort)    { resorts.insert(resort.id); save() }
    func remove(_ resort: Resort) { resorts.remove(resort.id); save() }

    func save() { /* persist `resorts` here */ }
}
```

Injected once with `.environment(favorites)` at the split view, then read by type wherever it's needed:

```swift
struct ResortView: View {
    @Environment(Favorites.self) var favorites
    // ...
}
```

!!! note "Persistence is stubbed"
    `init()` and `save()` are intentionally empty right now — favorites live only for the current session and reset on relaunch. The `key` property is already in place; wiring `UserDefaults` (encode the `Set` on `save()`, decode it in `init()`) is the next step. The `@Observable` + environment plumbing is the part this project was really about.

### A detail view that rearranges itself

`ResortView` reads two environment values and only stacks its stat blocks vertically when the screen is compact **and** the text is large — the one combination where two side-by-side blocks would overflow.

```swift
@Environment(\.horizontalSizeClass) var horizontalSizeClass
@Environment(\.dynamicTypeSize) var dynamicTypeSize

HStack {
    if horizontalSizeClass == .compact && dynamicTypeSize > .large {
        VStack(spacing: 10) { ResortDetailsView(resort: resort) }
        VStack(spacing: 10) { SkiDetailsView(resort: resort) }
    } else {
        ResortDetailsView(resort: resort)
        SkiDetailsView(resort: resort)
    }
}
```

### Facility icons → alerts, accessibly

Each facility maps a name to an SF Symbol and a description. The icon is a `View` that carries its own `.accessibilityLabel`, so an icon-only button still announces what it is. Tapping one sets the optional `selectedFacility`, which drives the alert.

```swift
struct Facility: Identifiable {
    let id = UUID()
    var name: String

    private let icons = ["Accommodation": "house", "Beginners": "1.circle",
                         "Cross-country": "map", "Eco-friendly": "leaf.arrow.circlepath",
                         "Family": "person.3"]

    var icon: some View {
        if let iconName = icons[name] {
            Image(systemName: iconName)
                .accessibilityLabel(name)        // icon-only, but VoiceOver still reads it
                .foregroundStyle(.secondary)
        } else {
            fatalError("Unknown facility type: \(name)")
        }
    }
}

// In ResortView — present using the optional itself:
.alert(selectedFacility?.name ?? "More information",
       isPresented: $showingFacility,
       presenting: selectedFacility) { _ in
} message: { facility in
    Text(facility.description)
}
```

### The model and its data

`Resort` is `Codable` for JSON decoding, `Hashable` so it can be a `navigationDestination` value, and `Identifiable` for the `List`. The generic `Bundle.decode<T>()` from Moonshot loads the bundled JSON.

```swift
struct Resort: Codable, Hashable, Identifiable {
    var id: String
    var name: String
    var country: String
    var description: String
    var imageCredit: String
    var price: Int
    var size: Int
    var snowDepth: Int
    var elevation: Int
    var runs: Int
    var facilities: [String]

    static let allResorts: [Resort] = Bundle.main.decode("resorts.json")
    static let example = allResorts[0]            // real data for previews

    var facilityTypes: [Facility] { facilities.map(Facility.init) }
}
```

---

## Questions I Had

Real questions that came up while building this. Written here so the confusion doesn't happen twice.

---

**Why `NavigationSplitView` instead of `NavigationStack`?**

`NavigationStack` is a single column — fine on iPhone, but on iPad it leaves the whole right half of the screen empty. `NavigationSplitView` declares a master *and* a detail pane. On a wide screen you get both side by side; on a compact screen (iPhone, or iPad in a narrow split) SwiftUI automatically collapses it back to a push stack. One declaration, correct on every device.

---

**Why store favorites as `Set<String>` of ids instead of `Set<Resort>`?**

Two reasons. A `Set` gives O(1) `contains`, which the row calls for every visible resort. And storing the **id** instead of the whole value means identity is stable and the set is trivial to persist later — it's just a set of strings. Storing whole `Resort` values would work (it's `Hashable`) but it's heavier and ties favorites to the exact decoded value.

---

**Why inject `Favorites` through `.environment` instead of passing it in each initializer?**

The favorite state is needed in two places that are far apart in the tree: the list rows (to show the heart) and `ResortView` (to toggle it). Threading it through every view's `init` would be noise. Injecting it once with `.environment(favorites)` lets any descendant pull it out with `@Environment(Favorites.self)` — no plumbing in between.

---

**Why `Image(decorative:)` for the resort photo but a labelled `Image` for the flag?**

The hero photo is pure decoration — it conveys nothing a VoiceOver user needs, so `Image(decorative:)` keeps it out of the accessibility tree. The flag and the favorite heart *do* carry meaning ("Austria", "this is a favorite resort"), so those get real `.accessibilityLabel`s. The rule is: hide what's decorative, label what informs.

---

**Why check both `horizontalSizeClass` and `dynamicTypeSize` for the layout?**

Two stat blocks fit side by side in almost every situation. The one case they *don't* is a narrow (compact) screen where the user has also bumped their text size up — then the side-by-side text overflows. Both conditions have to be true at once, so the check is `&&`, not `||`. It's a targeted fix for one real failure mode, not a blanket "small screen = vertical" rule.

---

**Why a `static let example` on the model?**

Previews need a real `Resort` to render `ResortView`, `ResortDetailsView`, and `SkiDetailsView`. `static let example = allResorts[0]` hands every `#Preview` actual decoded data without each one re-running a decode, so previews stay fast and consistent.

---

## The Click

**Adaptive UI from one container.** `NavigationSplitView` was the headline lesson: you describe *what* the two panes are, not *how* to lay them out per device. SwiftUI decides whether that's a sidebar+detail (iPad) or a push stack (iPhone) based on the available width — and switches live when the window resizes. No size checks, no separate iPad code path.

**`.searchable` is plumbing, not logic.** The search bar feels like a feature, but all it does is bind a `String`. The actual searching is a plain computed property that filters an array. Once that clicked, search stopped being special — it's the same "view is a function of state" loop as everything else: `searchText` changes → `filteredResorts` recomputes → the `List` redraws.

**`@Observable` belongs in the environment when it's shared and far-reaching.** `Favorites` is read in the list and written in the detail. Injecting it once and reading it by type — `@Environment(Favorites.self)` — is cleaner than passing it down, and because it's `@Observable`, every view that reads it re-renders the instant the set changes. Add a favorite in `ResortView`, the heart appears back in the list with zero extra wiring.
