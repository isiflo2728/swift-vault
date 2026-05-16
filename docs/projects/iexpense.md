# iExpense

> A personal finance tracker that categorizes expenses as personal or business and persists all data across app launches with UserDefaults.

**Branch:** `iexpense` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui/tree/iexpense)

---

## What It Does

- Add expenses with a name, type (personal/business), and amount
- View expenses in a filtered list — color-coded by amount
- Swipe to delete individual items
- All data persists between launches via `UserDefaults` + `Codable`
- Add expense via a sheet presented modally

---

## Concepts Covered

| Concept | Used For |
|---|---|
| [`@State`](../concepts/state-data-flow.md) | Sheet presentation toggle, form fields |
| [`@Observable`](../concepts/state-data-flow.md) | `Expenses` model class shared across views |
| [`UserDefaults`](../concepts/persistence.md) | Saving and loading expenses |
| [`Codable`](../concepts/networking.md) | Encoding/decoding `ExpenseItem` to JSON |
| `.sheet(isPresented:)`  | Modal form for adding expenses |
| `onDelete(perform:)` | Swipe-to-delete on `List` rows |
| `NumberFormatter` | Currency formatting for amounts |
| `Picker` | Selecting expense type |

---

## Key Code

### Observable model with UserDefaults persistence

```swift
@Observable
class Expenses {
    var items: [ExpenseItem] = [] {
        didSet {
            if let encoded = try? JSONEncoder().encode(items) {
                UserDefaults.standard.set(encoded, forKey: "Items")
            }
        }
    }

    init() {
        if let savedItems = UserDefaults.standard.data(forKey: "Items"),
           let decodedItems = try? JSONDecoder().decode([ExpenseItem].self, from: savedItems) {
            items = decodedItems
        }
    }
}
```

### Codable expense model

```swift
struct ExpenseItem: Identifiable, Codable {
    var id = UUID()
    let name: String
    let type: String
    let amount: Double
}
```

### Swipe-to-delete from List

```swift
List {
    ForEach(expenses.items) { item in
        HStack {
            VStack(alignment: .leading) {
                Text(item.name).font(.headline)
                Text(item.type)
            }
            Spacer()
            Text(item.amount, format: .currency(code: "USD"))
                .foregroundStyle(item.amount < 10 ? .green : item.amount < 100 ? .orange : .red)
        }
    }
    .onDelete(perform: removeItems)
}

func removeItems(at offsets: IndexSet) {
    expenses.items.remove(atOffsets: offsets)
}
```

### Sheet presentation

```swift
.sheet(isPresented: $showingAddExpense) {
    AddView(expenses: expenses)
}
```

---

## What I Learned

!!! success "Key Takeaways"
    - `didSet` on a stored property is the cleanest place to trigger a side effect like saving to `UserDefaults`
    - `@Observable` replaces `@ObservableObject`/`@Published` — one annotation, no boilerplate
    - `JSONEncoder` + `UserDefaults` is plenty for small data sets; SwiftData is for anything relational
    - `.currency(code:)` format style handles locale-aware formatting with zero extra code

!!! tip "Architecture Insight"
    Keeping the persistence logic inside the model class (not the view) means views stay declarative — they just read and write to `expenses.items` without knowing anything about UserDefaults.
