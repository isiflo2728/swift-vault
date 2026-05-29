# Layouts & Lists

> Arranging views on screen — from simple stacks to adaptive grids and dynamic data-driven lists.

**Appears in:** WordScramble · Moonshot · iExpense · BookWorm · HotProspects

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

Read the parent's size and position at render time:

```swift
GeometryReader { geometry in
    Image("banner")
        .frame(width: geometry.size.width)
}
```

**Caution:** `GeometryReader` takes all available space and aligns children to the top-left. Wrap it tightly and only use it when you genuinely need the parent dimensions.

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
