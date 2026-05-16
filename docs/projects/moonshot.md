# Moonshot

> An interactive history of the Apollo missions — browse every mission in a dark-themed grid, drill into mission details, and explore individual astronaut bios.

**Branch:** `main` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui)

---

## What It Does

- `LazyVGrid` home screen showing all Apollo missions with mission patch images
- Tap a mission → detail view with crew list, mission description, and launch date
- Tap a crew member → full astronaut bio with photo
- All data decoded from bundled JSON at launch
- Custom dark color theme using asset catalog colors

---

## Concepts Covered

| Concept | Used For |
|---|---|
| [`LazyVGrid`](../concepts/layouts-lists.md) | Adaptive mission card grid on home screen |
| [`NavigationStack`](../concepts/navigation.md) | Three-level drill-down (grid → mission → astronaut) |
| `Codable` + `Bundle` extension | Generic JSON decoding helper |
| Custom `Color` assets | Dark theme palette defined in asset catalog |
| `ScrollView` | Scrollable mission detail with crew list |
| `ContainerRelativeShape` | Adaptive shape that respects container |
| Generic `Bundle.decode<T>()` | Reusable JSON loader for both model types |

---

## Key Code

### Generic Bundle JSON decoder — used everywhere

```swift
extension Bundle {
    func decode<T: Codable>(_ file: String) -> T {
        guard let url = self.url(forResource: file, withExtension: nil) else {
            fatalError("Failed to locate \(file) in bundle.")
        }
        guard let data = try? Data(contentsOf: url) else {
            fatalError("Failed to load \(file) from bundle.")
        }
        guard let loaded = try? JSONDecoder().decode(T.self, from: data) else {
            fatalError("Failed to decode \(file) from bundle.")
        }
        return loaded
    }
}

// Usage — type inference does the work
let missions: [Mission] = Bundle.main.decode("missions.json")
let astronauts: [String: Astronaut] = Bundle.main.decode("astronauts.json")
```

### LazyVGrid with adaptive columns

```swift
let columns = [GridItem(.adaptive(minimum: 150))]

ScrollView {
    LazyVGrid(columns: columns) {
        ForEach(missions) { mission in
            NavigationLink(value: mission) {
                MissionView(mission: mission)
            }
        }
    }
}
.navigationDestination(for: Mission.self) { mission in
    MissionDetailView(mission: mission, astronauts: astronauts)
}
```

### Crew member cross-reference (mission ↔ astronaut)

```swift
struct CrewMember {
    let role: String
    let astronaut: Astronaut
}

// Resolved at view init time — no lookup at render time
let crew: [CrewMember] = mission.crew.map { member in
    guard let astronaut = astronauts[member.name] else {
        fatalError("Missing \(member.name)")
    }
    return CrewMember(role: member.role, astronaut: astronaut)
}
```

---

## What I Learned

!!! success "Key Takeaways"
    - A generic `Bundle.decode<T>()` eliminates copy-pasted JSON loading across the whole app
    - `.adaptive(minimum:)` on `GridItem` automatically calculates the right number of columns for any screen width
    - `navigationDestination(for:)` is the modern replacement for `NavigationLink(destination:)` — it separates navigation logic from link labels
    - Resolving cross-references (crew → astronauts dictionary) at init time is faster than looking up inside `body`

!!! tip "Dark Theme"
    Define custom colors in the asset catalog (not in code) and reference them as `Color("MissionBackground")`. They automatically switch for light/dark mode without any logic in the view.
