

<div class="timeline-item" markdown>

## Project 8 — InstaFilter
**Focus: Core Image — photo processing pipeline**

First project to touch media processing. The main challenge was understanding that SwiftUI's `Image` is display-only — all the real work happens in Core Image's own type system before converting back to SwiftUI at the end. Also introduced `PhotosPicker`, `ShareLink`, and StoreKit review prompts.

**New concepts:** `CIFilter`, `CIContext`, `CIImage`, `UIImage` ↔ `CGImage` conversion, `PhotosPicker`, `kCIInputIntensityKey` / `kCIInputRadiusKey`, `ShareLink`, `SharePreview`, `@AppStorage`, `StoreKit` `requestReview`, `ContentUnavailableView`, `confirmationDialog`

**The click:** The image pipeline — `UIImage → CIImage → filter → CGImage → UIImage → SwiftUI Image`. Once that chain clicked, Core Image stopped feeling like magic and started feeling mechanical. Also: `CIContext` is expensive, so it lives at the struct level, not inside the processing function.

</div>

</div>