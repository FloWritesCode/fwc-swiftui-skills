---
name: swiftui-iphone-duo
description: >-
  Adapts and reviews SwiftUI apps for iPhone Duo and continuously changing
  window sizes. Use when supporting the foldable iPhone Duo, hinge or fold
  layout, ArrangementView, reserved regions, scene accessories, outer or inner
  display, NavigationSplitView, adaptive TabView, ViewThatFits, AnyLayout, or
  when replacing UIDevice / UIScreen / idiom / orientation layout branches.
---

# SwiftUI iPhone Duo

**Do not design a separate “Duo version” of the app.**

Start with an adaptive SwiftUI interface that works across continuously changing widths and heights. Add Duo-specific behavior only when the fold, hinge, second display, or vertical system bars materially improve the experience.

The target is not “Make this app support iPhone Duo.” The target is:

**Make this app excellent at every size, then use Duo's unique hardware where it creates additional value.**

Before reviewing or changing layout, read the full rulebook: [reference.md](reference.md).

## When to use

- User asks to support **iPhone Duo**, a **foldable iPhone**, **hinge**, **fold**, or **two displays**.
- User asks for **adaptive SwiftUI layout** across compact → wide, including **multitasking**.
- Code uses `UIDevice`, `UIScreen.main.bounds`, idiom, orientation, `isDuo`, or `isFolded` for layout.
- Work involves `ArrangementView`, reserved regions, `onHingeChange`, or scene accessories.

## Related skills

| Skill | When to load |
|-------|--------------|
| **swiftui-whats-new-27** | Toolbar overflow, `visibilityPriority`, `ToolbarOverflowMenu`, `.topBarPinnedTrailing` |
| **swiftui-specialist** | Broader SwiftUI layout and navigation correctness |
| **swiftui-liquid-glass** | System bars and glass chrome on iOS 26+ |

---

## Optimization order

1. Adaptive layout
2. Adaptive navigation
3. Adaptive toolbars and tabs
4. Fold-safe positioning
5. Duo-specific arrangements
6. Hinge-driven interactions
7. Second-display experiences

## Progressive API tiers

Classify every change before writing code. **Always complete Tier 1 before proposing Tier 2 or Tier 3.**

**Tier 1 — Universal adaptive improvements** (do these first):

```text
NavigationSplitView
adaptive TabView
adaptive grids
ViewThatFits
AnyLayout
size classes
container-relative sizing
system toolbars
safe-area correctness
```

These benefit all Apple platforms and window sizes.

**Tier 2 — Duo-aware layout** (only when needed):

```text
Reserved Regions
ArrangementView
vertical toolbar behavior
Duo-specific safe-area handling
```

**Tier 3 — Duo-exclusive experiences** (only when the product benefits):

```text
onHingeChange
hinge angle
scene accessories
multi-display interactions
multi-scene Duo workflows
```

---

## Workflow

Copy and track progress:

```
- [ ] 1. Read reference.md
- [ ] 2. Review screens with the decision tree
- [ ] 3. Flag red-flag patterns with file evidence
- [ ] 4. Classify each change as Tier 1 / 2 / 3
- [ ] 5. Implement Tier 1 first
- [ ] 6. Apply Tier 2/3 only when space-based layout is insufficient
- [ ] 7. Keep state above adaptive layout; verify continuity across widths
```

### 1) Review existing UI

Walk every SwiftUI screen with the decision tree below. Flag:

- Device identity used as a layout switch (`UIDevice`, idiom, model, `isDuo`, `isFolded`)
- `UIScreen.main.bounds` or orientation used for ordinary layout
- Fixed column counts, hardcoded sidebar widths, arbitrary `if width > N`
- Separate compact vs expanded view hierarchies with duplicated state
- Custom toolbars / tab bars that cannot move to a vertical edge
- Critical content that would sit on the fold
- Wide layouts that only stretch instead of exposing hierarchy
- Hinge angle used to decide sidebar, columns, or navigation

### 2) Implement or refactor

1. Replace device checks with container space: size classes, `ViewThatFits`, `AnyLayout`, adaptive grids, `containerRelativeFrame` / `onGeometryChange`.
2. Prefer `NavigationSplitView` for collection → selection → detail. Prefer adaptive `TabView` (`.sidebarAdaptable`) over a hand-built sidebar.
3. Prefer system `.toolbar` / `ToolbarItem` / `ToolbarOverflowMenu` so bars can become vertical.
4. Keep one view hierarchy; hoist navigation, scroll, selection, editor, and playback state above layout.
5. Use extra width for panes, inspectors, columns, and persistent navigation — not longer text lines. Cap readable content (e.g. `.frame(maxWidth: 700)`).
6. Move fold-sensitive controls locally. Do not rebuild the whole screen because a reserved region appeared.
7. Use `ArrangementView` only for one two-part experience (player + playlist, editor + inspector). Do not use it as app navigation.
8. Use `onHingeChange` / hinge angle only for physical interaction. **Layout reacts to space. Interaction may react to hinge state.**
9. For the outer display, use scene accessories. Do not target `innerDisplay` / `outerDisplay` as independent canvases.

### 3) Untestable Duo behavior

Until the project can be tested with the iPhone Duo simulator in Xcode 27.1:

The agent **may**:

- improve general adaptability
- remove device assumptions
- adopt standard adaptive containers
- prepare code boundaries for Duo-specific APIs
- identify likely fold-sensitive UI

The agent **should avoid**:

- hardcoding predicted hinge coordinates
- guessing exact Duo dimensions
- adding untested Duo-only layout branches
- restructuring working screens around assumptions

**If Duo-specific behavior cannot yet be verified in the simulator, prefer preparation over speculative implementation.**

---

## Agent decision tree

When reviewing a SwiftUI screen, evaluate it in this order.

1. **Does the layout work across continuously changing widths?**  
   If no: fix the general adaptive layout first.

2. **Is navigation manually switching between phone and tablet implementations?**  
   If yes: investigate `NavigationSplitView`, adaptive `TabView`, or another system navigation container.

3. **Are there fixed widths or screen-size assumptions?**  
   If yes: replace them with container-relative layout where possible.

4. **Does the wide layout simply stretch?**  
   If yes: look for a sidebar, detail pane, inspector, additional columns, or supplementary content.

5. **Could important content intersect the fold?**  
   If yes: use reserved-region-aware positioning.

6. **Are two related views being manually rearranged across layouts?**  
   If yes: evaluate `ArrangementView`, `ViewThatFits`, or `AnyLayout`.

7. **Is there a custom toolbar or tab bar?**  
   If yes: determine whether SwiftUI system bars can replace it and adapt vertically.

8. **Does the requested behavior genuinely depend on physical hinge position?**  
   If no: do not use the hinge API.  
   If yes: use `onHingeChange` and hinge state/angle.

9. **Would the outer display provide useful supplementary information?**  
   If yes: evaluate a scene accessory. Do not manually target a second screen.

---

## Agent rules

These are strict. Full explanations and code are in [reference.md](reference.md).

1. When encountering device-specific layout logic, first attempt to replace it with layout behavior based on available container space.
2. Prefer semantic adaptive containers over manual breakpoints. Order: system navigation/container → adaptive grid → `ViewThatFits` → size classes → exact geometry last.
3. If a screen contains master/detail navigation, strongly prefer `NavigationSplitView` over a manually constructed `HStack` sidebar.
4. Do not stretch narrow interfaces indefinitely. Use extra space to increase information density or introduce complementary panes.
5. If repeated content appears in a grid, prefer minimum-item-width-driven adaptive columns (`GridItem(.adaptive(minimum:))`).
6. Use `ViewThatFits` when the question is “which arrangement fits?” rather than “which device am I on?”
7. Prefer changing layout containers (`AnyLayout`) over conditionally rebuilding separate view hierarchies.
8. Never use `UIScreen.main.bounds` for SwiftUI layout unless there is an exceptional non-layout requirement.
9. Avoid placing important interactive or semantic content across an active division region. Decorative backgrounds and scrolling content may usually cross the fold.
10. Respond to fold interference locally before making a global structural change.
11. Use `ArrangementView` only when the two views form one adaptive experience; do not substitute it for app navigation.
12. Before creating a custom toolbar, verify that the design cannot be expressed with SwiftUI's toolbar APIs.
13. Primary toolbar actions must remain visible; secondary actions may move into overflow as available bar space decreases.
14. If tabs represent primary app sections and the wide layout benefits from persistent navigation, prefer SwiftUI's adaptive sidebar system.
15. Never manually mirror one safe-area inset onto another edge. Duo safe areas can be asymmetric.
16. Treat the fold like the spine of a book: backgrounds may span it; precise content should not.
17. Layout reacts to space. Interaction may react to hinge state. Do not use `hinge.angle` for sidebar, columns, grid count, or navigation collapse.
18. Use scene accessories for intentionally secondary display content; keep the main experience attached to the primary scene.
19. A layout transition must not become an application-state transition.
20. Every expansive layout must have a graceful path back through intermediate widths to compact presentation. Unfolded does not mean wide app.
21. More available width does not imply wider text lines.
22. Wide layouts should reveal more useful structure, not just more whitespace.
23. Optimize for reach and grouping, not maximum geometric distribution.
24. Always complete Tier 1 before proposing Tier 2 or Tier 3 changes.
25. If Duo-specific behavior cannot yet be verified in the simulator, prefer preparation over speculative implementation.

---

## Code-review red flags

Flag these when used for ordinary layout decisions:

```swift
UIScreen.main.bounds
UIDevice.current.userInterfaceIdiom == .pad
if orientation == .landscape
if isIPhoneDuo
if isFolded { ... }
```

Also flag:

- fixed large frames
- hardcoded sidebar widths
- fixed grid column counts
- custom fake toolbars
- content centered directly across the fold
- duplicated compact and expanded state
- layouts tested only at two widths
- excessive full-width text
- controls placed outside safe areas

Preferred replacements are in [reference.md](reference.md#preferred-transformation-examples).

---

## Review checklist

- [ ] No `if isDuo` / idiom / orientation / `UIScreen` layout branches
- [ ] Layout follows container space across narrow → intermediate → wide
- [ ] List/detail uses `NavigationSplitView` with preserved selection
- [ ] Grids use `GridItem(.adaptive(minimum:))` rather than fixed column counts
- [ ] Local rearrangements use `ViewThatFits` or `AnyLayout`, not duplicate hierarchies
- [ ] Wide layout exposes hierarchy (sidebar, detail, inspector, columns) instead of stretching
- [ ] Readable content has a maximum width
- [ ] System toolbars/tabs used; items work horizontally and vertically
- [ ] Important controls stay in (possibly asymmetric) safe areas
- [ ] Critical content is not centered on the fold
- [ ] State (nav, scroll, selection, editors, playback) survives fold/display changes
- [ ] Hinge APIs used only for physical interaction
- [ ] Second display uses scene accessories, not manual display targeting
- [ ] Tier 2/3 APIs added only after Tier 1, and only when testable

---

## Additional resources

- Full 25 rules, API sketches, and before/after transformations: [reference.md](reference.md)
