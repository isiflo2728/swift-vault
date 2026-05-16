# Animations

> Making interfaces feel alive — implicit animations, explicit animation blocks, view transitions, and gesture-driven motion.

**Appears in:** AnimationTechnique · WordScramble (row insertion)

---

## The Two Types

SwiftUI has two animation systems that compose together:

| Type | Modifier | When it fires |
|---|---|---|
| **Implicit** | `.animation(_:value:)` | Automatically, whenever `value` changes |
| **Explicit** | `withAnimation {}` | Only the state changes inside the closure |

---

## Implicit Animations

Attach to a view. Every time the bound value changes, SwiftUI animates all animatable properties of that view.

```swift
Button("Tap") { animationAmount += 0.5 }
    .scaleEffect(animationAmount)
    .animation(.easeInOut(duration: 0.3), value: animationAmount)
```

**Always bind to a specific value.** Without `value:`, every state change anywhere in the view triggers the animation.

---

## Explicit Animations

Wrap state changes in `withAnimation`. Only those changes animate — everything else stays instant.

```swift
withAnimation(.spring(response: 0.4, dampingFraction: 0.6)) {
    isFlipped.toggle()
    cardOffset = 0
}
```

**When to prefer explicit:** When you want to animate multiple state changes together, or when you want animation to fire only on a specific user action (not on every state change).

---

## Animation Curves

```swift
.linear(duration:)                    // constant speed
.easeIn / .easeOut / .easeInOut       // standard curves
.spring()                             // bouncy, natural feel
.spring(response:dampingFraction:)    // tunable spring
.interpolatingSpring(stiffness:damping:)
```

**Spring is almost always the right choice for interactive elements.** It feels physical. Use `response` (how fast) and `dampingFraction` (how bouncy — 1.0 = no bounce, 0.5 = very bouncy).

---

## Transitions

Control how a view enters and exits the view hierarchy. Only fires when the view is conditionally inserted or removed.

```swift
if isVisible {
    Rectangle()
        .transition(.scale)
}
```

### Built-in transitions

```swift
.opacity       // fade in/out
.scale         // grow/shrink from center
.slide         // slide in from leading edge
.move(edge:)   // slide from a specific edge
.push(from:)   // pushes existing content
```

### Combining transitions

```swift
.transition(.scale.combined(with: .opacity))
```

### Asymmetric — different in and out

```swift
.transition(.asymmetric(
    insertion: .scale,
    removal: .opacity.combined(with: .move(edge: .trailing))
))
```

---

## Gesture-Driven Animation

Link motion to a `DragGesture` so the view follows the finger:

```swift
@State private var offset = CGSize.zero

Circle()
    .offset(offset)
    .gesture(
        DragGesture()
            .onChanged { gesture in
                offset = gesture.translation
            }
            .onEnded { _ in
                withAnimation(.spring()) {
                    offset = .zero     // snap back
                }
            }
    )
```

---

## 3D Rotation

```swift
.rotation3DEffect(
    .degrees(isFlipped ? 180 : 0),
    axis: (x: 0, y: 1, z: 0)
)
```

---

## withAnimation in Lists

`List` + `ForEach` animates row insertion and removal automatically when wrapped in `withAnimation`:

```swift
withAnimation {
    usedWords.insert(newWord, at: 0)
}
```

---

## Common Mistakes

!!! danger "Animating without binding a value"
    ```swift
    // BAD — animates on EVERY state change in the view
    .animation(.easeInOut)

    // GOOD — only animates when animationAmount changes
    .animation(.easeInOut, value: animationAmount)
    ```

!!! warning "Transition without conditional"
    Transitions only fire on view *insertion/removal*. If the view is always in the hierarchy (just hidden), transition has no effect — you want `opacity` animation instead.
