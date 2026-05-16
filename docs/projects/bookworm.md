# BookWorm

> A book tracking app powered by SwiftData. Add books, rate them with a custom star component, write reviews, and delete ones you regret reading.

**Branch:** `bookworm` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui/tree/bookworm)

---

## What It Does

- Add books with title, author, genre (Picker), and rating
- Custom `RatingView` — a reusable star rating component built as a `@Binding`-driven view
- Write a text review for each book
- Browse all books in a `List`, color-coded by rating
- Swipe to delete
- All data persisted in SwiftData — survives app kills and reinstalls

---

## Concepts Covered

| Concept | Used For |
|---|---|
| [`SwiftData`](../concepts/persistence.md) | Database layer replacing Core Data |
| `@Model` | Marking `Book` as a SwiftData entity |
| `@Query` | Fetching all books reactively from the database |
| `ModelContainer` | Configuring the database in `@main` |
| `modelContext.insert()` | Saving a new book |
| `modelContext.delete()` | Deleting a book |
| Custom `@Binding` view | `RatingView` — reusable component with two-way binding |
| `Picker` with `ForEach` | Genre selection from a fixed list |

---

## Key Code

### SwiftData model

```swift
import SwiftData

@Model
class Book {
    var title: String
    var author: String
    var genre: String
    var review: String
    var rating: Int

    init(title: String, author: String, genre: String, review: String, rating: Int) {
        self.title = title
        self.author = author
        self.genre = genre
        self.review = review
        self.rating = rating
    }
}
```

### Wiring up the ModelContainer

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

### Querying and deleting

```swift
struct ContentView: View {
    @Environment(\.modelContext) var modelContext
    @Query(sort: \Book.title) var books: [Book]

    func deleteBooks(at offsets: IndexSet) {
        for offset in offsets {
            let book = books[offset]
            modelContext.delete(book)
        }
    }
}
```

### Custom star rating component

```swift
struct RatingView: View {
    @Binding var rating: Int
    var label = ""
    var maximumRating = 5
    var offImage: Image?
    var onImage = Image(systemName: "star.fill")
    var offColor = Color.gray
    var onColor = Color.yellow

    var body: some View {
        HStack {
            if !label.isEmpty { Text(label) }
            ForEach(1..<maximumRating + 1, id: \.self) { number in
                image(for: number)
                    .foregroundStyle(number > rating ? offColor : onColor)
                    .onTapGesture {
                        rating = number
                    }
            }
        }
    }

    func image(for number: Int) -> Image {
        number > rating ? (offImage ?? onImage) : onImage
    }
}
```

---

## What I Learned

!!! success "Key Takeaways"
    - SwiftData is dramatically less boilerplate than Core Data — `@Model` + `@Query` replaces NSManagedObject, fetch requests, and NSPersistentContainer
    - `@Query` is live — the `List` updates automatically when any book is added or deleted
    - A custom `@Binding` view is the right pattern for reusable input components (star rater, color picker, etc.)
    - `modelContext` comes from `@Environment` — no need to pass it explicitly through the view tree

!!! tip "When to use SwiftData vs UserDefaults"
    SwiftData if your data is relational, sortable, filterable, or grows over time. UserDefaults for simple scalar values and settings. The boundary is roughly: "do I need to query it?" → SwiftData. "Do I just need to remember a value?" → UserDefaults.
