# InstaFilter

> A photo filter app powered by Core Image. Import a photo, pick a filter, dial in the intensity, and share the result — all built with SwiftUI and Core Image.

**Branch:** `instafilter` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui/tree/instafilter)

---

## What It Does

- Import photos from the device library via `PhotosPicker`
- Apply Core Image filters: Sepia Tone, Crystallize, Pixellate, Gaussian Blur, Edges, Unsharp Mask, Vignette, Twirl Distortion
- Adjust filter intensity with a `Slider`
- Share the processed image via `ShareLink`
- Prompt for an App Store review after 20 filter changes using StoreKit
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
| `.onChange(of:)` | Reacting to slider and picker changes |

---

## The Image Pipeline

This was the core concept of the project. SwiftUI's `Image` is display-only — it can't be processed. Core Image needs its own type system. The full pipeline:

```
PhotosPicker → Data → UIImage → CIImage → CIFilter → CGImage → UIImage → SwiftUI Image
```

Every step is a type conversion. You work in Core Image's world, then hand the result back to SwiftUI at the very end.

---

## Code Snippets

### State setup

```swift
@State private var processedImage: Image?
@State private var filterIntensity = 0.5
@State private var selectedItem: PhotosPickerItem?
@State private var currentFilter: CIFilter = CIFilter.sepiaTone()
@State private var showingFilters = false

let context = CIContext() // expensive to create — lives at struct level, not inside a function

@AppStorage("filterCount") var filterCount = 0
@Environment(\.requestReview) var requestReview
```

!!! warning "Keep CIContext alive"
    `CIContext` is slow to create. If you put it inside `loadImage()` or `applyProcessing()`, it gets recreated every time the slider moves. Declare it once as a `let` on the struct.

---

### PhotosPicker — importing a photo

```swift
PhotosPicker(selection: $selectedItem) {
    if let processedImage {
        processedImage
            .resizable()
            .scaledToFit()
    } else {
        ContentUnavailableView(
            "No picture",
            systemImage: "photo.badge.plus",
            description: Text("Tap to import a photo")
        )
    }
}
.onChange(of: selectedItem, loadImage)
```

The `PhotosPicker` wraps your UI — tapping it opens the photo library. `.onChange(of: selectedItem, loadImage)` fires `loadImage()` automatically when the user picks a photo.

---

### Loading and converting the image

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

`loadTransferable` is async — it has to fetch the full image data from the photo library. That's why it lives inside a `Task {}`. The chain: `Data → UIImage → CIImage` then hands off to `applyProcessing()`.

---

### Applying the filter — cross-filter safe intensity

Different filters support different input keys. Checking before setting means any filter works without crashing:

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

The final line converts back to a SwiftUI `Image` for display. Core Image stays in its own world the entire time.

---

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

`@MainActor` ensures UI updates happen on the main thread. `filterCount` is persisted in `@AppStorage` so it survives app restarts — the review prompt only fires once the user has genuinely used the app.

---

### Filter picker with confirmationDialog

```swift
.confirmationDialog("Select a filter", isPresented: $showingFilters) {
    Button("Crystallize")    { setFilter(CIFilter.crystallize()) }
    Button("Edges")          { setFilter(CIFilter.edges()) }
    Button("Gaussian Blur")  { setFilter(CIFilter.gaussianBlur()) }
    Button("Pixellate")      { setFilter(CIFilter.pixellate()) }
    Button("Sepia Tone")     { setFilter(CIFilter.sepiaTone()) }
    Button("Unsharp Mask")   { setFilter(CIFilter.unsharpMask()) }
    Button("Vignette")       { setFilter(CIFilter.vignette()) }
    Button("Cancel", role: .cancel) { }
}
```

!!! tip "Attach confirmationDialog to the container, not the button"
    If you attach `.confirmationDialog` directly to a `Button`, iOS sometimes renders it as a popover bubble instead of a bottom sheet — and the Cancel button disappears. Attach it to the parent `NavigationStack` or `VStack` instead.

---

### Sharing the processed image

```swift
if let processedImage {
    ShareLink(
        item: processedImage,
        preview: SharePreview("InstaFilter", image: processedImage)
    )
}
```

`ShareLink` is conditional — it only appears once there's a processed image to share. `SharePreview` gives the share sheet a thumbnail and title.

---

### Slider wired to filter intensity

```swift
HStack {
    Text("Intensity")
    Slider(value: $filterIntensity)
        .onChange(of: filterIntensity, applyProcessing)
}
```

`.onChange(of: filterIntensity, applyProcessing)` calls `applyProcessing()` every time the slider moves, reprocessing the image live.

---

## Where I Needed Clarification

**Why doesn't `didSet` fire when dragging a slider bound to `@State`?**

Because `@State` stores the value *outside* the struct in SwiftUI's memory. When the slider moves, SwiftUI updates that external storage directly — it never goes through your property setter, so `didSet` never fires. When you set the value yourself in code (like a button tap), that does go through the setter. Use `.onChange(of:)` instead — it's the correct tool for watching state changes.

---

**So `@State` isn't actually changing the view?**

Exactly. `@State` hands the value off to SwiftUI to store outside the struct. When the value changes, SwiftUI throws away the old view and builds a fresh copy with the updated value. Your struct never mutates — SwiftUI just swaps in a new one.

---

**Why did `.confirmationDialog` show as a popover bubble instead of a bottom sheet?**

When `.confirmationDialog` is attached directly to a `Button`, iOS sometimes renders it as a popover. Moving the modifier to the parent container (`NavigationStack`, `VStack`) fixes it and makes Cancel visible at the bottom.

---

**What does "SwiftUI's Image is a great endpoint but not useful elsewhere" mean?**

SwiftUI `Image` can only *display* a picture — you can't run filters or manipulate pixels through it. All Core Image processing happens on `UIImage` and `CIImage`. SwiftUI `Image` only appears at the very end of the pipeline to put the result on screen.

---

**What are the `inputKeys.contains()` checks actually doing?**

Different filters expose different settings — a blur has a radius, a color filter has intensity, etc. Checking before setting means you never try to set a knob a filter doesn't have. Think of it like a universal remote — it sends every button command, but only the ones your TV understands actually do anything. This way you can swap any filter in without the code crashing.

---

**`@State private var image = Image?` — what's wrong with it?**

The `?` is a type annotation (meaning "this can be nil"), not a value being assigned. Type annotations need a colon, not an equals sign:

```swift
@State private var image: Image?
```

---

**How do I reference an image from Assets.xcassets in code?**

```swift
Image(.example)               // SwiftUI Image for display
UIImage(resource: .example)   // UIImage for Core Image processing
```

Xcode auto-generates type-safe resource names from asset names — the dot syntax works because of that generation.

---

**Why was my view completely blank?**

No visible `PhotosPicker` or button in the view tree. A `ScrollView` or `VStack` with no content and no trigger renders blank. You always need a visible element the user can tap to load data.

---

**`forkey` vs `forKey` — why won't it compile?**

Swift is case-sensitive. The correct parameter label is `forKey` with a capital K:

```swift
currentFilter.setValue(filterIntensity, forKey: kCIInputIntensityKey)
```

`forkey` is not recognized and the compiler rejects it.

---

## What I Learned

!!! success "Key Takeaways"
    - SwiftUI `Image` is display-only — all Core Image work happens on `UIImage` and `CIImage`, converted back at the very end
    - `CIContext` is expensive to create — declare it once at the struct level, never inside a processing function
    - Checking `inputKeys` before setting values makes filter code work with *any* `CIFilter`, not just one specific type
    - `.onChange(of:)` is the right tool for watching state changes — `didSet` doesn't fire reliably on `@State` driven by bindings
    - StoreKit's `requestReview` via `@Environment` is one line — no setup, just call it at the right moment
    - `@AppStorage` persists a value across launches with zero boilerplate — perfect for counters like filter usage

!!! tip "CIContext — the most common Core Image mistake"
    Every beginner puts `CIContext()` inside the function that processes the image. That works, but it creates a new context on every slider tick. Move it to the struct level and the performance difference is immediate.