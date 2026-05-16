# WordScramble

> A word game that challenges players to build anagrams from a random root word, validating input against a real dictionary.

**Branch:** `main` · **Repo:** [mastering-swift-ui](https://github.com/isiflo2728/mastering-swift-ui)

---

## What It Does

- Presents a random 8-letter root word
- Player enters valid English words using only the letters in the root word
- Validates: real word, not already used, not the root word itself, minimum length
- Shows all found words in a growing `List`
- Tracks score, allows restart

---

## Concepts Covered

| Concept | Used For |
|---|---|
| [`List`](../concepts/layouts-lists.md) | Displaying found words with dynamic rows |
| [`NavigationStack`](../concepts/navigation.md) | Title bar and toolbar |
| [`TextField`](../concepts/state-data-flow.md) | Player input |
| `@State` | Root word, used words, error state |
| `onSubmit` | Trigger validation when player hits return |
| `UITextChecker` | Spell-check validation against the iOS dictionary |
| `Bundle` resource loading | Reading `start.txt` word list at runtime |
| `onAppear` | Starting the game when view loads |

---

## Key Code

### Loading the word list from Bundle

```swift
func startGame() {
    guard let startWordsURL = Bundle.main.url(forResource: "start", withExtension: "txt"),
          let startWords = try? String(contentsOf: startWordsURL) else {
        fatalError("Could not load start.txt from bundle.")
    }
    let allWords = startWords.components(separatedBy: "\n")
    rootWord = allWords.randomElement() ?? "silkworm"
    usedWords = []
}
```

### Chained validation

```swift
func addNewWord() {
    let answer = newWord.lowercased().trimmingCharacters(in: .whitespacesAndNewlines)
    guard answer.count > 2 else { return }
    guard isOriginal(word: answer) else { return wordError(title: "Already used", message: "Be more original!") }
    guard isPossible(word: answer) else { return wordError(title: "Not possible", message: "You can't spell that from '\(rootWord)'!") }
    guard isReal(word: answer) else { return wordError(title: "Not recognized", message: "You can't just make them up!") }

    withAnimation {
        usedWords.insert(answer, at: 0)
    }
    newWord = ""
}
```

### UITextChecker — real dictionary validation

```swift
func isReal(word: String) -> Bool {
    let checker = UITextChecker()
    let range = NSRange(location: 0, length: word.utf16.count)
    let misspelledRange = checker.rangeOfMisspelledWord(in: word, range: range, startingAt: 0, wrap: false, language: "en")
    return misspelledRange.location == NSNotFound
}
```

---

## What I Learned

!!! success "Key Takeaways"
    - `List` with `withAnimation` on insert gives smooth row additions for free
    - `UITextChecker` bridges UIKit into SwiftUI cleanly — you don't need to reinvent the dictionary
    - Chaining guard statements for validation keeps logic flat and readable
    - `Bundle.main.url(forResource:withExtension:)` is the right way to load static assets at runtime

!!! tip "Tricky Part"
    Getting the `NSRange`/`utf16.count` pairing right for `UITextChecker`. Swift strings and `NSRange` don't share the same index model — using `.utf16.count` bridges them correctly.
