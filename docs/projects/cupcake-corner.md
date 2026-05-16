# Cupcake Corner

> A multi-screen cupcake ordering app that encodes your order as JSON and sends it to a real REST endpoint via URLSession.

**Branch:** `cupcake-corner` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui/tree/cupcake-corner)

---

## What It Does

- Multi-step order form: cupcake type, quantity, special requests, delivery address
- Validates the address before allowing checkout
- Encodes the order as JSON and sends a POST request to a public test API
- Decodes the API response and shows a confirmation screen
- `AsyncImage` loads a remote cupcake photo

---

## Concepts Covered

| Concept | Used For |
|---|---|
| [`@Observable`](../concepts/state-data-flow.md) | `Order` class shared across all form screens |
| [`Codable`](../concepts/networking.md) | Encoding order to JSON for POST, decoding response |
| [`URLSession`](../concepts/networking.md) | Async network request with `async/await` |
| `async/await` | Non-blocking network call in a `Task {}` |
| `AsyncImage` | Loading a remote image with a loading placeholder |
| Form validation | `disabled()` modifier tied to address completeness |
| `.navigationDestination` | Pushing address and confirmation screens |
| `Stepper` | Adjusting cupcake quantity |

---

## Key Code

### Sending a POST request with async/await

```swift
func placeOrder() async {
    guard let encoded = try? JSONEncoder().encode(order) else {
        print("Failed to encode order"); return
    }

    let url = URL(string: "https://reqres.in/api/cupcakes")!
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")

    do {
        let (data, _) = try await URLSession.shared.upload(for: request, from: encoded)
        let decodedOrder = try JSONDecoder().decode(Order.self, from: data)
        confirmationMessage = "Your order for \(decodedOrder.quantity) cupcakes is on its way!"
        showingConfirmation = true
    } catch {
        print("Checkout failed: \(error.localizedDescription)")
    }
}
```

### Calling async code from a SwiftUI button

```swift
Button("Place Order") {
    Task {
        await placeOrder()
    }
}
```

### Validation with disabled modifier

```swift
Section {
    NavigationLink("Deliver to this address") {
        CheckoutView(order: order)
    }
}
.disabled(order.hasInvalidAddress)

// In Order model
var hasInvalidAddress: Bool {
    if streetAddress.isEmpty || city.isEmpty || state.isEmpty || zip.isEmpty {
        return true
    }
    return false
}
```

### AsyncImage with placeholder

```swift
AsyncImage(url: URL(string: "https://hws.dev/img/cupcakes@3x.jpg")) { image in
    image
        .resizable()
        .scaledToFit()
} placeholder: {
    ProgressView()
}
.frame(height: 233)
```

---

## What I Learned

!!! success "Key Takeaways"
    - `URLSession.shared.upload(for:from:)` handles multipart + JSON POSTs without any third-party library
    - `Task {}` inside a button action is the correct bridge from sync SwiftUI event handlers to async functions
    - Sharing an `@Observable` class across multiple screens via `NavigationStack` means all screens see the same live order object — no need to pass copies
    - `disabled()` on a `Section` disables everything inside it — clean single-line form validation

!!! warning "Error Handling"
    Network calls can fail silently if you `try?` everything. Use `do/catch` and surface errors to the user (an alert, inline message) — never swallow networking errors in production code.
