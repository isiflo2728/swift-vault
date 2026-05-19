# Core Image

> Processing photos with filters — converting between image types, applying Core Image effects, and piping the result back into SwiftUI.

**Appears in:** InstaFilter

---

## The Pipeline

SwiftUI's `Image` is display-only. You can't run filters on it. All processing happens in Core Image's own type system, then gets converted back to SwiftUI at the very end.

```
PhotosPicker → Data → UIImage → CIImage → CIFilter → CGImage → UIImage → SwiftUI Image
```

Every arrow is a type conversion. The rule: do all your work in Core Image's world, then hand the finished result to SwiftUI.

---

## The Three Image Types

| Type | Lives In | What It Is |
|---|---|---|
| `UIImage` | UIKit | High-level image — supports PNG, JPEG, SVG, animation sequences |
| `CGImage` | Core Graphics | Low-level 2D pixel array |
| `CIImage` | Core Image | An "image recipe" — stores instructions, not pixels, until rendered |

**Conversions:**
```swift
// UIImage → CIImage
let ciImage = CIImage(image: uiImage)

// CIImage → CGImage (requires a CIContext)
let cgImage = context.createCGImage(ciImage, from: ciImage.extent)

// CGImage → UIImage
let uiImage = UIImage(cgImage: cgImage)

// UIImage → SwiftUI Image
let swiftUIImage = Image(uiImage: uiImage)
```

---

## CIContext

The rendering engine. Converting a `CIImage` to a `CGImage` requires a context.

```swift
let context = CIContext()
```

!!! warning "Create it once — never inside a function"
    `CIContext` is expensive to initialize. If you put it inside `applyProcessing()`, it gets recreated every time the slider moves. Declare it as a `let` constant on the view struct so it's created once and reused for every render.

```swift
struct ContentView: View {
    let context = CIContext() // ✓ created once
    ...
}
```

---

## Applying a Filter

```swift
// 1. Pick a filter
let currentFilter: CIFilter = CIFilter.sepiaTone()

// 2. Feed it the image
let beginImage = CIImage(image: inputImage)
currentFilter.setValue(beginImage, forKey: kCIInputImageKey)

// 3. Set intensity
currentFilter.setValue(0.8, forKey: kCIInputIntensityKey)

// 4. Render
guard let outputImage = currentFilter.outputImage else { return }
guard let cgImage = context.createCGImage(outputImage, from: outputImage.extent) else { return }
let result = Image(uiImage: UIImage(cgImage: cgImage))
```

---

## Cross-Filter Safe Intensity

Different filters support different input keys. Setting a key a filter doesn't have will crash. Check first:

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

Think of it like a universal remote — it sends every command, but only the ones the filter understands actually do anything. This lets you swap any filter in without changing the processing code.

---

## PhotosPicker

Import photos from the device library without leaving SwiftUI.

```swift
import PhotosUI

@State private var selectedItem: PhotosPickerItem?
@State private var processedImage: Image?

PhotosPicker(selection: $selectedItem) {
    if let processedImage {
        processedImage
            .resizable()
            .scaledToFit()
    } else {
        ContentUnavailableView("No picture", systemImage: "photo.badge.plus")
    }
}
.onChange(of: selectedItem, loadImage)
```

Loading the image data is async — `PhotosPickerItem` is just a reference until you call `loadTransferable`:

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

## Swapping Filters

```swift
@MainActor func setFilter(_ filter: CIFilter) {
    currentFilter = filter
    loadImage() // re-run pipeline with the new filter
}
```

`@MainActor` ensures UI updates happen on the main thread after the async load.

---

## Available Filters

```swift
CIFilter.sepiaTone()       // warm brown tone
CIFilter.pixellate()       // blocky pixel effect
CIFilter.crystallize()     // crystal/voronoi shatter
CIFilter.gaussianBlur()    // soft blur
CIFilter.edges()           // edge detection / sketch look
CIFilter.unsharpMask()     // sharpen details
CIFilter.vignette()        // darken edges
CIFilter.twirlDistortion() // swirl warp
```

---

## Sharing the Result

```swift
if let processedImage {
    ShareLink(
        item: processedImage,
        preview: SharePreview("InstaFilter", image: processedImage)
    )
}
```

`ShareLink` is conditional — only render it when there's actually a processed image to share.

---

## Quick Reference

| Task | API |
|---|---|
| Import photo from library | `PhotosPicker` + `loadTransferable(type: Data.self)` |
| Convert UIImage for Core Image | `CIImage(image: uiImage)` |
| Apply a filter | `filter.setValue(value, forKey: key)` |
| Render CIImage to pixels | `context.createCGImage(output, from: output.extent)` |
| Convert back to SwiftUI | `Image(uiImage: UIImage(cgImage: cgImage))` |
| Share the result | `ShareLink(item:preview:)` |
| Check supported keys | `filter.inputKeys.contains(kCIInputIntensityKey)` |