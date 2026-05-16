# State & Data Flow

> How data moves through a SwiftUI app — from a single button tap to shared state across an entire screen stack.

**Appears in:** WordScramble · iExpense · Moonshot · Cupcake Corner · BookWorm

---

## The Core Idea

SwiftUI is declarative — views are a *function of state*. When state changes, SwiftUI recomputes the affected views. The question is always: **who owns the data?**

```
@State         → this view owns it, no one else needs it
@Binding       → this view uses it, parent owns it
@Observable    → shared class, multiple views can observe it
@Environment   → injected from above, not passed explicitly
@Query         → SwiftData live query, automatically updates List
```

---

## @State

Owned by a single view. Private. Triggers re-render on change.

```swift
struct CounterView: View {
    @State private var count = 0

    var body: some View {
        Button("Count: \(count)") {
            count += 1
        }
    }
}
```

**When to use:** Toggles, form field values, sheet presentation booleans, local UI state that nothing else cares about.

---

## @Binding

A reference to someone else's `@State`. Changes propagate back to the owner.

```swift
struct RatingView: View {
    @Binding var rating: Int   // owned by parent

    var body: some View {
        Stepper("Rating: \(rating)", value: $rating, in: 1...5)
    }
}

// Parent passes the binding with $
RatingView(rating: $book.rating)
```

**When to use:** Reusable child components that need to write back to a parent value (star raters, custom toggles, pickers).

---

## @Observable

An observation-ready class. Views that read any property automatically re-render when that property changes.

```swift
@Observable
class Order {
    var cupcakeType = 0
    var quantity = 3
    var specialRequestEnabled = false
}
```

Pass the instance to child views directly — no `@ObservedObject`, no `@EnvironmentObject`:

```swift
struct ContentView: View {
    var order = Order()

    var body: some View {
        OrderFormView(order: order)
    }
}
```

**When to use:** Any shared state that multiple views in a navigation stack need to read or write.

---

## @Environment

Values injected by the system (or by you) that flow down the view tree without explicit passing.

```swift
// Reading a system value
@Environment(\.colorScheme) var colorScheme

// Reading modelContext injected by .modelContainer()
@Environment(\.modelContext) var modelContext

// Injecting a custom value
.environment(myAppSettings)
```

**When to use:** Data that should be accessible anywhere below a certain point in the tree without cluttering every init signature.

---

## @Query (SwiftData)

Declarative fetch from SwiftData. Live — updates the view whenever the database changes.

```swift
@Query(sort: \Book.title) var books: [Book]
```

**When to use:** Displaying any SwiftData-persisted collection.

---

## Decision Tree

```
Does this state belong only to this one view?
  YES → @State
  NO  →
    Is the parent passing a writable value down?
      YES → @Binding
      NO  →
        Is this a SwiftData collection?
          YES → @Query
          NO  →
            Is this a shared class model?
              YES → @Observable (pass it directly)
              NO  → @Environment (system/global values)
```
