# SwiftDataProject

> A user and job management app that goes deeper into SwiftData — relationships between models, dynamic filtering with `#Predicate`, runtime-configurable sorting, programmatic navigation, and iCloud sync via CloudKit.

**Branch:** `swiftdata-project` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui/tree/swiftdata-project)

---

## What It Does

- Add sample users and their associated jobs with a single tap
- Browse users sorted alphabetically with a live job count badge per row
- Swipe to delete individual users
- Toggle between showing all users or only upcoming ones via a toolbar button
- Dynamic filtering powered by `#Predicate` passed through a custom view initializer
- Dynamic sorting via `SortDescriptor` arrays
- Edit user details (name, city, join date) via a dedicated `EditUserView`
- All data synced to iCloud via CloudKit — persists across devices

---

## Concepts Covered

| Concept | Used For |
|---|---|
| `@Model` with relationships | `User` owns many `Job` objects via `@Relationship(deleteRule: .cascade)` |
| `#Predicate` | Dynamic filtering by join date |
| `SortDescriptor` | Runtime-configurable sort order |
| Custom `@Query` initializer | Passing filter/sort config into a child view via `init` |
| `@Bindable` | Two-way bindings on `@Model` objects in edit views |
| `NavigationStack` + `path` | Programmatic navigation — auto-navigate to new user on creation |
| `modelContext.delete(model:)` | Bulk-delete all records before inserting fresh sample data |
| CloudKit iCloud sync | Enabled via Signing & Capabilities — no extra code required |
| Optional properties + default values | Required by CloudKit sync — all `@Model` properties must be optional or defaulted |

---

## Key Code

### Models with a relationship

```swift
@Model
class User {
    var name: String = "Anonymous"
    var city: String = "Unknown"
    var joinDate: Date = Date.now
    @Relationship(deleteRule: .cascade) var jobs: [Job]? = [Job]()

    var unwrappedJobs: [Job] {
        jobs ?? []
    }

    init(name: String, city: String, joinDate: Date) {
        self.name = name
        self.city = city
        self.joinDate = joinDate
    }
}

@Model
class Job {
    var name: String = "None"
    var priority: Int = 1
    var owner: User?

    init(name: String, priority: Int) {
        self.name = name
        self.priority = priority
    }
}
```

### Dynamic filtering via custom initializer

```swift
struct UsersView: View {
    @Query var users: [User]

    init(minimumJoinDate: Date) {
        _users = Query(filter: #Predicate<User> { user in
            user.joinDate >= minimumJoinDate
        }, sort: \User.name)
    }

    var body: some View {
        List(users) { user in
            Text(user.name)
        }
    }
}
```

The `_users` underscore syntax reaches into the `@Query` property wrapper itself — not the array — so the query can be constructed at init time from external input.

### Toggling filters from a parent view

```swift
@State private var showingUpcomingOnly = false

// In body:
UsersView(minimumJoinDate: showingUpcomingOnly ? .now : .distantPast)

// Toolbar button:
Button(showingUpcomingOnly ? "Show Everyone" : "Show Upcoming") {
    showingUpcomingOnly.toggle()
}
```

### Editing a SwiftData object with @Bindable

```swift
struct EditUserView: View {
    @Bindable var user: User

    var body: some View {
        Form {
            TextField("Name", text: $user.name)
            TextField("City", text: $user.city)
            DatePicker("Join Date", selection: $user.joinDate)
        }
    }
}
```

`@Bindable` unlocks `$` syntax on `@Observable`/`@Model` objects, enabling two-way bindings without any extra wiring.

### Programmatic navigation to a new object

```swift
@State private var path = [User]()

// In toolbar:
Button("Add User", systemImage: "plus") {
    let user = User(name: "", city: "", joinDate: .now)
    modelContext.insert(user)
    path = [user]  // immediately navigate to edit screen
}
```

---

## iCloud Sync Setup

No code changes needed — just Xcode capability configuration:

1. **Signing & Capabilities** → add **iCloud** → check **CloudKit** → add container `iCloud.com.isidoro.swiftdataproject`
2. Add **Background Modes** capability → check **Remote notifications**
3. Make all `@Model` properties optional or give them default values (CloudKit requirement)
4. Test on a **real device** — the simulator does not sync reliably

---

## What I Learned

!!! success "Key Takeaways"
    - `#Predicate` is powerful but strict — only a subset of Swift expressions are supported, and `if/else` blocks are valid but can always be simplified to a single boolean expression with `&&`
    - The `_propertyName` underscore syntax gives you access to the property wrapper itself — essential for constructing `@Query` dynamically inside `init`
    - `@Bindable` is the missing piece when editing `@Model` objects — without it, `$` syntax won't compile on `@Observable` types
    - CloudKit sync requires all model properties to be optional or defaulted — this is a hard requirement, not a suggestion. Violating it silently breaks sync with no obvious error in the UI
    - Programmatic navigation with `path = [user]` lets you instantly deep-link to a newly created object — cleaner than presenting a sheet

!!! tip "SwiftData + CloudKit mental model"
    Think of CloudKit as a free backend that Apple manages. Once the capability is wired up and your models are CloudKit-compatible, SwiftData syncs automatically. You write zero networking, serialization, or conflict-resolution code — it just works across devices on the same iCloud account.
