# InstaFilter

> A photo filter app powered by Core Image. Import a photo, pick a filter, dial in the intensity, and share the result — all built with SwiftUI and Core Image.

**Branch:** `instafilter` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui/tree/instafilter)

---

## What It Does

- Import photos from the device library via `PhotosPicker`
- Apply Core Image filters: Sepia Tone, Crystallize, Pixellate, Gaussian Blur, Edges, Unsharp Mask, Vignette, and Twirl Distortion
- Adjust filter intensity with a `Slider`
- Share the processed image via `ShareLink`
- Prompt the user for an App Store review after 20 filter changes using StoreKit
- Shows `ContentUnavailableView` when no photo is selected

---

## Concepts Covered

| Concept | Used For |
|---|---|
| `CIFilter` / `CIContext` | Applying Core Image filters to photos |
| `PhotosPicker` | Importing photos from the device library |
| `UIImage` → `CIImage` → `CGImage` → `UIImage` | Full image processing pipeline |
| `kCIInputIntensityKey` / `kCIInputRadiusKey` | Safe cross-filter intensity setting |
| `ShareLink` + `SharePreview` | Sharing the processed image |
| `@AppStorage` | Persisting the filter usage count across launches |
| `StoreKit` `requestReview` | Prompting for an App Store review |
| `ContentUnavailableView` | Empty state when no photo is loaded |
| `confirmationDialog` | Filter picker bottom sheet |

---

## The Image Pipeline

This was the core concept of the project. SwiftUI's `Image` can only *display* pictures — it can't be processed. Core Image needs its own types. The full pipeline:

```
PhotosPicker → Data → UIImage → CIImage → CIFilter → CGImage → UIImage → SwiftUI Image
```

```swift
func loadImage() {
    Task {
        guard let imageData = try await selectedItem?.loadTransferable(type: Data.self) else { return }
        guard let inputImage = UIImage(data: imageData) else { return }
        let beginImage = CIImage(image: inputImage)
        currentFilter.setValue(beginImage, forKey: kCIInputImageKey)
        applyProcessing()
    }
}
```

---

## Key Code

### Cross-filter intensity setting

Different filters support different input keys. This checks before setting so any filter works without crashing:

```swift
func applyProcessing() {
    let inputKeys = currentFilter.inputKeys
    if inputKeys.contains(kCIInputIntensityKey) {
        currentFilter.setValue(filterIntensity, forKey: kCIInputIntensityKey)
    }
    if inputKeys.contains(kCIInputRadiusKey) {
        currentFilter.setValue(filterIntensity * 200, forKey: kCIInputRadiusKey)
    }
    if inputKeys.contains(kCIInputScaleKey) {
        currentFilter.setValue(filterIntensity * 10, forKey: kCIInputScaleKey)
    }

    guard let outputImage = currentFilter.outputImage else { return }
    guard let cgImage = context.createCGImage(outputImage, from: outputImage.extent) else { return }
    processedImage = Image(uiImage: UIImage(cgImage: cgImage))
}
```

### Swapping filters + StoreKit review prompt

```swift
@MainActor func setFilter(_ filter: CIFilter) {
    currentFilter = filter
    loadImage()

    filterCount += 1
    if filterCount >= 20 {
        requestReview()
    }
}
```

### Sharing the result

```swift
if let processedImage {
    ShareLink(
        item: processedImage,
        preview: SharePreview("InstaFilter", image: processedImage)
    )
}
```

---

## Questions I Had While Building This

**Why doesn't `didSet` fire when dragging a slider bound to `@State`?**

Because `@State` stores the value outside the struct in SwiftUI's memory. When the slider moves, SwiftUI updates that external storage directly — never through the property setter. Use `.onChange(of:)` instead.

**So `@State` isn't changing the view — it's wrapping a value stored outside?**

Exactly. `@State` hands the value off to SwiftUI. When it changes, SwiftUI throws away the old view and builds a fresh copy with the updated value. The struct never mutates — SwiftUI just swaps in a new one.

**Why did `.confirmationDialog` render as a popover bubble instead of a bottom sheet?**

Attaching `.confirmationDialog` directly to a `Button` causes iOS to sometimes render it as a popover. Fix: move the modifier to the parent container (`VStack`, `NavigationStack`) instead.

**Why wasn't Cancel visible in the confirmation dialog?**

On iPhone, Cancel is always separated at the bottom of the sheet. In popover mode it's hidden entirely — tapping outside dismisses instead. Fixing the popover issue makes Cancel appear.

**What does "SwiftUI's Image is a great endpoint but not useful elsewhere" mean?**

SwiftUI `Image` is display-only — you can't run Core Image filters on it. All processing happens on `UIImage` / `CIImage`. SwiftUI `Image` is only used at the very end to render the result on screen.

**What are the `inputKeys.contains()` checks doing?**

Different filters have different settings. Checking before setting means you can swap any filter in without crashing — like a universal remote that only sends commands the device understands.

**`@State private var image = Image?` — what's wrong?**

The `?` is a type annotation, not a value. Use a colon: `@State private var image: Image?`

**How do I reference an image from Assets.xcassets?**

```swift
Image(.example)              // SwiftUI display
UIImage(resource: .example) // UIImage for processing
```

Xcode generates type-safe resource names from the asset name automatically.

**Why was my view blank even though the code looked right?**

No visible `PhotosPicker` button — nothing to tap to trigger image loading. Always make sure the picker is rendered somewhere in the view tree.

**`forkey` vs `forKey` — why won't it compile?**

Swift is case-sensitive. The parameter is `forKey` (capital K). `forkey` is not a valid label and the compiler rejects it.

---

## What I Learned

!!! success "Key Takeaways"
    - SwiftUI `Image` is display-only — all Core Image work happens on `UIImage` and `CIImage`, then gets converted back at the end
    - `CIContext` is expensive to create — declare it once at the struct level, never inside a function
    - Checking `inputKeys` before setting values makes your filter code work with any `CIFilter`, not just one specific type
    - `.onChange(of:)` is the right tool for reacting to state changes — `didSet` doesn't fire reliably on `@State` properties driven by bindings
    - `StoreKit`'s `requestReview` via `@Environment` is one line — no setup required, just call it at the right moment

!!! tip "CIContext: keep it alive"
    Creating a `CIContext` is slow. If you put it inside `loadImage()` or `applyProcessing()`, you recreate it every time the slider moves. Declare it as a `let` constant on the view struct so it's created once and reused.