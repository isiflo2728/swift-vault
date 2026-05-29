# Persistence

> Keeping data alive between app launches — from a simple key-value store to a full relational database.

**Appears in:** iExpense (UserDefaults) · BookWorm (SwiftData) · HotProspects (SwiftData + @AppStorage)

---

## UserDefaults

A key-value store backed by a plist file. Fast, simple, appropriate for settings and small data.

### Read and write simple types

```swift
// Write
UserDefaults.standard.set(42, forKey: "highScore")
UserDefaults.standard.set("dark", forKey: "theme")

// Read
let score = UserDefaults.standard.integer(forKey: "highScore")
let theme = UserDefaults.standard.string(forKey: "theme") ?? "light"
```

### Storing custom types — Codable + JSONEncoder

```swift
// Save
if let encoded = try? JSONEncoder().encode(myItems) {
    UserDefaults.standard.set(encoded, forKey: "SavedItems")
}

// Load
if let data = UserDefaults.standard.data(forKey: "SavedItems"),
   let decoded = try? JSONDecoder().decode([MyItem].self, from: data) {
    myItems = decoded
}
```

### Triggering a save automatically with didSet

```swift
@Observable
class Store {
    var items: [Item] = [] {
        didSet {
            if let encoded = try? JSONEncoder().encode(items) {
                UserDefaults.standard.set(encoded, forKey: "Items")
            }
        }
    }
}
```

**Limits of UserDefaults:**
- Not designed for large datasets (no queries, no indexing)
- No relationships between objects
- Not encrypted by default

---

## SwiftData

Apple's modern persistence framework (iOS 17+). Replaces Core Data with a dramatically simpler API.

### 1. Define a model

```swift
import SwiftData

@Model
class Book {
    var title: String
    var author: String
    var rating: Int

    init(title: String, author: String, rating: Int) {
        self.title = title
        self.author = author
        self.rating = rating
    }
}
```

`@Model` automatically makes `Book` observable, persistent, and queryable.

### 2. Configure the container

```swift
@main
struct BookwormApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Book.self)
    }
}
```

### 3. Insert new objects

```swift
@Environment(\.modelContext) var modelContext

let book = Book(title: "Dune", author: "Herbert", rating: 5)
modelContext.insert(book)
```

### 4. Query objects

```swift
// Live query — List updates automatically when data changes
@Query(sort: \Book.title) var books: [Book]

// With filter
@Query(filter: #Predicate<Book> { $0.rating > 3 },
       sort: \Book.title) var highRatedBooks: [Book]
```

### 5. Delete objects

```swift
modelContext.delete(book)
```

No explicit save needed — SwiftData autosaves.

---

## Choosing the Right Tool

| Scenario | Use |
|---|---|
| A single boolean / integer setting | `UserDefaults` direct |
| A small array of `Codable` structs | `UserDefaults` + `JSONEncoder` |
| A collection that needs sorting, filtering, or relationships | `SwiftData` |
| Per-user sensitive data | `Keychain` (not covered yet) |

**Rule of thumb:** If you're asking "do I need to query it?" → SwiftData. "Do I just need to remember a value?" → UserDefaults.
