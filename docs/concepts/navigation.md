# Navigation

> Moving between views — pushing onto a stack, presenting sheets, and multi-level drill-down.

**Appears in:** WordScramble · Moonshot · iExpense · Cupcake Corner · BookWorm · HotProspects · SnowSeeker

---

## NavigationStack

The root container for all push-based navigation. Replace the old `NavigationView`.

```swift
NavigationStack {
    ContentView()
        .navigationTitle("My App")
}
```

### navigationDestination — type-safe push navigation

Instead of embedding destination views inside every `NavigationLink`, declare destinations at the stack level:

```swift
NavigationStack {
    List(missions) { mission in
        NavigationLink(value: mission) {
            MissionRow(mission: mission)
        }
    }
    .navigationDestination(for: Mission.self) { mission in
        MissionDetailView(mission: mission)
    }
    .navigationDestination(for: Astronaut.self) { astronaut in
        AstronautView(astronaut: astronaut)
    }
}
```

**Why this pattern is better:** The link only holds data — not a view. This is faster (no eager view creation) and separates navigation logic from link presentation.

---

## NavigationSplitView

A two- (or three-) column master/detail container. On a wide screen (iPad, landscape) it shows the columns side by side; on a compact screen (iPhone) SwiftUI **automatically collapses it to a `NavigationStack`** — same code, right layout per device.

```swift
NavigationSplitView {
    List(filteredResorts) { resort in
        NavigationLink(value: resort) {
            ResortRow(resort: resort)
        }
    }
    .navigationTitle("Resorts")
    .navigationDestination(for: Resort.self) { resort in
        ResortView(resort: resort)
    }
} detail: {
    WelcomeView()   // shown on the right when nothing is selected
}
```

- The **first** closure is the sidebar/primary column; the `detail:` closure is the secondary pane.
- `navigationDestination(for:)` still drives the push — on iPhone it pushes, on iPad it swaps the detail pane.
- The `detail:` view is the "nothing selected yet" placeholder a wide layout needs.

!!! tip "Steering the collapsed layout"
    Use `NavigationSplitView(preferredCompactColumn:)` to choose which column wins when collapsed, and `.navigationSplitViewStyle(.balanced)` to tune how a three-column layout distributes width.

---

## Searchable

`.searchable` adds the system search bar to a `NavigationStack` / `NavigationSplitView`. It only binds a `String` — *you* do the filtering, usually with a computed property.

```swift
@State private var searchText = ""

var filteredResorts: [Resort] {
    if searchText.isEmpty {
        resorts
    } else {
        resorts.filter { $0.name.localizedStandardContains(searchText) }
    }
}

// ...
List(filteredResorts) { resort in /* ... */ }
    .searchable(text: $searchText, prompt: "Search for a resort")
```

**`localizedStandardContains`** is the right matcher for user-facing text — it's case- *and* diacritic-insensitive, so "saint" matches "Saint" and "resume" matches "résumé". Don't hand-roll `lowercased().contains()`.

---

## Sheets

Modal presentation. The presented view sits *on top of* the current view, not inside the nav stack.

```swift
@State private var showingAddExpense = false

Button("Add") { showingAddExpense = true }
    .sheet(isPresented: $showingAddExpense) {
        AddExpenseView(expenses: expenses)
    }
```

**When to use:** Forms, pickers, confirmation flows that are logically separate from the main content.

---

## Dismiss

Let a presented sheet dismiss itself:

```swift
struct AddExpenseView: View {
    @Environment(\.dismiss) var dismiss

    var body: some View {
        Button("Save") {
            // save...
            dismiss()
        }
    }
}
```

---

## Toolbar items

Add buttons to the navigation bar:

```swift
.toolbar {
    ToolbarItem(placement: .topBarTrailing) {
        Button("Add", systemImage: "plus") {
            showingAddItem = true
        }
    }
    ToolbarItem(placement: .topBarLeading) {
        EditButton()
    }
}
```

---

## Multi-level drill-down (Moonshot pattern)

Three levels deep — grid → mission → astronaut — all inside one `NavigationStack`:

```swift
NavigationStack {
    MissionGridView()               // Level 1
        .navigationDestination(for: Mission.self) { mission in
            MissionDetailView()     // Level 2
                // Level 3 NavigationLinks for astronauts live inside MissionDetailView
        }
        .navigationDestination(for: Astronaut.self) { astronaut in
            AstronautView()         // Level 3
        }
}
```

**Key:** Both `navigationDestination` modifiers live on the *same* `NavigationStack` ancestor — SwiftUI resolves the right destination regardless of how deep the tap came from.

---

## TabView

A tab bar at the bottom of the screen. Each child is a separate tab.

```swift
TabView {
    ProspectsView(filter: .none)
        .tabItem {
            Label("Everyone", systemImage: "person.3")
        }
    ProspectsView(filter: .contacted)
        .tabItem {
            Label("Contacted", systemImage: "checkmark.circle")
        }
    MeView()
        .tabItem {
            Label("Me", systemImage: "person.crop.square")
        }
}
```

!!! tip "TabView wraps NavigationStack — not the other way around"
    `TabView` should be the outermost container. Each tab can have its own `NavigationStack` inside. If you put `TabView` *inside* a `NavigationStack`, the tab bar gets buried under the navigation chrome.

### Programmatic tab switching

Tag each tab and bind the selection:

```swift
@State private var selectedTab = "everyone"

TabView(selection: $selectedTab) {
    SomeView()
        .tabItem { Label("Everyone", systemImage: "person.3") }
        .tag("everyone")
    OtherView()
        .tabItem { Label("Me", systemImage: "person.crop.square") }
        .tag("me")
}
```

Setting `selectedTab = "me"` from anywhere switches to that tab.

---

## Quick Reference

| Need | Solution |
|---|---|
| Push a new screen | `NavigationLink(value:)` + `.navigationDestination(for:)` |
| Master/detail (adaptive iPad ↔ iPhone) | `NavigationSplitView { } detail: { }` |
| Filter a list with a search bar | `.searchable(text:)` + a computed `filtered` array |
| Present modal | `.sheet(isPresented:)` |
| Full-screen modal | `.fullScreenCover(isPresented:)` |
| Dismiss from inside sheet | `@Environment(\.dismiss)` |
| Nav bar buttons | `.toolbar { ToolbarItem(placement:) }` |
| Editable list | `.toolbar { EditButton() }` |
| Tab bar layout | `TabView` with `.tabItem` |
