# Accessibility

iOS accessibility is built on two systems: **VoiceOver** (screen reader for blind/low-vision users) and **Voice Control** (speech commands for motor-impaired users). SwiftUI exposes the full UIAccessibility API through modifiers that chain onto any view.

---

## The Accessibility Tree

SwiftUI builds an accessibility tree from your view hierarchy. Every interactive or meaningful view gets an element in the tree. VoiceOver walks the tree and reads elements aloud; Voice Control overlays tap labels on visible elements so users can say "tap [label]" to activate them.

---

## Labels and Hints

```swift
Image("logo")
    .accessibilityLabel("App logo")           // what VoiceOver reads
    .accessibilityHint("Tap to open settings") // what will happen
```

- **Label**: the identity — what this thing *is*. Short noun phrase.
- **Hint**: the action — what happens when activated. Verb phrase. Don't duplicate the label.

---

## Hiding Views

```swift
// Decorative image — VoiceOver skips entirely
Image(decorative: "background-pattern")

// Any view — removed from the accessibility tree
Image("divider")
    .accessibilityHidden(true)
```

Only hide views that provide zero information value. Never hide interactive elements.

---

## Grouping Views

```swift
// .combine — children stay separate, read with a pause between them
VStack {
    Text("Score")
    Text("1000")
}
.accessibilityElement(children: .combine)

// .ignore — children are silenced, parent speaks for the whole group
VStack {
    Text("Score")
    Text("1000")
}
.accessibilityElement(children: .ignore)
.accessibilityLabel("Score: 1000")
```

Use `.combine` when child content stands on its own. Use `.ignore` when the combined meaning needs a custom rewrite.

---

## Traits

Traits signal semantic meaning. SwiftUI infers most traits automatically, but you can override them.

```swift
Image(pictures[index])
    .onTapGesture { ... }
    .accessibilityAddTraits(.isButton)    // signals it's interactive
    .accessibilityRemoveTraits(.isImage)  // removes the default image trait
```

Common traits: `.isButton`, `.isHeader`, `.isImage`, `.isLink`, `.isSelected`, `.playsSound`, `.startsMediaSession`

---

## Adjustable Controls

For controls with a numeric value — sliders, steppers, custom pickers — expose them as adjustable so VoiceOver users can swipe up/down to change the value.

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
```

---

## Voice Control Input Labels

Voice Control shows a label over every interactive element so users can say "tap [label]". If the visible text is hard to pronounce or ambiguous, provide alternatives.

```swift
Button("→") { ... }
    .accessibilityInputLabels(["Next", "Arrow", "Forward"])
// User can say "tap Next", "tap Arrow", or "tap Forward"
```

---

## Quick Reference

| Modifier | System | Purpose |
|---|---|---|
| `.accessibilityLabel(_:)` | VoiceOver | What it is |
| `.accessibilityHint(_:)` | VoiceOver | What it does |
| `.accessibilityHidden(true)` | Both | Remove from accessibility tree |
| `Image(decorative:)` | Both | Mark image as non-semantic |
| `.accessibilityElement(children: .combine)` | VoiceOver | Group children, read separately with pause |
| `.accessibilityElement(children: .ignore)` | VoiceOver | Group children under a single custom label |
| `.accessibilityAddTraits(_:)` | VoiceOver | Add semantic meaning |
| `.accessibilityRemoveTraits(_:)` | VoiceOver | Remove inferred traits |
| `.accessibilityValue(_:)` | VoiceOver | Announce current value of a control |
| `.accessibilityAdjustableAction` | VoiceOver | Enable swipe up/down to adjust value |
| `.accessibilityInputLabels(_:)` | Voice Control | Alternative spoken names for the element |

---

## Projects That Use This

- [AccessibilitySandbox](../projects/accessibility-sandbox.md) — dedicated accessibility learning project
