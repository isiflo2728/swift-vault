# AccessibilitySandbox

> A focused SwiftUI sandbox for learning and experimenting with iOS accessibility APIs — VoiceOver, Voice Control, and the full UIAccessibility surface area.

**Branch:** `mastering-swiftui` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui/tree/mastering-swiftui)

---

## What It Does

A learning sandbox — not a shipped app, but a collection of focused experiments covering every major iOS accessibility modifier. Each snippet isolates one concept so the behavior is unambiguous when tested with VoiceOver or Voice Control enabled.

---

## Concepts Covered

| Concept | Used For |
|---|---|
| `.accessibilityLabel(_:)` | Custom text VoiceOver reads aloud instead of the default |
| `.accessibilityHint(_:)` | Describes what a tap or action will do |
| `.accessibilityHidden(true)` | Removes purely decorative views from the accessibility tree |
| `Image(decorative:)` | Declares an image as decorative so VoiceOver skips it |
| `.accessibilityElement(children: .combine)` | Groups related views and reads them together with a pause |
| `.accessibilityElement(children: .ignore)` | Collapses children into the parent; requires a custom label on the parent |
| `.accessibilityAddTraits(_:)` | Adds semantic meaning — `.isButton`, `.isImage`, `.isHeader` |
| `.accessibilityRemoveTraits(_:)` | Removes default traits (e.g. strip `.isImage` from a tappable image) |
| `.accessibilityValue(_:)` | Announces a readable current value for controls like steppers |
| `.accessibilityAdjustableAction` | Lets VoiceOver users swipe up/down to increment or decrement a value |
| `.accessibilityInputLabels(_:)` | Multiple spoken names for Voice Control — user can say any of them |

---

## Key Code

### Grouping views for VoiceOver

```swift
// .combine: reads "Your score is" pause "1000"
VStack {
    Text("Your score is")
    Text("1000").font(.title)
}
.accessibilityElement(children: .combine)

// .ignore: reads only the custom label — "Your score is 1000"
VStack {
    Text("Your score is")
    Text("1000").font(.title)
}
.accessibilityElement(children: .ignore)
.accessibilityLabel("Your score is 1000")
```

### Custom traits on a tappable image

```swift
Image(pictures[selectedPicture])
    .resizable()
    .scaledToFit()
    .onTapGesture { selectedPicture = Int.random(in: 0...3) }
    .accessibilityLabel(labels[selectedPicture])
    .accessibilityAddTraits(.isButton)    // announces as interactive
    .accessibilityRemoveTraits(.isImage)  // removes default image trait
```

### Adjustable stepper pattern

```swift
VStack {
    Text("Value \(value)")
    Button("Increment") { value += 1 }
    Button("Decrement") { value -= 1 }
}
.accessibilityElement()
.accessibilityLabel("Value")
.accessibilityValue(String(value))
.accessibilityAdjustableAction { direction in
    switch direction {
    case .increment: value += 1
    case .decrement: value -= 1
    default: break
    }
}
// VoiceOver: select the group, swipe up/down to adjust value
```

### Voice Control input labels

```swift
Button("Tap me") {
    print("Button tapped")
}
.accessibilityInputLabels(["Button", "Me", "Tap me"])
// Voice Control: user can say "tap Button", "tap Me", or "tap Tap me"
```

---

## What I Learned

!!! success "Key Takeaways"
    - `.combine` vs `.ignore` is the most important grouping decision: `.combine` keeps children readable but separate; `.ignore` collapses everything into one element that you label yourself
    - `accessibilityAdjustableAction` is the right pattern for anything that looks like a slider or stepper — VoiceOver users expect swipe-up/down to work on these
    - `.accessibilityInputLabels` matters for Voice Control — if a button's visible text is awkward to say, give it shorter spoken alternatives
    - `Image(decorative:)` vs `.accessibilityHidden(true)`: use `Image(decorative:)` when the image has no semantic value; use `.accessibilityHidden(true)` to suppress any view type

!!! tip "Tricky Part"
    `.accessibilityElement(children: .ignore)` silences children completely — if you forget to add `.accessibilityLabel` on the parent, VoiceOver reads nothing. Always pair `.ignore` with a label.

!!! warning "Voice Control vs VoiceOver"
    These are two separate systems. VoiceOver is for blind/low-vision users (screen reader, swipe navigation). Voice Control is for users with motor disabilities (speech commands, tap by name). `.accessibilityInputLabels` only affects Voice Control — it has no effect on VoiceOver.
