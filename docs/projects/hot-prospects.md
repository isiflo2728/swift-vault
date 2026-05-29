# HotProspects

> A conference networking app for tracking people you've met. Scan their QR code, drop them into your contacts, mark them as contacted or uncontacted, and send yourself a local reminder to follow up.

**Branch:** `hot-prospects` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui/tree/hot-prospects)

---

## What It Does

- Displays prospects across three tabs — Everyone, Contacted, and Uncontacted — each filtered dynamically from the same SwiftData store
- Tap the Scan button to open a `CodeScannerView` and import a contact's name and email from their QR code
- Swipe left on any row to delete or toggle contacted/uncontacted status
- Multi-select mode via `EditButton` — select multiple prospects and bulk-delete from a bottom action button
- Generates a personal QR code from your own name and email using Core Image
- Share your QR code via `ShareLink` from a context menu
- Schedule a local notification reminder for any prospect

---

## Concepts Covered

| Concept | Used For |
|---|---|
| `TabView` | Four-tab layout — Everyone, Contacted, Uncontacted, Me |
| `SwiftData` — `@Model`, `@Query`, `modelContext` | Persisting and querying `Prospect` objects |
| `#Predicate` + custom `@Query` init | Dynamically filtering prospects by `isContacted` at query time |
| `SortDescriptor` | Sorting prospects alphabetically by name |
| `CodeScanner` | QR code scanning via camera to import contacts |
| `UNUserNotificationCenter` | Scheduling local notifications as follow-up reminders |
| `CIFilter.qrCodeGenerator()` + `CIContext` | Generating a QR code image from name + email string |
| `ShareLink` + `SharePreview` | Sharing the generated QR code image |
| `.swipeActions` | Per-row delete and toggle-contacted actions |
| `List(selection:)` + `EditButton` | Multi-select mode for bulk operations |
| `.safeAreaInset(edge: .bottom)` | Bottom delete button that sits above the tab bar |
| `@AppStorage` | Persisting the user's own name and email across launches |
| `.onChange(of:)` | Regenerating the QR code when name or email changes |

---

## Architecture

Four SwiftUI views tied together by a `TabView` in `ContentView`:

```
ContentView  (TabView)
├── ProspectsView(filter: .none)         → Everyone tab
├── ProspectsView(filter: .contacted)    → Contacted tab
├── ProspectsView(filter: .uncontacted)  → Uncontacted tab
└── MeView                               → Personal QR code tab
```

`ProspectsView` is reused three times with a different `FilterType`. The `filter` drives a custom `@Query` initializer so each tab runs a separate SwiftData query — no manual filtering in the view body.

---

## Code Snippets

### The Prospect model

```swift
@Model
class Prospect {
    var name: String
    var emailAddress: String
    var isContacted: Bool

    init(name: String, emailAddress: String, isContacted: Bool) {
        self.name = name
        self.emailAddress = emailAddress
        self.isContacted = isContacted
    }
}
```

`@Model` makes `Prospect` automatically persistent, observable, and queryable with no extra setup.

---

### Dynamic @Query in init — the key pattern

The same view is used for all three tabs. The filter changes which query runs:

```swift
init(filter: FilterType) {
    self.filter = filter

    if filter != .none {
        let showContactedOnly = filter == .contacted
        _prospects = Query(filter: #Predicate {
            $0.isContacted == showContactedOnly
        }, sort: [SortDescriptor(\Prospect.name)])
    }
}
```

!!! tip "The underscore prefix"
    `_prospects` reaches into the `@Query` property wrapper itself to swap the query at init time. You can't assign to `prospects` directly — that's the unwrapped value. `_prospects` is the wrapper, and that's what you replace.

---

### Swipe actions with conditional toggle

```swift
.swipeActions {
    Button("Delete", systemImage: "trash", role: .destructive) {
        modelContext.delete(prospect)
    }
    if prospect.isContacted {
        Button("Mark uncontacted", systemImage: "person.crop.circle.badge.xmark") {
            prospect.isContacted.toggle()
        }
        .tint(.blue)
    } else {
        Button("Mark contacted", systemImage: "person.crop.circle.fill.badge.checkmark") {
            prospect.isContacted.toggle()
        }
        .tint(.green)
    }
}
```

SwiftData detects the `isContacted` mutation and automatically moves the prospect to the correct tab — no manual array manipulation needed.

---

### Multi-select + bulk delete

```swift
@State private var selectedProspects = Set<Prospect>()

List(prospects, selection: $selectedProspects) { ... }
    .toolbar {
        ToolbarItem(placement: .topBarLeading) { EditButton() }
    }
    .safeAreaInset(edge: .bottom) {
        if selectedProspects.isEmpty == false {
            Button("Delete Selected", action: delete)
                .padding()
                .frame(maxWidth: .infinity)
                .background(.regularMaterial)
        }
    }

func delete() {
    for prospect in selectedProspects {
        modelContext.delete(prospect)
    }
}
```

!!! warning "`.bottomBar` vs `.safeAreaInset`"
    Placing the delete button in a `ToolbarItem(placement: .bottomBar)` causes it to render behind the `TabView` tab bar because the conditional appearance doesn't properly account for the tab bar height. `.safeAreaInset(edge: .bottom)` always respects the safe area and sits correctly above the tab bar.

---

### QR code scan handler

```swift
func handleScan(result: Result<ScanResult, ScanError>) {
    isShowingScanner = false   // dismiss the sheet
    switch result {
    case .success(let result):
        let details = result.string.components(separatedBy: "\n")
        guard details.count == 2 else { return }
        let person = Prospect(name: details[0], emailAddress: details[1], isContacted: false)
        modelContext.insert(person)
    case .failure(let error):
        print("Scanning failed: \(error.localizedDescription)")
    }
}
```

The QR format is `"Name\nEmail"` — two components separated by a newline. `guard details.count == 2` rejects anything that doesn't match that format.

---

### QR code generation in MeView

```swift
let context = CIContext()
let filter = CIFilter.qrCodeGenerator()
@State private var qrCode = UIImage()

func generateQRCode(from string: String) -> UIImage {
    filter.message = Data(string.utf8)
    if let outputImage = filter.outputImage {
        if let cgImage = context.createCGImage(outputImage, from: outputImage.extent) {
            return UIImage(cgImage: cgImage)
        }
    }
    return UIImage(systemName: "xmark.circle") ?? UIImage()
}

func updateCode() {
    qrCode = generateQRCode(from: "\(name)\n\(emailAddress)")
}
```

Wired up with:

```swift
.onAppear(perform: updateCode)
.onChange(of: name, updateCode)
.onChange(of: emailAddress, updateCode)
```

The QR regenerates live as you type your name or email.

---

### Local notification scheduling

```swift
Button("Remind Me") {
    let center = UNUserNotificationCenter.current()
    center.requestAuthorization(options: [.alert, .badge, .sound]) { success, error in
        guard success else { return }
        let content = UNMutableNotificationContent()
        content.title = "Contact \(prospect.name)"
        content.subtitle = prospect.emailAddress
        content.sound = UNNotificationSound.default
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
        let request = UNNotificationRequest(identifier: UUID().uuidString, content: content, trigger: trigger)
        center.add(request)
    }
}
```

---

## Where I Needed Clarification

**Why was `.font(.secondary)` causing a compiler error?**

`.font()` takes a `Font` value, not a `Color`. `.secondary` is a `Color`. The fix is `.foregroundStyle(.secondary)`, which sets the text color to the system secondary color. Easy to confuse because both are lowercase dot-syntax values.

---

**Why did the scanner sheet open but immediately close — or open and never close?**

Inside `handleScan`, `isShowingScanner` was set to `true` instead of `false`. When a scan completes you want to *dismiss* the sheet, which means setting the binding to `false`. Setting it to `true` when it's already `true` does nothing — the sheet stays up forever on success, making it look like scanning didn't work.

---

**Why did `.onChange(of: name, perform: updateCode)` fail to compile?**

The old SwiftUI API `onChange(of:perform:)` passes the new value into the closure — so `perform:` expects `(String) -> Void`. But `updateCode` is `() -> Void`. These types don't match. The fix is the iOS 17+ API, which has a zero-parameter action overload:

```swift
.onChange(of: name, updateCode)      // ✅ new API — no value passed in
.onChange(of: emailAddress, updateCode)
```

---

**Why was the Delete Selected button hidden behind the tab bar?**

`ToolbarItem(placement: .bottomBar)` conditionally appearing inside a `NavigationStack` nested in a `TabView` doesn't always get the correct safe area offset. SwiftUI doesn't recalculate the tab bar height when the toolbar item appears for the first time. Using `.safeAreaInset(edge: .bottom)` on the `List` instead always works because it's always in the view tree — SwiftUI accounts for it in layout from the start.

---

## What I Learned

!!! success "Key Takeaways"
    - The `_property` underscore syntax lets you reach into a property wrapper at init time — the only way to construct a dynamic `@Query` based on runtime values
    - SwiftData automatically moves objects between filtered views when a property changes — no manual array work, just mutate `isContacted` and the tabs update instantly
    - `ToolbarItem(placement: .bottomBar)` and `TabView` don't play well together for conditional items — `.safeAreaInset(edge: .bottom)` is the reliable alternative
    - `UNUserNotificationCenter` requires explicit permission before scheduling — always check the `success` flag before calling `add()`
    - `.onChange(of:perform:)` is the old API; it passes the new value into the closure. The new iOS 17+ API passes nothing — use `.onChange(of: value, action)` with a `() -> Void` function
    - `.font()` takes a `Font`. `.foregroundStyle()` takes a `ShapeStyle` (including `Color`). They are not interchangeable
