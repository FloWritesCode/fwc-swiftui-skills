---
name: swiftui-liquid-glass
description: >-
  Implement, review, and refactor SwiftUI features using the iOS 26+ Liquid Glass
  API. Use when adopting Liquid Glass in new UI, converting existing surfaces to
  glass, reviewing glass usage for correctness, or fixing common Liquid Glass
  pitfalls (custom blur stacks, scroll views, button shapes, bottom bars).
---

# SwiftUI Liquid Glass

Use native Liquid Glass APIs on iOS 26+. Do not recreate the effect with materials, blurs, or shadows.

## Core rules

1. **Don't build custom Liquid Glass** through adding backgrounds, outlines, blurs and shadows.
2. **Buttons — style**: use `.buttonStyle(.glass)` for non-colored buttons, and `.buttonStyle(.glassProminent)` for tinted buttons. `.glassProminent` supports `.tint()`, but this can only be a color (not a gradient).
3. **Buttons — shape**: use `.buttonBorderShape()` instead of applying a shape to the button label by hand.
4. **Buttons — padding**: using this button setup adds roughly 13pt of padding inside the liquid glass shape; keep this in mind when adapting designs.
5. **Custom views**: use `.glassEffect(.regular, in: ...)` to embed custom Views inside liquid glass containers.
6. **Grouped glass**: use `GlassEffectContainer` when multiple liquid glass elements are next to each other. this View / Container adds a liquid merge effect when the elements grow / touch each other.
7. **Scrolling**: avoid using liquid glass inside ScrollView and List (anything that scrolls).
8. **Bottom bars**: when anchoring a liquid glass View to the bottom of the screen, prefer embedding it in `.safeAreaBar(.bottom)` instead of a VStack or `.overlay()`. safeAreaBar adds a subtle blur effect behind its content.

## Decision tree

```
Need a button?
├─ Neutral / secondary → .buttonStyle(.glass)
└─ Tinted / primary    → .buttonStyle(.glassProminent).tint(someColor)

Need a custom non-button surface (chip, badge, card)?
└─ .glassEffect(.regular, in: shape) on the view content

Multiple glass elements nearby?
└─ Wrap in GlassEffectContainer(spacing: ...) { ... }

Fixed bottom toolbar / action bar?
└─ .safeAreaBar(.bottom) { ... }  — not VStack + overlay

Inside ScrollView, List, or Form rows?
└─ Do not use Liquid Glass — use solid/material fallback
```

## Workflow

### 1) Review existing UI

- Flag custom blur/material stacks masquerading as glass.
- Check buttons use `.glass` / `.glassProminent`, not hand-built capsules.
- Confirm `.buttonBorderShape()` is used instead of clipping the label.
- Verify grouped elements sit in `GlassEffectContainer`.
- Flag glass inside scrollable containers.
- Check bottom-anchored bars use `.safeAreaBar(.bottom)`.
- Gate with `#available(iOS 26, *)` and provide fallbacks.

### 2) Implement or refactor

1. Pick the right primitive (button style vs `glassEffect` vs `safeAreaBar`).
2. Apply layout and typography first; add glass modifiers last.
3. Wrap adjacent glass elements in `GlassEffectContainer`.
4. Account for ~13pt internal button padding when matching designs.
5. Add iOS 26 availability checks and pre-26 fallbacks.

## Patterns

### Glass buttons

```swift
// Secondary / neutral
Button("Cancel") { dismiss() }
    .buttonStyle(.glass)
    .buttonBorderShape(.capsule)

// Primary / tinted — color only, not gradient
Button("Save") { save() }
    .buttonStyle(.glassProminent)
    .tint(.blue)
    .buttonBorderShape(.roundedRectangle(radius: 12))
```

Do **not** clip the label yourself:

```swift
// ❌ Wrong — shape on label, not the glass button
Button { action() } label: {
    Text("Save")
        .padding()
        .background(.ultraThinMaterial, in: Capsule())
}

// ✅ Right — native glass handles shape and padding
Button("Save") { action() }
    .buttonStyle(.glassProminent)
    .buttonBorderShape(.capsule)
```

### Custom glass surfaces

```swift
Label("3 items", systemImage: "tray")
    .padding(.horizontal, 16)
    .padding(.vertical, 10)
    .glassEffect(.regular, in: .capsule)
```

Add `.interactive()` when the surface responds to touch:

```swift
Text("Tap me")
    .padding()
    .glassEffect(.regular.interactive(), in: .rect(cornerRadius: 16))
```

### Grouped glass (merge effect)

```swift
GlassEffectContainer(spacing: 24) {
    HStack(spacing: 24) {
        ToolButton(icon: "pencil")
        ToolButton(icon: "eraser")
        ToolButton(icon: "lasso")
    }
}

private struct ToolButton: View {
    let icon: String
    var body: some View {
        Image(systemName: icon)
            .frame(width: 56, height: 56)
            .font(.title2)
            .glassEffect(.regular, in: .circle)
    }
}
```

Tune `spacing` to control how close elements must be before the liquid merge kicks in.

### Bottom action bar

```swift
ContentView()
    .safeAreaBar(.bottom) {
        HStack {
            Button("Share") { share() }
                .buttonStyle(.glass)
            Button("Done") { done() }
                .buttonStyle(.glassProminent)
        }
    }
```

Prefer this over pinning with `VStack { Spacer(); ... }` or `.overlay(alignment: .bottom)`.

### Availability fallback

```swift
if #available(iOS 26, *) {
    content.glassEffect(.regular, in: .rect(cornerRadius: 16))
} else {
    content.background(.ultraThinMaterial, in: RoundedRectangle(cornerRadius: 16))
}
```

## Anti-patterns

| Anti-pattern | Why it fails | Do instead |
|---|---|---|
| `.background(.ultraThinMaterial)` + blur + stroke + shadow | Not Liquid Glass; wrong optics and no merge/morph | `.glassEffect(...)` or button styles |
| Gradient `.tint()` on `.glassProminent` | API accepts color only | Solid `Color` tint, or custom label without glassProminent tint |
| Glass on `List`/`ScrollView` rows | Scroll + glass = visual glitches, perf cost | Opaque/material rows; glass only on fixed chrome |
| Shape applied to button label | Misses native padding (~13pt) and border rendering | `.buttonBorderShape()` |
| Bottom bar via `.overlay` | No system blur bar treatment | `.safeAreaBar(.bottom)` |
| Multiple glass views without container | No merge effect, worse rendering | `GlassEffectContainer` |

## Review checklist

- [ ] No hand-rolled blur/material glass imitations
- [ ] Buttons use `.glass` or `.glassProminent` with `.buttonBorderShape()`
- [ ] Design spacing accounts for ~13pt internal button padding
- [ ] Custom surfaces use `.glassEffect(.regular, in: ...)`
- [ ] Adjacent glass wrapped in `GlassEffectContainer`
- [ ] No glass inside scroll views
- [ ] Bottom chrome uses `.safeAreaBar(.bottom)`
- [ ] `#available(iOS 26, *)` with fallback on older OS

## Additional resources

- Detailed API notes and morphing transitions: [reference.md](reference.md)
- [Applying Liquid Glass to custom views](https://developer.apple.com/documentation/SwiftUI/Applying-Liquid-Glass-to-custom-views)
- [Landmarks: Building an app with Liquid Glass](https://developer.apple.com/documentation/SwiftUI/Landmarks-Building-an-app-with-Liquid-Glass)
