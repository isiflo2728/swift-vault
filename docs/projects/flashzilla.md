# FlashZilla

> A timed flashcard app with swipe-to-dismiss cards and full accessibility support. Build a deck of prompt/answer cards, swipe right to mark correct or left to mark wrong, and race against a 100-second countdown.

**Branch:** `FlashZilla` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui/tree/FlashZilla)

---

## What It Does

- Displays a deck of flashcards stacked on screen — swipe right to dismiss as correct, swipe left as wrong
- Cards tilt and fade based on drag direction, giving immediate directional feedback
- A 100-second countdown timer runs at the top; when it reaches zero, all card interaction locks out
- Tap any card to flip and reveal the answer
- A `+` button in the top-right opens the edit screen to add or remove cards from the deck
- When the deck is cleared, a Restart button reappears — resetting the deck and the timer
- Timer automatically pauses when the app is sent to the background and resumes when it returns
- Accessibility-aware: shows explicit correct/wrong buttons for users with Differentiate Without Color or VoiceOver enabled

---

## Concepts Covered

| Concept | Used For |
|---|---|
| `DragGesture` | Swipe-to-dismiss cards with live drag tracking |
| `.rotationEffect` + `.offset` | Tilting and sliding the card proportionally to drag width |
| `.stacked(at:in:)` custom `View` extension | Layering cards in an offset stack |
| `allowsHitTesting` | Disabling all card interaction when the timer expires |
| `Timer.publish(every:on:in:).autoconnect()` | 1-second countdown tick as a Combine publisher |
| `internal import Combine` | Required to access `Timer`'s publisher API |
| `.onReceive` | Subscribing the view to the timer publisher |
| `scenePhase` + `.onChange(of:)` | Pausing the timer on background, resuming on foreground |
| `UserDefaults` + `Codable` | Persisting the card deck across launches |
| `accessibilityDifferentiateWithoutColor` | Showing manual correct/wrong buttons as a color-free swipe alternative |
| `accessibilityVoiceOverEnabled` | Displaying answer text directly rather than requiring a tap |
| Sequenced gestures | Chaining `LongPressGesture` before `DragGesture` with `.sequenced(before:)` |
| Simultaneous gestures | Running parent and child gestures at the same time with `.simultaneousGesture` |
| `.contentShape(.rect)` | Making `Spacer`s inside a `VStack` tappable |
| `.accessibilityHidden` | Hiding all cards below the top from VoiceOver |
| `.accessibilityAddTraits(.isButton)` | Telling VoiceOver that a card is interactive |

---

## Architecture

Three views wired together through `ContentView`:

```
ContentView
├── ZStack
│   ├── background Image (decorative, ignoresSafeArea)
│   └── VStack
│       ├── Timer label (Text — "Time Remaining: N")
│       └── ZStack  ← card area
│           ├── VStack → Edit button (top-right HStack + Spacer)
│           ├── Accessibility buttons (bottom HStack, conditional)
│           └── ForEach → CardView × cards.count  (stacked, top card interactive)
└── Restart Button (shown when cards.isEmpty)
    .sheet → EditCardsView
```

`CardView` owns the `DragGesture` and calls back via a `removal` closure when the swipe threshold is crossed. `ContentView` handles the actual removal and side effects (pausing, restarting).

`EditCardsView` reads and writes the `UserDefaults` card store directly. When it dismisses, `ContentView`'s `onDismiss` callback fires `resetCards()` to reload the deck.

---

## Code Snippets

### The Card model

```swift
struct Card: Codable {
    var prompt: String
    var answer: String

    static let example = Card(prompt: "Who played the 13th Doctor?", answer: "Jodie Whittaker")
}
```

`Codable` is all that's needed — no `@Model`, no SwiftData. The full deck is encoded to `Data` and stored in `UserDefaults` as a single key.

---

### Custom `.stacked(at:in:)` View extension

```swift
extension View {
    func stacked(at position: Int, in total: Int) -> some View {
        let offset = Double(total - position)
        return self.offset(y: offset * 10)
    }
}
```

Applied inside the `ForEach`:

```swift
ForEach(0..<cards.count, id: \.self) { index in
    CardView(card: cards[index]) {
        withAnimation { removeCard(at: index) }
    }
    .stacked(at: index, in: cards.count)
    .allowsTightening(index == cards.count - 1)
    .accessibilityHidden(index < cards.count - 1)
}
```

Cards at lower indices are pushed further down. The top card (highest index) gets `allowsHitTesting(true)`; everything below it is hidden from VoiceOver and untouchable.

---

### CardView — DragGesture with rotation and opacity

```swift
@State private var offset = CGSize.zero

var body: some View {
    ZStack {
        RoundedRectangle(cornerRadius: 25)
            .fill(
                accessibilityDifferentiateWithoutColor
                    ? .white
                    : .white.opacity(1 - Double(abs(offset.width / 50)))
            )
            .background(
                accessibilityDifferentiateWithoutColor
                    ? nil
                    : RoundedRectangle(cornerRadius: 25)
                        .fill(offset.width > 0 ? .green : .red)
            )
            .shadow(radius: 10)
        // ... text content
    }
    .rotationEffect(.degrees(offset.width / 5.0))
    .offset(x: offset.width)
    .gesture(
        DragGesture()
            .onChanged { gesture in offset = gesture.translation }
            .onEnded { _ in
                if abs(offset.width) > 100 {
                    removal?()
                } else {
                    offset = .zero
                }
            }
    )
    .animation(.bouncy, value: offset)
}
```

!!! tip "The threshold number"
    `abs(offset.width) > 100` — if the card was dragged more than 100 points horizontally it's dismissed, otherwise it snaps back to center. The `.bouncy` animation on `offset` handles the snap-back automatically because changing `offset` back to `.zero` triggers it.

The background shows green or red based on `offset.width > 0` — but only when `accessibilityDifferentiateWithoutColor` is false. Color-blind users get the manual button row instead.

---

### Locking interaction when time expires

```swift
ZStack {
    // card stack
}
.allowsHitTesting(timeRemaining > 0)
```

One modifier on the outer `ZStack` disables all gestures and taps inside the entire card area the moment the timer hits zero. No per-card logic needed.

---

### Timer publisher with Combine

```swift
import Foundation
internal import Combine

let timer = Timer.publish(every: 1, on: .main, in: .common).autoconnect()

// In body:
.onReceive(timer) { time in
    guard isActive else { return }
    if timeRemaining > 0 {
        timeRemaining -= 1
    }
}
```

!!! note "`internal import Combine`"
    `Timer.publish` lives in Combine, but the module isn't always automatically imported even in SwiftUI files. Adding `internal import Combine` makes it explicit without leaking the import to other modules.

`isActive` is a separate Bool that tracks whether the timer should actually count down — it's set to `false` when the app is backgrounded or all cards are gone, and back to `true` when the app returns and cards still exist.

---

### Pausing on background with scenePhase

```swift
@Environment(\.scenePhase) var scenePhase

.onChange(of: scenePhase) {
    if scenePhase == .active {
        if cards.isEmpty == false {
            isActive = true
        }
    } else {
        isActive = false
    }
}
```

The timer publisher keeps firing in the background — the `guard isActive else { return }` inside `.onReceive` is what prevents the count from actually dropping. Setting `isActive = false` on any non-active phase covers both `.inactive` and `.background`.

---

### UserDefaults persistence

```swift
func loadData() {
    if let data = UserDefaults.standard.data(forKey: "Cards") {
        if let decoded = try? JSONDecoder().decode([Card].self, from: data) {
            cards = decoded
        }
    }
}

func saveData() {
    if let data = try? JSONEncoder().encode(cards) {
        UserDefaults.standard.set(data, forKey: "Cards")
    }
}
```

`loadData` runs on `onAppear` in both `ContentView` and `EditCardsView`. `saveData` runs in `EditCardsView` immediately after any add or delete — no explicit save button.

---

### Accessibility: manual correct/wrong buttons

```swift
@Environment(\.accessibilityDifferentiateWithoutColor) var accessibilityDifferentiateWithoutColor
@Environment(\.accessibilityVoiceOverEnabled) var accessibilityWithVoiceOverEnabled

if accessibilityDifferentiateWithoutColor || accessibilityWithVoiceOverEnabled {
    VStack {
        Spacer()
        HStack {
            Button { withAnimation { removeCard(at: cards.count - 1) } } label: {
                Image(systemName: "xmark.circle")
                    .padding()
                    .background(.black.opacity(0.75))
                    .clipShape(.circle)
            }
            .accessibilityLabel("Wrong")
            .accessibilityHint("Mark your answer as being incorrect")

            Spacer()

            Button { withAnimation { removeCard(at: cards.count - 1) } } label: {
                Image(systemName: "checkmark.circle")
                    .padding()
                    .background(.black.opacity(0.75))
                    .clipShape(.circle)
            }
            .accessibilityLabel("Correct")
            .accessibilityHint("Mark your answer as being correct")
        }
        .foregroundStyle(.white)
        .font(.largeTitle)
        .padding()
    }
}
```

!!! warning "Both conditions trigger the same buttons"
    VoiceOver users can't reliably perform a precise drag gesture across a card, so the manual buttons activate for them too — even though the color differentiation setting is off. Both accessibility environments get the same fallback.

---

## Where I Needed Clarification

**Why does `.animation(.bouncy, value: offset)` snap the card back but not affect the removal animation?**

`.animation(_:value:)` only fires when `offset` changes. The snap-back happens when `offset` is set to `.zero` inside `.onEnded`. The removal animation is triggered separately by `withAnimation` wrapping the `removeCard` call in `ContentView` — that's a different animation scope entirely. Both can coexist because they animate different things.

---

**Why is the card stack interactive only at the top card?**

Two modifiers work together: `.allowsTightening(index == cards.count - 1)` restricts hit-testing to the top card, and `.accessibilityHidden(index < cards.count - 1)` hides all lower cards from VoiceOver. Visually the cards are all visible — but only the top one responds to input.

---

**Why does the timer keep counting even when the app is in the background?**

`Timer.publish` doesn't know about your app's state — it fires on the run loop regardless. The `isActive` flag inside `.onReceive` is what gates the decrement. Without it, 30 seconds in the background would cost 30 seconds of game time. The `scenePhase` `.onChange` sets `isActive = false` the moment the app leaves the foreground.

---

**Why use `internal import Combine` instead of just `import Combine`?**

`internal` means the import doesn't escape the current file — other files in the module don't inherit it. Since Combine is only needed here for `Timer.publish`, keeping the import internal is cleaner. On some Xcode versions the compiler requires an explicit `import Combine` for `Timer.publish` to resolve even in a SwiftUI file — this is the most scoped way to satisfy that.

---

**Why does `removeCard(at:)` guard against `index >= 0`?**

The `ForEach` callback captures the card's current index, but by the time it fires the deck may have changed — especially if two swipes happen in rapid succession. Guarding against a negative index prevents a crash if the array shrinks out from under the index before the removal runs.

---

## What I Learned

!!! success "Key Takeaways"
    - `DragGesture` gives you `gesture.translation` as a `CGSize` — use `.width` for horizontal swipe detection. The card tilts with `offset.width / 5.0` and fades with `abs(offset.width / 50)`. Both are simple math on the same single value
    - `Timer.publish` is a Combine publisher that fires on the run loop — it doesn't stop on its own. Always pair it with an `isActive` guard and `scenePhase` tracking or the timer bleeds through background
    - `allowsHitTesting(false)` on a container disables all interaction inside it — one modifier, no per-child work needed. Use it to lock the card area the moment the timer expires
    - The `stacked(at:in:)` extension pattern — a small `View` extension that encapsulates a layout modifier. Clean call site, reusable, no extra view layer
    - Accessibility environment values (`accessibilityDifferentiateWithoutColor`, `accessibilityVoiceOverEnabled`, `accessibilityReduceMotion`) are read from `@Environment` just like any other environment value — no special API
    - `UserDefaults` + `Codable` is the right tool for a small flat list. No SwiftData needed when the data model is simple and there are no relationships
    - Both `.onAppear` and the `onDismiss` callback on `.sheet` are valid places to call `loadData` — `onDismiss` is the right one when the sheet is what modifies the data
