# Layouts & Lists

> Arranging views on screen — from simple stacks to adaptive grids, dynamic data-driven lists, and scroll-driven geometry effects.

**Appears in:** WordScramble · Moonshot · iExpense · BookWorm · HotProspects · LayoutandGeometry

---

## Stack Layout

The building blocks. All three are transparent to the layout system — they arrange their children and report the combined size.

```swift
VStack(alignment: .leading, spacing: 12) { ... }   // vertical
HStack(alignment: .center, spacing: 8) { ... }     // horizontal
ZStack(alignment: .bottomTrailing) { ... }          // depth / overlay
```

**`Spacer()`** expands to fill available space along the stack axis — use it to push views to edges.

---

## List

A scrollable, interactive list of rows. Handles separators, swipe actions, and edit mode automatically.

```swift
List(usedWords, id: \.self) { word in
    HStack {
        Image(systemName: "\(word.count).circle")
        Text(word)
    }
}
```

### Dynamic rows with ForEach + onDelete

```swift
List {
    ForEach(items) { item in
        Text(item.name)
    }
    .onDelete(perform: delete)
    .onMove(perform: move)
}
```

**Why ForEach inside List instead of `List(items)`:** `List(items)` doesn't support `onDelete`/`onMove`. `ForEach` inside `List` does.

### Section grouping

```swift
List {
    Section("Personal") {
        ForEach(personalItems) { item in ItemRow(item: item) }
    }
    Section("Business") {
        ForEach(businessItems) { item in ItemRow(item: item) }
    }
}
```

---

## LazyVGrid

A grid that only renders visible cells. Essential for image-heavy grids.

```swift
let columns = [GridItem(.adaptive(minimum: 150))]

ScrollView {
    LazyVGrid(columns: columns, spacing: 20) {
        ForEach(missions) { mission in
            MissionCard(mission: mission)
        }
    }
    .padding(.horizontal)
}
```

### GridItem sizing options

| Style | Behavior |
|---|---|
| `.adaptive(minimum: 150)` | As many columns as fit, each at least 150pt |
| `.flexible()` | Equal-width columns that stretch |
| `.fixed(200)` | Exactly 200pt wide, always |

**Adaptive is almost always what you want** — it handles every screen size automatically.

---

## ScrollView

When you need scroll but not the interactivity of `List`:

```swift
ScrollView {
    VStack(spacing: 0) {
        ForEach(items) { item in
            ItemView(item: item)
            Divider()
        }
    }
}
```

---

## GeometryReader

Reads the proposed size and live position of its container at render time. The `proxy` parameter exposes `.size` and `.frame(in:)`.

```swift
GeometryReader { proxy in
    Image("banner")
        .frame(width: proxy.size.width * 0.9)
}
```

**Caution:** `GeometryReader` takes all available space and aligns children to the top-left. Wrap it tightly and always give it an explicit `.frame()` when used inside a `ForEach`.

### Coordinate Spaces

`proxy.frame(in:)` returns the view's rect in whichever coordinate space you request:

| Space | Returns position relative to... |
|---|---|
| `.global` | Top-left corner of the screen |
| `.local` | Top-left corner of the immediate parent container |
| `.named("X")` | Top-left of any ancestor marked `.coordinateSpace(name: "X")` |

```swift
// Tag an ancestor with a name
OuterView()
    .coordinateSpace(name: "Custom")

// Read position relative to it
GeometryReader { proxy in
    Text("Center")
        .onTapGesture {
            print(proxy.frame(in: .global).midY)       // from screen top
            print(proxy.frame(in: .named("Custom")).midY) // from OuterView top
            print(proxy.frame(in: .local).midY)        // from direct parent top
        }
}
```

### Scroll-Driven Effects

Inside a `ScrollView`, `proxy.frame(in: .global).minY` updates in real time as the user scrolls — making it a live sensor for each row's position on screen.

```swift
GeometryReader { fullView in
    ScrollView {
        ForEach(0..<50) { index in
            GeometryReader { proxy in
                let minY = proxy.frame(in: .global).minY

                Text("Row \(index)")
                    .rotation3DEffect(.degrees(minY - fullView.size.height / 2) / 5,
                                      axis: (x: 0, y: 1, z: 0))
                    .opacity(max(minY / 200, 0))
                    .scaleEffect(min(max(minY / 500, 0.5), 1))
            }
            .frame(height: 40)
        }
    }
}
```

The clamping pattern `min(max(value, floor), ceiling)` keeps derived values inside a safe range:
- `max(minY / 200, 0)` — opacity floors at 0 when the row scrolls above the screen (`minY` goes negative)
- `min(max(minY / 500, 0.5), 1)` — scale stays in the range `[0.5, 1.0]`

### `visualEffect` — The Modern Alternative

`visualEffect` gives you the same `proxy` as `GeometryReader` without disrupting layout. The transform is applied after layout completes — the view keeps its natural size and position:

```swift
Text("Number \(num)")
    .frame(width: 200, height: 200)
    .visualEffect { content, proxy in
        content.rotation3DEffect(
            .degrees(proxy.frame(in: .global).minX) / 8,
            axis: (x: 0, y: 1, z: 0)
        )
    }
```

Use `visualEffect` when you only need to *transform* a view. Use `GeometryReader` when you need to *size* a view based on available space.

### `scrollTargetLayout` and `scrollTargetBehavior`

Snap-to-card scrolling — but placement matters:

```swift
ScrollView(.horizontal) {
    HStack(spacing: 0) {
        ForEach(items) { item in CardView(item: item) }
    }
    .scrollTargetLayout()           // ← on the HStack, marks children as targets
}
.scrollTargetBehavior(.viewAligned) // ← on the ScrollView, controls snap behavior
```

Putting either modifier on the wrong view does nothing — no error, just no snapping.

---

## Swipe Actions

Per-row buttons that appear when the user swipes. Added directly to the row content, not to `List` or `ForEach`.

```swift
List(prospects) { prospect in
    ProspectRow(prospect: prospect)
        .swipeActions {
            Button("Delete", systemImage: "trash", role: .destructive) {
                modelContext.delete(prospect)
            }
        }
        .swipeActions(edge: .leading) {
            Button("Pin", systemImage: "pin") {
                prospect.isPinned.toggle()
            }
            .tint(.orange)
        }
}
```

- Default edge is `.trailing` (swipe left)
- Use `edge: .leading` for swipe right
- `role: .destructive` makes the button red automatically
- Use `.tint()` to set custom colors on non-destructive buttons

---

## Multi-select List

Enable row selection with a `Set` binding and `EditButton`:

```swift
@State private var selectedItems = Set<MyModel>()

List(items, selection: $selectedItems) { item in
    Text(item.name)
}
.toolbar {
    ToolbarItem(placement: .topBarLeading) { EditButton() }
}
```

When `EditButton` activates edit mode, checkboxes appear on each row. The selected items accumulate in `selectedItems`. Use `.safeAreaInset(edge: .bottom)` for a bulk-action button — not `.bottomBar` in a toolbar, which conflicts with `TabView`.

---

## Quick Reference

| Need | Use |
|---|---|
| Simple vertical/horizontal layout | `VStack` / `HStack` |
| Overlay views on top of each other | `ZStack` |
| Scrollable list with swipe/edit | `List` + `ForEach` |
| Per-row swipe buttons | `.swipeActions` |
| Multi-select rows | `List(selection:)` + `EditButton` |
| Image or card grid | `LazyVGrid` with `.adaptive` |
| Scroll without List chrome | `ScrollView` |
| Measure parent size | `GeometryReader` |
| Scroll-driven transforms (no layout disruption) | `visualEffect` |
| Snap-to-card scrolling | `scrollTargetLayout` + `scrollTargetBehavior` |
| Screen-relative position | `proxy.frame(in: .global)` |
| Parent-relative position | `proxy.frame(in: .local)` |
| Ancestor-relative position | `proxy.frame(in: .named(...))` |
