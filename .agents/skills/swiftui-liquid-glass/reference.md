# Liquid Glass reference

Extended patterns for the `swiftui-liquid-glass` skill. Read when implementing morphing transitions, tuning container spacing, or choosing fallbacks.

## API surface (iOS 26+)

| API | Purpose |
|-----|---------|
| `.glassEffect(_:in:)` | Apply glass to any custom view |
| `.buttonStyle(.glass)` | Neutral glass button |
| `.buttonStyle(.glassProminent)` | Tinted glass button; pair with `.tint(Color)` |
| `.buttonBorderShape(_:)` | Capsule, circle, rounded rect for button chrome |
| `GlassEffectContainer` | Groups nearby glass for merge/morph |
| `.glassEffectID(_:in:)` | Identity for morphing transitions |
| `.glassEffectUnion(id:namespace:)` | Unite separate views into one glass blob |
| `.safeAreaBar(_:)` | System bottom/top bar with blur behind content |
| `.sharedBackgroundVisibility(_:)` | Hide shared glass on `ToolbarContent` (e.g. `ToolbarItem`) |

## Glass configuration

```swift
.glassEffect(.regular, in: .rect(cornerRadius: 16))
.glassEffect(.regular.tint(.orange), in: .capsule)
.glassEffect(.regular.interactive(), in: .circle)
```

- `.regular` — default variant
- `.tint(Color)` — color wash for prominence (not available as gradient on buttons via `.glassProminent`)
- `.interactive()` — glass reacts to touch/pointer; use on tappable custom surfaces

Apply `.glassEffect` **after** layout and appearance modifiers (padding, font, foregroundStyle).

## Button sizing and design handoff

Native glass buttons include ~13pt internal padding. When matching Figma or fixed-height specs:

- Measure the **outer** glass bounds, not just label text size.
- Prefer letting the button style size itself; avoid stacking extra `.padding()` on the label.
- Use `.buttonBorderShape(.roundedRectangle(radius:))` to match design corner radius without clipping content.

```swift
Button("Continue") { next() }
    .buttonStyle(.glassProminent)
    .tint(.accentColor)
    .buttonBorderShape(.roundedRectangle(radius: 14))
    .controlSize(.large) // optional — system sizing
```

## GlassEffectContainer spacing

`GlassEffectContainer(spacing:)` controls merge distance:

- **Smaller spacing** — elements must be closer to visually merge.
- **Larger spacing** — merge begins at greater distance (useful for toolbars that expand).

```swift
GlassEffectContainer(spacing: 40) {
    HStack(spacing: 16) {
        // glass elements
    }
}
```

The container `spacing` and inner stack `spacing` are independent — tune both for the desired merge behavior.

## Morphing transitions

Use when glass elements appear, disappear, or move with animation:

```swift
@Namespace private var glassNS
@State private var showEraser = false

var body: some View {
    GlassEffectContainer(spacing: 32) {
        HStack(spacing: 32) {
            Image(systemName: "pencil")
                .frame(width: 64, height: 64)
                .glassEffect()
                .glassEffectID("pencil", in: glassNS)

            if showEraser {
                Image(systemName: "eraser.fill")
                    .frame(width: 64, height: 64)
                    .glassEffect()
                    .glassEffectID("eraser", in: glassNS)
            }
        }
    }

    Button("Toggle") {
        withAnimation { showEraser.toggle() }
    }
    .buttonStyle(.glass)
}
```

Requires `@Namespace`, matching `glassEffectID` values, and `withAnimation` on hierarchy changes.

## Toolbar items and shared glass

On iOS 26+, items in navigation bars and window toolbars receive a shared Liquid Glass background within their logical grouping. Use `.sharedBackgroundVisibility(.hidden)` when an item should not show that capsule — common for plain icon buttons, custom titles, status labels, or logos.

```swift
NavigationStack {
    DetailView()
        .toolbar {
            ToolbarItem(placement: .principal) {
                BuildStatusLabel()
            }
            .sharedBackgroundVisibility(.hidden)

            ToolbarItem(placement: .topBarTrailing) {
                Button("Share", systemImage: "square.and.arrow.up") { share() }
            }
            .sharedBackgroundVisibility(.hidden)

            ToolbarItem(placement: .topBarTrailing) {
                Button("Done") { done() }
                    .buttonStyle(.glassProminent)
            }
        }
}
```

Notes:

- Apply on **`ToolbarItem`** (or other `ToolbarContent`), not on the inner `Button` or label.
- Hiding the effect places the item in its own grouping; spacing may differ from glass-backed neighbors.
- iOS 26+ only — omit or wrap in `#available(iOS 26, *)` when supporting earlier OS versions.

## safeAreaBar

For bottom (or top) fixed chrome with system blur:

```swift
NavigationStack {
    DetailContent()
        .safeAreaBar(.bottom) {
            HStack {
                Button("Edit") { edit() }
                    .buttonStyle(.glass)
                Spacer()
                Button("Save") { save() }
                    .buttonStyle(.glassProminent)
            }
        }
}
```

**Prefer over:**

```swift
// ❌ Manual overlay — misses system bar blur treatment
.overlay(alignment: .bottom) {
    bottomBar.background(.ultraThinMaterial)
}

// ❌ VStack pinning — fights safe area, no bar chrome
VStack {
    content
    Spacer()
    bottomBar
}
```

## Why to avoid glass in scroll views

Scroll views continuously shift content relative to the glass sampling region. Symptoms:

- Shimmering or unstable blur as cells move
- Incorrect light/refraction sampling
- Extra GPU work per frame during scroll

**Alternatives for scrollable content:**

- Standard list/card backgrounds (solid, semantic, or `.ultraThinMaterial` on pre-26)
- Glass only on **fixed** chrome: nav bars, tab bars, bottom bars via `safeAreaBar`, floating toolbars outside the scroll view

## Pre-iOS 26 fallbacks

| iOS 26+ | Fallback |
|---------|----------|
| `.glassEffect(...)` | `.background(.ultraThinMaterial, in: shape)` |
| `.buttonStyle(.glass)` | `.buttonStyle(.bordered)` |
| `.buttonStyle(.glassProminent)` | `.buttonStyle(.borderedProminent)` |
| `GlassEffectContainer` | Plain `HStack` / `VStack` |
| `.safeAreaBar(.bottom)` | `.safeAreaInset(edge: .bottom)` with material background |
| `.sharedBackgroundVisibility(.hidden)` | Omit (no shared toolbar glass before iOS 26) |

Wrap feature entry points:

```swift
@ViewBuilder
func glassSurface<Content: View>(
    @ViewBuilder content: () -> Content
) -> some View {
    if #available(iOS 26, *) {
        content().glassEffect(.regular, in: .rect(cornerRadius: 12))
    } else {
        content().background(.ultraThinMaterial, in: RoundedRectangle(cornerRadius: 12))
    }
}
```

## Custom glass imitation — do not

These patterns look "glass-like" but are **not** Liquid Glass:

```swift
// ❌ All of these
.background(.ultraThinMaterial)
.background { RoundedRectangle(cornerRadius: 16).fill(.white.opacity(0.2)) }
.overlay { RoundedRectangle(cornerRadius: 16).stroke(.white.opacity(0.3)) }
.shadow(radius: 8)
.background { VisualEffectBlur() } // UIKit blur wrappers
```

Liquid Glass requires the system material pipeline for correct refraction, merge, and interaction. Always use `.glassEffect` or glass button styles.

## Apple documentation

- [View.glassEffect(_:in:isEnabled:)](https://developer.apple.com/documentation/SwiftUI/View/glassEffect(_:in:isEnabled:))
- [GlassEffectContainer](https://developer.apple.com/documentation/SwiftUI/GlassEffectContainer)
- [GlassButtonStyle](https://developer.apple.com/documentation/SwiftUI/GlassButtonStyle)
- [GlassProminentButtonStyle](https://developer.apple.com/documentation/SwiftUI/GlassProminentButtonStyle)
