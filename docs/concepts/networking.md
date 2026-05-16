# Networking

> Talking to the internet — making HTTP requests, handling async responses, and decoding JSON.

**Appears in:** Cupcake Corner

---

## The Stack

```
Codable     ← defines the shape of your data
URLRequest  ← describes the HTTP request
URLSession  ← executes the request
async/await ← keeps it non-blocking
```

---

## Codable

A protocol that makes a type encodable (→ JSON) and decodable (← JSON) automatically.

```swift
struct User: Codable {
    var id: Int
    var name: String
    var email: String
}
```

If your Swift property names match the JSON keys exactly (camelCase vs snake_case aside), no extra configuration needed.

### Handling snake_case JSON

```swift
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
// "first_name" → firstName automatically
```

---

## GET Request

```swift
func fetchUsers() async throws -> [User] {
    let url = URL(string: "https://reqres.in/api/users")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode([User].self, from: data)
}
```

Calling it from a view:

```swift
.task {
    do {
        users = try await fetchUsers()
    } catch {
        errorMessage = error.localizedDescription
    }
}
```

---

## POST Request — send JSON

```swift
func placeOrder(order: Order) async throws -> Order {
    let url = URL(string: "https://reqres.in/api/cupcakes")!
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    let encoded = try JSONEncoder().encode(order)

    let (data, _) = try await URLSession.shared.upload(for: request, from: encoded)
    return try JSONDecoder().decode(Order.self, from: data)
}
```

---

## async/await

Swift's native concurrency model. `await` suspends the current function until the result is ready — without blocking the main thread.

```swift
// async function — can be awaited
func loadData() async -> [Item] { ... }

// Task{} bridges sync SwiftUI context to async
Button("Load") {
    Task {
        let items = await loadData()
        self.items = items
    }
}

// .task modifier — auto-cancelled when view disappears
.task {
    await loadData()
}
```

**Prefer `.task` over `.onAppear { Task {} }`** — `.task` is automatically cancelled when the view leaves the screen, preventing stale data writes.

---

## AsyncImage

Load and display a remote image with a loading placeholder — no third-party library needed.

```swift
AsyncImage(url: URL(string: "https://example.com/photo.jpg")) { image in
    image
        .resizable()
        .scaledToFill()
} placeholder: {
    ProgressView()
}
.frame(width: 200, height: 200)
.clipped()
```

---

## Error Handling Pattern

```swift
@State private var isLoading = false
@State private var errorMessage: String?
@State private var data: [Item] = []

.task {
    isLoading = true
    defer { isLoading = false }
    do {
        data = try await fetchData()
    } catch {
        errorMessage = error.localizedDescription
    }
}
```

Always give the user feedback when a network call fails. Don't swallow errors with `try?` in user-facing code.

---

## Quick Reference

| Task | API |
|---|---|
| GET request | `URLSession.shared.data(from:)` |
| POST request | `URLSession.shared.upload(for:from:)` |
| Decode JSON response | `JSONDecoder().decode(T.self, from:)` |
| Encode to JSON | `JSONEncoder().encode(value)` |
| Load remote image | `AsyncImage(url:)` |
| Run async on view appear | `.task { await ... }` |
| Bridge button to async | `Button { Task { await ... } }` |
