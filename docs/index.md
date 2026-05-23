# Mastering SwiftUI

> Skills are not given. They're built — one commit at a time.

This is a living knowledge vault documenting the process of mastering iOS development with SwiftUI. Every project here was built from scratch, every bug was real, and every concept was learned the hard way.

---

## What's Inside

=== "Projects"
    Nine real SwiftUI apps, each targeting a new layer of the iOS ecosystem.

    | Project | Focus | Key Concepts |
    |---|---|---|
    | [WordScramble](projects/wordscramble.md) | Word game | `List`, `TextField`, `NavigationStack` |
    | [AnimationTechnique](projects/animation-technique.md) | Motion | Implicit/explicit animations, transitions |
    | [iExpense](projects/iexpense.md) | Finance tracker | `UserDefaults`, `@State`, sheets |
    | [Moonshot](projects/moonshot.md) | Apollo missions | `LazyVGrid`, JSON decoding, multi-level nav |
    | [Cupcake Corner](projects/cupcake-corner.md) | Ordering app | `URLSession`, `Codable`, `@Observable` |
    | [BookWorm](projects/bookworm.md) | Book tracker | `SwiftData`, `@Model`, custom components |
    | [SwiftDataProject](projects/swiftdataproject.md) | User & job manager | `#Predicate`, `SortDescriptor`, `@Bindable`, CloudKit |
    | [InstaFilter](projects/instafilter.md) | Photo filter app | `CIFilter`, `PhotosPicker`, `ShareLink`, StoreKit |
    | [BucketList](projects/bucketlist.md) | Location bookmarks | `MapKit`, `MapReader`, `LocalAuthentication`, Wikipedia API |

=== "Concepts"
    Concepts extracted and linked across all projects so patterns become visible.

    - [State & Data Flow](concepts/state-data-flow.md) — `@State`, `@Binding`, `@Observable`, `@Environment`
    - [Navigation](concepts/navigation.md) — `NavigationStack`, drill-down, sheets, full-screen covers
    - [Layouts & Lists](concepts/layouts-lists.md) — `VStack`, `LazyVGrid`, `List`, custom rows
    - [Animations](concepts/animations.md) — implicit, explicit, transitions, `withAnimation`
    - [Persistence](concepts/persistence.md) — `UserDefaults`, `SwiftData`, `@Model`
    - [Networking](concepts/networking.md) — `URLSession`, `async/await`, `Codable`
    - [Core Image](concepts/core-image.md) — `CIFilter`, `CIContext`, photo processing pipeline
    - [Maps & Location](concepts/maps-location.md) — `MapKit`, `MapReader`, `Annotation`, coordinates

=== "Timeline"
    [The full learning progression →](timeline.md) — from first `List` to MapKit and biometric auth.

=== "Knowledge Graph"
    [See how everything connects →](graph.md) — a visual map of concepts across projects.

---

## The Mindset

Nobody starts out knowing how to build apps. Every developer you look up to was once stuck on the same errors, confused by the same concepts, wondering if it was worth continuing. The difference is they kept going.

This vault is proof of that process. Not every commit is clean. Not every project is perfect. But every single one moved the needle forward — a new concept clicked, a bug finally made sense, a feature that seemed impossible got shipped.

**That's how it works. You build until you are ready.**

---

*Built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) · Source on [GitHub](https://github.com/isiflo2728/mastering-swift-ui)*