# Swift Vault

This is my personal documentation and depiction of mastery of SwiftUI.

It is not a tutorial site. It is not written for anyone else. It is a living record of every concept learned, every project built, and every mistake made while becoming an iOS developer — documented in my own words, at my own pace.

## What This Is

A knowledge vault built alongside real SwiftUI projects. As each app gets built, the concepts it introduces get documented here — explained the way I actually understand them, not the way a textbook would explain them.

Every entry is written after the concept clicked. That means the explanations reflect genuine understanding, not surface-level repetition.

## What's Inside

- **Projects** — Deep-dives into each SwiftUI app I've built, covering what it does, the key code, and what I actually learned from building it
- **Concepts** — Standalone explanations of SwiftUI patterns that appear across multiple projects — state, navigation, persistence, networking, animations, and more
- **Timeline** — The full learning progression in order, showing how each project built on the last
- **Knowledge Graph** — A visual map of how concepts connect across the whole vault

## The Projects

| Project | Focus |
|---|---|
| WordScramble | Core SwiftUI primitives — `List`, `TextField`, `@State` |
| AnimationTechnique | Animations — implicit, explicit, transitions, gestures |
| iExpense | Shared state, `UserDefaults`, sheets |
| Moonshot | Complex data, `LazyVGrid`, multi-level navigation |
| Cupcake Corner | Networking — `URLSession`, `async/await`, `Codable` |
| BookWorm | SwiftData — `@Model`, `@Query`, custom components |
| SwiftDataProject | SwiftData deep dive — relationships, `#Predicate`, `SortDescriptor`, iCloud sync |
| InstaFilter | Core Image — `CIFilter`, `CIContext`, `PhotosPicker`, `ShareLink`, StoreKit review prompts |
| BucketList | MapKit — custom annotations, `MapReader`, Wikipedia API, Face ID / Touch ID |

## The Code

Every project lives on its own branch in [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui). This vault documents the thinking behind the code.

---

## Questions I Had — InstaFilter

Real questions that came up while building InstaFilter. Written here so the confusion doesn't happen twice.

---

**Why doesn't `didSet` fire when I drag a slider bound to `@State`?**

Because `@State` stores the value *outside* the struct in SwiftUI's memory. When the slider moves, SwiftUI updates that external storage directly — it never goes through your property setter, so `didSet` never fires. When you set the value yourself in code (like inside a button), it *does* go through the setter, so `didSet` fires then. Use `.onChange(of:)` instead to reliably track changes.

---

**So `@State` isn't changing the view — it's wrapping a value and storing it outside?**

Exactly. `@State` wraps your value and hands it off to SwiftUI to store outside the struct. When that value changes, SwiftUI throws away the old view and builds a fresh one with the updated value. Your struct never actually mutates — SwiftUI just swaps in a new copy.

---

**Why is `.confirmationDialog` showing as a popover instead of a bottom sheet on iPhone?**

When `.confirmationDialog` is attached directly to a `Button`, iOS sometimes renders it as a popover (bubble shape) rather than a bottom sheet. Fix it by moving the modifier off the button and onto the parent container (`VStack`, `NavigationStack`, etc.). That tells SwiftUI to present it as a proper sheet.

---

**Why isn't the Cancel button visible in my confirmation dialog?**

On iPhone, Cancel is intentionally separated from the other buttons — it appears at the very bottom of the sheet in its own section. If the dialog is rendering as a popover (bubble), Cancel is hidden entirely because tapping outside dismisses it instead. Fix the popover issue (see above) and Cancel will appear.

---

**What does "SwiftUI's `Image` is a great endpoint but not useful elsewhere" mean?**

SwiftUI `Image` can only *display* a picture — you can't run filters or manipulate pixels on it. For Core Image processing you need to work with `UIImage` or `CGImage`, apply your filters there, then convert the result back to a SwiftUI `Image` just to show it on screen. The flow is: `UIImage → CIImage → filter → CGImage → UIImage → SwiftUI Image`.

---

**What are all those `inputKeys.contains()` checks doing in Core Image?**

Different filters have different settings — a blur has a radius, a color filter has intensity, etc. Instead of assuming every filter supports every setting (which would crash), the code checks first: "Does this filter have an intensity knob? If yes, set it." Think of it like a universal remote — it tries every button, but only the ones your device understands actually do anything. This means you can swap in any filter and the code still works without crashing.

---

**What's wrong with `@State private var image = Image?`?**

The `?` is a type annotation (meaning "this can be nil"), not a value. Type annotations use a colon, not an equals sign. It should be:
```swift
@State private var image: Image?
```

---

**How do I use an image from Assets.xcassets in code?**

Once the image is added to Assets.xcassets with a name (e.g. "example"), reference it with:
```swift
Image(.example)         // SwiftUI Image
UIImage(resource: .example)  // UIImage for Core Image processing
```
The dot syntax works because Xcode auto-generates a type-safe resource name from the asset name.

---

**Why is my view blank — nothing showing up even though my code looks right?**

Most likely you have the `PhotosPicker` or data loading set up but no button or trigger visible on screen to kick it off. A `ScrollView` or `VStack` with no content renders blank. Make sure you have a visible `PhotosPicker` or button that the user can actually tap.

---

**`forkey` vs `forKey` — why won't my code compile?**

Swift is case-sensitive. The parameter label is `forKey` with a capital K. `forkey` is not recognized and the compiler will throw an error. Always use:
```swift
currentFilter.setValue(value, forKey: kCIInputIntensityKey)
```

---

## The Point

Skills are not given. They're built — one commit at a time.

This vault exists to make that process visible. Not just the finished apps, but the understanding behind them. Every concept documented here is one that was earned.

---