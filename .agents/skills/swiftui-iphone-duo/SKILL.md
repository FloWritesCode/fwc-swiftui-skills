---
name: swiftui-iphone-duo
description: >-
  Adapts and reviews SwiftUI apps for iPhone Duo (Xcode 27.1 / iOS 27.1 SDK)
  and continuously changing window sizes. Use when supporting the foldable
  iPhone Duo, outer or inner display, vertical bars (toolbarVerticalBehavior,
  axisBehavior, visibilityPriority, ToolbarOverflowMenu), sheet placement,
  reserved regions and the fold, ArrangementView, onHingeChange,
  CameraCaptureAccessory, NavigationSplitView, adaptive TabView sidebar,
  ViewThatFits, AnyLayout, or when replacing UIDevice / UIScreen.main / idiom /
  orientation layout branches.
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
- Code uses `UIDevice`, `UIScreen.main` (bounds or scale), idiom, orientation, `isDuo`, or `isFolded` for layout.
- Work involves vertical bars, toolbar item ordering/overflow, sheet placement, `ArrangementView`, reserved regions, `onHingeChange` / `UIHingeInteraction`, or `CameraCaptureAccessory`.
- The project moves to the iOS 27.1 SDK (that build is what turns on full-screen layout and vertical bars).

## Related skills

| Skill | When to load |
|-------|--------------|
| **swiftui-liquid-glass** (this repo) | System bars and glass chrome on iOS 26+ |
| **App Resizability** (Apple, Xcode 27.1) | First-pass scan for resizing and Duo issues. Export for other agents with `xcrun agent skills export` |

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
reserved regions (reservedRegions(kind:options:))
ArrangementView
vertical bar tuning (axisBehavior, visibilityPriority, compression, toolbarVerticalEdge)
toolbarVerticalBehavior(.disabled), only for the documented exceptions
sheet placement (presentationPlacement)
Duo-specific safe-area handling
```

**Tier 3 — Duo-exclusive experiences** (only when the product benefits):

```text
onHingeChange / UIHingeInteraction
hinge angle
CameraCaptureAccessory (scene accessory, camera apps only)
multi-scene workflows (new windows only on the inner display)
```

---

## Workflow

Copy and track progress:

```
- [ ] 1. Read reference.md (start with "Platform facts")
- [ ] 2. Check the SDK: full-screen layout and vertical bars need an iOS 27.1 SDK build (Xcode 27.1)
- [ ] 3. Review screens with the decision tree
- [ ] 4. Flag red-flag patterns with file evidence
- [ ] 5. Classify each change as Tier 1 / 2 / 3
- [ ] 6. Implement Tier 1 first
- [ ] 7. Apply Tier 2/3 only when space-based layout is insufficient; gate 27.1 APIs with #available
- [ ] 8. Keep state above adaptive layout; verify continuity across widths and every pose in Device Hub
```

### 1) Review existing UI

Walk every SwiftUI screen with the decision tree below. Flag:

- Device identity used as a layout switch (`UIDevice`, idiom, model, `isDuo`, `isFolded`)
- `UIScreen.main` (bounds, scale, stored screens) or orientation used for ordinary layout
- `horizontalSizeClass == .regular` treated as "iPad"
- Size read once (at launch or in `viewIsAppearing`) and never re-read
- Fixed column counts, hardcoded sidebar widths, arbitrary `if width > N`
- Separate compact vs expanded view hierarchies with duplicated state
- Custom toolbars / tab bars that cannot move to a vertical edge
- Toolbar items with a title but no icon, or a custom "..." menu instead of the system overflow menu
- Critical content that would sit on the fold
- Wide layouts that only stretch instead of exposing hierarchy
- Hinge angle used to decide sidebar, columns, or navigation

### 2) Implement or refactor

1. Replace device checks with container space: size classes, `ViewThatFits`, `AnyLayout`, adaptive grids, `containerRelativeFrame` / `onGeometryChange`.
2. Prefer `NavigationSplitView` for collection → selection → detail. Prefer adaptive `TabView` over a hand-built sidebar: `.tabViewStyle(.sidebarAdaptable)` plus `.defaultTabBarPlacement(.sidebar)` opts into the sidebar on the inner display (on iPhone, `.sidebarAdaptable` alone shows a tab bar).
3. Prefer system `.toolbar` / `ToolbarItem` / `ToolbarOverflowMenu` inside a `NavigationStack` / `NavigationSplitView` so bars can become vertical. Give every item a title and an icon; put Close (`.cancellationAction`) first and Done (`.topBarPinnedTrailing`) next; set `visibilityPriority`.
4. Keep one view hierarchy; hoist navigation, scroll, selection, editor, and playback state above layout.
5. Use extra width for panes, inspectors, columns, and persistent navigation — not longer text lines. Cap readable content (e.g. `.frame(maxWidth: 700)`).
6. Move fold-sensitive controls locally. Do not rebuild the whole screen because a reserved region appeared.
7. Use `ArrangementView` only for one two-part experience (player + playlist, editor + inspector). Do not use it as app navigation.
8. Use `onHingeChange` / hinge angle only for physical interaction. **Layout reacts to space. Interaction may react to hinge state.** For fold-aware layout, use reserved regions or `ArrangementView`.
9. Do not target `innerDisplay` / `outerDisplay` as independent canvases. The only supported outer-display content is a `CameraCaptureAccessory` during an active camera capture session.
10. Check every sheet in every pose; choose placement with `presentationPlacement(_:)`.

### 3) Untestable Duo behavior

Xcode 27.1 includes the iPhone Duo simulator (Device Hub: open, close, rotate, fold). It can't run most app extensions and has no camera, so camera capture accessories need a device.

Until the specific behavior can be checked in the simulator or on a device:

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

**If Duo-specific behavior cannot yet be verified in the simulator or on a device, prefer preparation over speculative implementation.**

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

7. **Is there a custom toolbar or tab bar, or toolbar items without an icon?**  
   If yes: determine whether SwiftUI system bars can replace it and adapt vertically, and give every item a title and an icon.

8. **Does the requested behavior genuinely depend on physical hinge position?**  
   If no: do not use the hinge API.  
   If yes: use `onHingeChange` (UIKit: `UIHingeInteraction`) and `hinge.status` / `hinge.angle`.

9. **Would the outer display provide useful supplementary information during camera capture?**  
   If yes: evaluate a `CameraCaptureAccessory`. Otherwise, do not try to target the second screen.

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
8. Never use `UIScreen.main` (bounds, scale, or a stored reference) for layout. Apple discourages it and plans to deprecate main-screen references.
9. Avoid placing important interactive or semantic content across an active division region. Decorative backgrounds and scrolling content may usually cross the fold.
10. Respond to fold interference locally before making a global structural change.
11. Use `ArrangementView` only when the two views form one adaptive experience; do not substitute it for app navigation.
12. Before creating a custom toolbar, verify that the design cannot be expressed with SwiftUI's toolbar APIs.
13. Primary toolbar actions must remain visible; secondary actions may move into overflow as available bar space decreases.
14. If tabs represent primary app sections and the wide layout benefits from persistent navigation, prefer SwiftUI's adaptive sidebar system (`.sidebarAdaptable` + `.defaultTabBarPlacement(.sidebar)`).
15. Never manually mirror one safe-area inset onto another edge. Duo safe areas can be asymmetric.
16. Treat the fold like the spine of a book: backgrounds may span it; precise content should not.
17. Layout reacts to space. Interaction may react to hinge state. Do not use `hinge.angle` for sidebar, columns, grid count, or navigation collapse.
18. Use `CameraCaptureAccessory` only for supplementary content during camera capture; keep the main experience attached to the primary scene.
19. A layout transition must not become an application-state transition.
20. Every expansive layout must have a graceful path back through intermediate widths to compact presentation. Unfolded does not mean wide app.
21. More available width does not imply wider text lines.
22. Wide layouts should reveal more useful structure, not just more whitespace.
23. Optimize for reach and grouping, not maximum geometric distribution.
24. Always complete Tier 1 before proposing Tier 2 or Tier 3 changes.
25. If Duo-specific behavior cannot yet be verified in the simulator or on a device, prefer preparation over speculative implementation.
26. Check every sheet in every pose; set placement deliberately instead of building custom presentations.
27. Never call a 27.1 Duo API without an availability check that matches the project's deployment target.
28. Every fill-mode image or video must keep its important content visible from the narrow outer display to the wide inner display.

---

## Code-review red flags

Flag these when used for ordinary layout decisions:

```swift
UIScreen.main.bounds
UIScreen.main.scale
UIWindow(frame: UIScreen.main.bounds)
UIDevice.current.userInterfaceIdiom == .pad
UIDevice.current.orientation / statusBarOrientation / interfaceOrientation
if orientation == .landscape
if horizontalSizeClass == .regular { iPadLayout }
bounds.height == 844
view.bounds.width - view.safeAreaInsets.left * 2
if isIPhoneDuo
if isFolded { ... }
```

Also flag:

- fixed large frames
- hardcoded sidebar widths
- fixed grid column counts
- custom fake toolbars (or custom `UIToolbar` / `UINavigationBar` / `UITabBar` instances)
- title-only toolbar items (they never go vertical)
- custom "..." overflow menus
- content centered directly across the fold
- duplicated compact and expanded state
- layouts tested only at two widths
- excessive full-width text
- controls placed outside safe areas
- `.scaleAspectFill` / `.aspectRatio(contentMode: .fill)` hero media without a focal point
- `toolbarVerticalBehavior(.disabled)` outside the documented exceptions, or toggled per view state

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
- [ ] App builds with the iOS 27.1 SDK; 27.1 APIs are availability-gated
- [ ] System toolbars/tabs used; items work horizontally and vertically
- [ ] Every toolbar item has a title and an icon; Close/Back first, then Done; overflow priorities set
- [ ] Important controls stay in (possibly asymmetric) safe areas
- [ ] Critical content is not centered on the fold
- [ ] State (nav, scroll, selection, editors, playback) survives fold/display changes
- [ ] Sheets checked in every pose; placement set where needed
- [ ] Split View checked with the app on the left and on the right
- [ ] Hinge APIs used only for physical interaction
- [ ] Outer display content only via `CameraCaptureAccessory`, not manual display targeting
- [ ] Tier 2/3 APIs added only after Tier 1, and only when testable

---

## Additional resources

- Full 28 rules, platform facts, API reference, and before/after transformations: [reference.md](reference.md)
- Apple: [Preparing your app for iPhone Duo](https://developer.apple.com/documentation/technologyoverviews/preparing-your-app-for-iphone-duo), [Designing for iPhone Duo](https://developer.apple.com/design/human-interface-guidelines/designing-for-iphone-duo), [Three steps to make your app shine on iPhone Duo](https://developer.apple.com/iphone-duo/prepare/)
