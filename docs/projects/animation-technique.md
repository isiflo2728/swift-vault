# AnimationTechnique

> A focused exploration of SwiftUI's animation system — implicit animations, explicit animations, transitions, and motion effects.

**Branch:** `animation-technique` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui/tree/animation-technique)

---

## What It Does

A sandbox app where each screen demonstrates a different animation primitive — designed to build muscle memory around SwiftUI motion before applying it in real apps.

---

## Concepts Covered

| Concept | Used For |
|---|---|
| [`animation(_:value:)`](../concepts/animations.md) | Implicit animations tied to state |
| [`withAnimation`](../concepts/animations.md) | Explicit animation blocks |
| `.transition()`  | Insertion/removal animations |
| `AnyTransition` | Custom and combined transitions |
| `Animation` modifiers | `.easeIn`, `.spring()`, `.interpolatingSpring()` |
| `GeometryEffect` | Custom geometry-based transforms |
| `DragGesture` | Linking motion to finger position |
| `.matchedGeometryEffect` | Hero animations between views |

---

## Key Code

### Implicit animation — reacts to state change automatically

```swift
struct ImplicitExample: View {
    @State private var animationAmount = 1.0

    var body: some View {
        Button("Tap Me") {
            animationAmount += 0.5
        }
        .scaleEffect(animationAmount)
        .animation(.easeInOut(duration: 0.3), value: animationAmount)
    }
}
```

### Explicit animation — you control exactly when it fires

```swift
Button("Tap Me") {
    withAnimation(.spring(response: 0.4, dampingFraction: 0.6)) {
        isFlipped.toggle()
    }
}
.rotation3DEffect(.degrees(isFlipped ? 180 : 0), axis: (x: 0, y: 1, z: 0))
```

### Asymmetric transition — different in vs out

```swift
if isVisible {
    Rectangle()
        .fill(.orange)
        .frame(width: 200, height: 200)
        .transition(.asymmetric(
            insertion: .scale,
            removal: .opacity
        ))
}
```

### Drag gesture linked to offset

```swift
@State private var offset = CGSize.zero

var body: some View {
    Circle()
        .offset(offset)
        .gesture(
            DragGesture()
                .onChanged { offset = $0.translation }
                .onEnded { _ in
                    withAnimation(.spring()) { offset = .zero }
                }
        )
}
```

---

## What I Learned

!!! success "Key Takeaways"
    - Implicit (`animation(_:value:)`) is for "this view always animates this property"
    - Explicit (`withAnimation {}`) is for "I decide when this block of state changes animates"
    - Transitions only fire on conditional insertion/removal from the view hierarchy — not on moves
    - `.spring()` almost always feels more natural than `.easeInOut` for interactive elements
    - `DragGesture` + `.onEnded` with a spring snap-back is the foundation of every swipe UI

!!! warning "Common Mistake"
    Forgetting `value:` in `animation(_:value:)`. Without it, **every** state change in the view triggers the animation — which causes all kinds of unintended motion.
