# iPhone Duo adaptation rules

Detailed rulebook for `swiftui-iphone-duo`. Read this before reviewing or adapting a screen.

## Core principle

**Do not design a separate “Duo version” of the app.**

Start with an adaptive SwiftUI interface that works across continuously changing widths and heights. Add Duo-specific behavior only when the fold, hinge, second display, or vertical system bars materially improve the experience.

Optimize in this order:

1. Adaptive layout
2. Adaptive navigation
3. Adaptive toolbars and tabs
4. Fold-safe positioning
5. Duo-specific arrangements
6. Hinge-driven interactions
7. Second-display experiences

---

## Platform facts (Xcode 27.1 / iOS 27.1)

Checked against Apple's "Preparing your app for iPhone Duo", the HIG page "Designing for iPhone Duo", the iPhone Duo Tech Talks, and the Xcode 27.1 release notes.

- **The SDK is the opt-in.**
  - iOS 26 SDK or earlier: the app runs, but sits centered with empty space around it on the open inner display, and to the left of the status bar and camera on the outer display.
  - iOS 27 SDK: the app resizes to fill most of the inner display, but still avoids the status bar on the right edge.
  - iOS 27.1 SDK or later: the app uses the full display, and toolbars and tab bars move into a vertical bar below the status bar. There is no separate switch. Before shipping a 27.1 build, make sure content adapts to vertical bars.
- **Size classes per display.** Outer display: compact width and regular height in portrait, compact in both in landscape (like other iPhones). Inner display: regular width and regular height. Apple: an iPhone app "can have any combination of size classes."
- **Orientation is not shape.** The inner display doesn't honor your supported interface orientations.
- **The main screen is ambiguous.** Apple discourages `UIScreen.main` and says main-screen references will be deprecated in a future release.
- **Tooling.** Xcode 27.1 requires macOS Tahoe 26.6 or later and runs the iPhone Duo simulator in Device Hub (open, close, rotate, fold). Known issues: the first launch can take several minutes, StandBy is unavailable, and most app extensions can't run or be debugged in the Duo simulator runtime. Simulator has no camera.
- **Apple's skill.** Xcode 27.1's App Resizability skill (ask the coding assistant to "get my app ready for iPhone Duo") scans for the patterns in this rulebook. Other agents can export it with `xcrun agent skills export`. Apple says it "finds most issues, but not all of them." Suggest it to the user, then use this rulebook for what it misses and for design decisions.

---

# Rule 1 — Design for available space, not device identity

Never make layout decisions primarily from:

```swift
UIDevice.current.userInterfaceIdiom
UIScreen.main.bounds
device model
interface orientation  // UIDevice.current.orientation, statusBarOrientation,
                       // interfaceOrientation, effectiveGeometry.interfaceOrientation
```

Do not write logic conceptually equivalent to:

```swift
if isDuo {
    duoLayout
}
```

or:

```swift
if deviceIsUnfolded {
    wideLayout
}
```

The same iPhone Duo can present the app:

- on the outer display
- fullscreen on the inner display
- beside another app
- in different fold states
- with different safe-area and toolbar configurations

Prefer:

```swift
@Environment(\.horizontalSizeClass)
@Environment(\.verticalSizeClass)
```

and container-level geometry.

Two Duo-specific traps:

- **"Regular width" no longer means iPad.** The inner display is regular × regular, so code that shows an iPad-only layout when `horizontalSizeClass == .regular` now runs on iPhone. Apple: on the inner display your app should look like a natural extension of its iPad layout. Make that layout work on iPhone instead of adding an idiom check.
- **Orientation doesn't tell you the shape.** To know whether the app is wider than it is tall, compare the width and height of the space you have (`GeometryReader` / `onGeometryChange` in SwiftUI, `view.bounds` in UIKit).

### Agent rule

**When encountering device-specific layout logic, first attempt to replace it with layout behavior based on available container space.**

---

# Rule 2 — Prefer semantic adaptive containers over manual breakpoints

Use SwiftUI containers that already know how to adapt.

Preferred tools:

```swift
NavigationSplitView
NavigationStack
TabView
List
Grid
LazyVGrid
ViewThatFits
AnyLayout
containerRelativeFrame
```

Avoid arbitrary thresholds such as:

```swift
if width > 700
```

unless the design genuinely requires a specific minimum usable width.

### Preferred order

When solving an adaptive-layout problem:

1. Try a system navigation/container API.
2. Try an adaptive grid or layout.
3. Try `ViewThatFits`.
4. Try size classes.
5. Use exact geometry only when necessary.

---

# Rule 3 — Use `NavigationSplitView` for list/detail interfaces

If the information architecture naturally contains:

```text
collection → selection → detail
```

prefer:

```swift
NavigationSplitView
```

rather than maintaining separate compact and expanded navigation trees.

Expected behavior:

```text
Narrow:
List → Detail

Wide:
List | Detail
```

The app should preserve navigation and selection state when moving between these configurations.

### Agent rule

**If a screen contains master/detail navigation, strongly prefer `NavigationSplitView` over a manually constructed `HStack` sidebar.**

---

# Rule 4 — Make wide layouts meaningfully better, not merely wider

When additional width becomes available, ask whether the app can expose information or controls simultaneously.

Good transformations include:

```text
List → List + Detail
Editor → Editor + Inspector
Player → Player + Queue
Preview → Preview + Controls
1-column grid → 2–3-column grid
Bottom tabs → Sidebar navigation
```

Bad transformation:

```text
300pt-wide form → 900pt-wide form
```

Use maximum readable/content widths where appropriate.

Example:

```swift
.frame(maxWidth: 700)
```

or container-relative sizing.

### Agent rule

**Do not stretch narrow interfaces indefinitely. Use extra space to increase information density or introduce complementary panes.**

---

# Rule 5 — Prefer adaptive grids over fixed column counts

For card, media, dashboard, or gallery interfaces, prefer:

```swift
GridItem(.adaptive(minimum: ...))
```

over:

```swift
if wide {
    3 columns
} else {
    1 column
}
```

The layout should naturally support intermediate widths, including side-by-side multitasking.

### Agent rule

**If repeated content appears in a grid, prefer minimum-item-width-driven adaptive columns.**

---

# Rule 6 — Use `ViewThatFits` for local layout decisions

For components that can sensibly appear in multiple orientations:

```swift
ViewThatFits(in: .horizontal) {
    HStack {
        PrimaryView()
        SecondaryView()
    }

    VStack {
        PrimaryView()
        SecondaryView()
    }
}
```

Prefer this over manually inspecting the screen width.

Good use cases:

- buttons
- metadata
- editor controls
- cards
- preview/detail components
- compact toolbars

### Agent rule

**Use `ViewThatFits` when the question is “which arrangement fits?” rather than “which device am I on?”**

---

# Rule 7 — Use `AnyLayout` when the same views should rearrange

If the interface contains the same semantic views but changes their spatial relationship, prefer:

```swift
AnyLayout(HStackLayout())
AnyLayout(VStackLayout())
```

This helps preserve view identity and state as layout changes.

Use it for:

- editor + controls
- chart + legend
- preview + properties
- content + supplementary information

### Agent rule

**Prefer changing layout containers over conditionally rebuilding separate view hierarchies.**

---

# Rule 8 — Think in containers, not screens

Views should normally size themselves relative to their immediate usable container.

Prefer:

```swift
containerRelativeFrame
GeometryReader
onGeometryChange
```

over global screen dimensions.

A view may occupy only one portion of the inner display, so physical display width is rarely the correct input.

When replacing `UIScreen.main`:

```swift
// Scale: read it from the environment or trait collection
@Environment(\.displayScale) private var displayScale   // SwiftUI
let scale = traitCollection.displayScale                // UIKit

// UIKit windows: create them from a scene,
// not with UIWindow(frame: UIScreen.main.bounds)
let window = UIWindow(windowScene: windowScene)

// If you truly need the screen, look it up when you need it; don't store it
let screen = view.window?.windowScene?.screen
```

Re-read size whenever it changes. A fold can resize the app mid-session, so a value captured at launch or in `viewIsAppearing` goes stale. In UIKit, do size-dependent layout in `layoutSubviews` / `viewDidLayoutSubviews`, and use `viewWillTransition(to:with:)` for work that must run when the size changes.

### Agent rule

**Never use `UIScreen.main` (bounds, scale, or a stored reference) for layout. Apple discourages it and plans to deprecate main-screen references.**

---

# Rule 9 — Treat the fold as a reserved region, not a breakpoint

For custom interfaces where important content may intersect the physical fold, inspect Duo's reserved regions.

Use reserved regions when positioning:

- critical controls
- faces or focal image content
- text that must remain uninterrupted
- QR codes
- draggable items
- custom canvases

Do not use fold geometry for ordinary system layouts that already adapt correctly. Alerts, context menus, sheets, popovers, and split views already move away from the fold.

API (iOS 27.1):

```swift
GeometryReader { proxy in
    let fold = proxy.reservedRegions(kind: .division)        // the fold
        .filter(\.isActive)
    let cameras = proxy.reservedRegions(kind: .occlusion)    // front cameras
        .filter(\.isActive)
    // Position custom content using each region's `frame` (includes `margins`).
}
```

- **Kinds:** `.division` (the fold, active only when the device is partially folded; inactive and zero-width when flat) and `.occlusion` (the outer front camera, always active; the inner front camera, active only while in use).
- **Options:** `.includeInactive`. Inactive regions still help with high-level decisions, such as preferring an even number of grid columns whenever a division region exists.
- Filter on `isActive` explicitly rather than relying on the default query behavior.
- The same proxy is available from `onGeometryChange`. UIKit: `view.reservedRegions(kind:options:)` returns `[UIView.ReservedRegion]`.
- SwiftUI mirrors region geometry for right-to-left layouts by default. Pass `layoutDirectionBehavior: .fixed` only if you mirror manually.

### Agent rule

**Avoid placing important interactive or semantic content across an active division region.**

Decorative backgrounds and continuously scrolling content may usually cross the fold.

---

# Rule 10 — Displace important content instead of redesigning everything

When the fold interferes with one element, move that element.

Do not completely restructure an otherwise good layout just because a fold region appears.

Preferred:

```text
Before:

[ Content          Important Action ]

Fold active:

[ Content ] |fold| [ Important Action ]
```

Avoid:

```text
Entire screen changes into an unrelated layout
```

Move related elements together so they keep their relationship, and avoid moving an element far from its source. Continuously scrolling content (articles, feeds, documents, lists) doesn't displace; it already adapts by scrolling.

### Agent rule

**Respond to fold interference locally before making a global structural change.**

---

# Rule 11 — Use `ArrangementView` for genuinely two-part experiences

Use `ArrangementView` when two related pieces of content should intelligently reorganize based on:

- available space
- aspect ratio
- size class
- Duo division regions

Examples:

```text
Player + Playlist
Editor + Inspector
Preview + Controls
Canvas + Properties
Document + Metadata
```

API (iOS 27.1). Use the split style (the default) when both views deserve dedicated space and neither should be obscured, such as a main/detail relationship. It splits side by side when wider than tall, and stacks when taller than wide:

```swift
ArrangementView {
    PrimaryView()
} secondary: {
    SecondaryView()
}
.arrangementViewStyle(.split)
```

Restrict the axes when needed. If the split can't use its primary axis, the arrangement shows only one view:

```swift
.arrangementViewStyle(.split.axes(.horizontal))
```

Use the overlay style when there is a clear foreground/background relationship (controls over content). It layers the primary view over the secondary view, and moves them to either side of the fold when the device is partially open:

```swift
.arrangementViewStyle(.overlay)
```

Related APIs: `overlayArrangementEdge(_:)`, `splitArrangementLayoutRatio(_:)`, and the `splitArrangementAxis` / `overlayArrangementZIndex` environment values. UIKit: `UIArrangementViewController` with `setViewController(_:for:animated:)` and `updateArrangement(_:animated:)` (`UISplitArrangement`, `UIOverlayArrangement`).

If an `HStack` / `VStack` layout already looks like this, it maps to split. A `ZStack` maps to overlay.

### Agent rule

**Use `ArrangementView` only when the two views form one adaptive experience; do not substitute it for app navigation.**

Keep navigation containers outside it (a `NavigationStack` around an `ArrangementView` is Apple's own example). Don't place it inside a `NavigationSplitView`, `List`, `ScrollView`, or other container that could make part of it inaccessible.

---

# Rule 12 — Let system navigation and bars adapt vertically

On Duo, navigation bars, toolbars, and tab bars move into one vertical bar on the side: always on the closed outer display, and on the inner display in landscape. The inner display in portrait keeps horizontal bars. This only happens when the app is built with the iOS 27.1 SDK.

Bars only move when they come from a navigation container. Attach `.toolbar` to content inside a `NavigationStack` or `NavigationSplitView`, and use `TabView`. In UIKit, set items on a view controller inside a `UINavigationController` / `UITabBarController`. Content from custom `UIToolbar`, `UINavigationBar`, or `UITabBar` instances is ignored.

Use:

```swift
.toolbar
ToolbarItem
ToolbarItemGroup
ToolbarOverflowMenu
```

Container rules:

- In a split view, only the detail column gets a vertical bar. Sidebar and content columns stay horizontal.
- Inspectors keep horizontal bars.
- The vertical bar is aligned with the hardware, so it stays on the same side in right-to-left languages.
- To position custom UI relative to the bar, read `@Environment(\.toolbarVerticalEdge)` (`HorizontalEdge?`, `nil` where no vertical bar is ever used). UIKit: `traitCollection.verticalBarEdge`.
- Let hero or background images extend under the vertical bar with `backgroundExtensionEffect()` (UIKit: `UIBackgroundExtensionView`).

Opting out (iOS 27.1):

```swift
NavigationStack {
    CalculatorView()
        .toolbarVerticalBehavior(.disabled)   // .automatic is the default
}
```

UIKit: override `preferredVerticalBarBehavior` and return `.disabled`. Apple's guidance is to keep the default in general. Opt out only for UIs better served by horizontal bars, like a fullscreen video player, a non-scrolling layout like Calculator, or a sheet with a single Close button. Treat it as a stable choice. Don't toggle it as the user navigates or as a function of one view's state. To hide bars on one screen, use `toolbarVisibility(_:for:)` instead.

The app should remain usable whether controls are arranged horizontally or vertically.

### Agent rule

**Before creating a custom toolbar, verify that the design cannot be expressed with SwiftUI's toolbar APIs.**

---

# Rule 13 — Design toolbar items for both horizontal and vertical presentation

Toolbar items should:

- favor symbols where appropriate
- avoid unnecessarily long labels
- group secondary actions
- prioritize important actions
- remain recognizable when stacked vertically

**Give every item both a title and an icon** (for example, `Button("Share", systemImage: "square.and.arrow.up")` or a `Label`). The system picks the representation:

- vertical bar: icon
- horizontal bar: icon or title, preferring the icon
- overflow menu: icon and title
- an item with a title and no icon is never presented vertically
- an item with a custom view stays horizontal unless you opt it in with `.axisBehavior(.verticalPreferred)`

**Order matters.** The top of the vertical bar is reserved for Back or Close, then prominent actions like Done. Remaining items keep their groupings.

```swift
NavigationStack {
    DetailView()
        .toolbar {
            ToolbarItem(placement: .cancellationAction) {     // top: Close
                Button("Close", systemImage: "xmark") { dismiss() }
            }
            ToolbarItem(placement: .topBarPinnedTrailing) {   // then: Done
                Button("Done", systemImage: "checkmark") { save() }
            }
            ToolbarItem(placement: .bottomBar) {
                Button("Share", systemImage: "square.and.arrow.up") { share() }
            }
            .visibilityPriority(.high)                         // overflows last
            ToolbarItem {
                SelectOrDoneButton()                           // switches symbol <-> text
            }
            .axisBehavior(.horizontalOnly)
            ToolbarOverflowMenu {                              // always in the overflow menu
                Button("Duplicate", systemImage: "plus.square.on.square") { duplicate() }
            }
        }
}
```

APIs:

- `.visibilityPriority(_:)` with `.automatic` (default), `.low`, `.high`, or `init(lowerThan:)` / `init(higherThan:)`. Items overflow from bottom to top by default. Prioritize whole groups first, then items within a group. Keep frequent actions (Compose, New Note) and badged items visible longest. UIKit: `UIBarButtonItem.visibilityPriority`.
- `.topBarPinnedTrailing`: pinned items only move into overflow when search is active and space runs out. UIKit: `navigationItem.pinnedTrailingGroup`.
- `.axisBehavior(_:)` with `.automatic`, `.horizontalOnly` (hidden if no horizontal bar is present), or `.verticalPreferred`. Use `.horizontalOnly` for items that switch between symbol and text; the system Edit button already does this. UIKit: `UIBarButtonItem.axisBehavior`.
- `ToolbarOverflowMenu { ... }`: move your own "..." menu into the system overflow menu. Reserve the ellipsis for overflow and give other menus a distinct symbol. UIKit: `navigationItem.additionalOverflowItems`.
- `.toolbarVerticalCompressionBehavior(_:)` with `.automatic`, `.prefersToolbarItems`, or `.prefersTabBar`. The default compresses toolbar items first so tabs stay visible, which suits navigation-focused screens. Use `.prefersToolbarItems` on task-oriented screens. UIKit: `navigationItem.verticalBarCompressionBehavior` (`.prefersBarItems`, `.prefersTabBar`).
- Use `.badge(_:)` instead of inline counts so a text + symbol item becomes symbol-only.
- Flexible spacers have zero size in the vertical axis. Don't add manual spacing; use `ToolbarItemGroup` instead.

Use vertical behavior APIs rather than building a second toolbar.

### Agent rule

**Primary actions must remain visible; secondary actions may move into overflow as available bar space decreases.**

---

# Rule 14 — Prefer adaptive sidebar navigation where it improves wide layouts

For tab-based apps, consider adaptive sidebar presentation on the inner display.

On iPhone, `.sidebarAdaptable` resolves to a bottom tab bar on its own. To opt into a sidebar where it's supported (Apple's Duo guidance: the inner display), add `defaultTabBarPlacement(.sidebar)` (iOS 27):

```swift
TabView {
    Tab("Home", systemImage: "house") { HomeView() }
    Tab("Library", systemImage: "books.vertical") { LibraryView() }
}
.tabViewStyle(.sidebarAdaptable)
.defaultTabBarPlacement(.sidebar)
```

- UIKit: `tabBarController.sidebar.preferredPlacement = .sidebar`.
- `defaultAdaptableTabBarPlacement(_:)` only affects iPadOS. Don't use it for Duo.
- Inside tab content, read `isTabViewSidebarAvailable` to gate sidebar-dependent UI rather than inspecting size classes. `tabBarPlacement` reports the current placement.

Do not manually reproduce a sidebar based solely on width.

### Agent rule

**If tabs represent primary app sections and the wide layout benefits from persistent navigation, prefer SwiftUI's adaptive sidebar system.**

---

# Rule 15 — Respect asymmetric safe areas

Do not assume:

```text
left inset == right inset
top inset == bottom inset
```

Duo can have asymmetric safe areas caused by:

- cameras
- vertical system bars, which can sit on the leading or trailing edge (for example in landscape or Split View)
- multitasking

Layout margins are asymmetric too. The fold is not a safe-area inset; handle it with reserved regions (Rule 9).

Keep important controls and readable content inside safe areas.

```swift
// Avoid: assumes both sides are equal
let width = view.bounds.width - view.safeAreaInsets.left * 2

// Better: handle each side independently
let width = view.bounds.inset(by: view.safeAreaInsets).width
```

Decorative backgrounds may extend using:

```swift
.ignoresSafeArea()
```

when appropriate. To fit custom UI to the screen corners, use `ConcentricRectangle` (SwiftUI) or `UICornerConfiguration` (UIKit) instead of hardcoded radii.

### Agent rule

**Never manually mirror one safe-area inset onto another edge.**

---

# Rule 16 — Do not center critical content across the hinge

Avoid positioning the following directly across the fold:

- buttons
- text
- faces
- QR codes
- input controls
- draggable handles
- important icons
- small visual details

Background photography, gradients, textures, and scrolling content may cross the hinge when visual interruption is acceptable.

### Agent rule

**Treat the fold like the spine of a book: backgrounds may span it; precise content should not.**

---

# Rule 17 — Use hinge state only when the hinge itself is meaningful

Duo exposes hinge information, including fold status and angle.

API (iOS 27.1):

```swift
.onHingeChange { _, newContext in
    // A nil hinge means the device doesn't have one
    if let hinge = newContext.hinge, hinge.status == .partiallyOpen {
        pitchBend = calculatePitchBend(angle: hinge.angle)   // `angle` is an Angle
    } else {
        pitchBend = 0
    }
}
```

- `onHingeChange(isEnabled:_:)`: the action receives the old and new `DeviceHingeContext`. `isEnabled` defaults to `true`.
- `DeviceHingeContext.hinge` is a `DeviceHinge?` (`nil` on devices without a hinge).
- `DeviceHinge.angle` is an `Angle`. `DeviceHinge.status` is `.closed`, `.partiallyOpen`, or `.fullyOpen`.
- UIKit: add a `UIHingeInteraction` to a view. The handler's `update.hinge` is a `UIHinge?` (`nil` when the interaction leaves a hierarchy that provides hinge updates). `UIHinge.angle` is in radians, and `UIHinge.Status` also has `.unknown`.

Apple: hinge data is observed live and is ideal for driving interactions or effects. For layout, use the arrangement and reserved-region APIs.

Use hinge angle for experiences such as:

- physical input
- game mechanics
- instrument controls
- camera positioning
- tabletop interactions
- effects tied directly to device posture

Do not use:

```swift
hinge.angle
```

to decide ordinary things such as:

- whether to display a sidebar
- whether to use two columns
- how many grid items fit
- whether navigation should collapse

### Agent rule

**Layout reacts to space. Interaction may react to hinge state.**

This should be treated as a strict rule.

---

# Rule 18 — Do not manually manage the two physical displays

Do not assume the app should access:

```text
innerDisplay
outerDisplay
```

as independent canvases. The main scene moves between displays as the device opens and closes.

The supported way to show supplementary content on the outer display while the app runs on the inner display is a **camera capture accessory** (iOS 27.1):

```swift
CameraView(model: model)
    .sceneAccessory {
        CameraCaptureAccessory(isEnabled: $model.isTeleprompterEnabled) {
            TeleprompterView(model: model)
        }
        .onAvailabilityChange { isAvailable in
            model.isTeleprompterAvailable = isAvailable
        }
    }
```

- The system presents it only while the device is open, the app is in the foreground and full screen on the inner display, and a camera capture session is active. The system decides when and where, and can withdraw the content at any time.
- Register it on the same view that shows the capture UI. Keep every essential control in the main UI. The content can be interactive, but keep interaction minimal.
- Share state through the same observable model. Turn content off with the `isEnabled` binding rather than unregistering.
- UIKit: `UISceneAccessory.cameraCapture(sceneConfiguration:)` registered with `registerSceneAccessory(_:)`. The accessory's scene uses the `.windowCameraCaptureAccessory` session role.
- Test on a device. Simulator has no camera.
- `ExternalNonInteractiveAccessory` targets connected external displays and AirPlay, not the Duo outer display.

Use cases (camera apps):

- teleprompter
- countdown
- showing the subject what the camera sees
- something fun for a child while taking their photo

### Agent rule

**Use `CameraCaptureAccessory` only for supplementary content during camera capture; keep the main experience attached to the primary scene. Don't try to target the outer display any other way.**

---

# Rule 19 — Support state continuity across fold and display transitions

Opening or closing Duo should not unexpectedly reset:

- navigation
- scroll position
- editor contents
- playback
- selection
- form state
- unsaved work

Avoid implementing completely independent folded and unfolded view hierarchies that create separate sources of state.

Keep state above adaptive layout decisions where appropriate.

### Agent rule

**A layout transition must not become an application-state transition.**

---

# Rule 20 — Treat side-by-side multitasking as a first-class layout

Do not optimize solely for:

```text
outer display
vs.
full inner display
```

Duo can expose intermediate app widths.

Test and design along a continuum:

```text
very narrow → compact → medium → wide
```

Do not assume that “unfolded” means “wide app.”

All apps take part in Split View multitasking on the inner display, and Duo also stacks video and apps together. Each app puts its controls along its outer edge, so the vertical bar can be on either side of your app. Test your app on the left and on the right in Device Hub.

Duo is the first iPhone to support multiple scenes. Apps that support multiple windows on iPad get this on Duo too, but new windows can't be created on the outer display. Handle errors when requesting new scenes, and use `UIWindowScene.ActivationAction`, which hides itself when new windows aren't available.

`UIRequiresFullScreen` is still honored, but the app still resizes when the device opens or closes.

### Agent rule

**Every expansive layout must have a graceful path back through intermediate widths to compact presentation.**

---

# Rule 21 — Keep readable content from becoming excessively wide

Text-heavy content should normally have a sensible maximum width.

Examples:

- articles
- settings forms
- onboarding screens
- login screens
- long descriptions

Prefer centered or contextual columns over edge-to-edge stretching.

For example:

```swift
TextContent()
    .frame(maxWidth: 700)
```

### Agent rule

**More available width does not imply wider text lines.**

---

# Rule 22 — Use extra width to expose hierarchy

When moving from compact to expansive presentation, prioritize:

1. persistent navigation
2. supplementary detail
3. inspectors
4. previews
5. additional controls
6. higher information density

before simply increasing spacing or element size.

### Agent rule

**Wide layouts should reveal more useful structure, not just more whitespace.**

---

# Rule 23 — Preserve touch ergonomics on a larger device

Do not move primary actions to distant corners simply because more screen area exists.

Prefer system toolbar placement and controls near natural interaction regions.

On expansive layouts:

- preserve reasonable touch targets
- avoid spreading frequently used controls excessively
- use vertical system bars when appropriate
- keep related controls grouped
- keep controls near the content they affect (Mail keeps list controls above the list pane, not in the trailing vertical bar)

### Agent rule

**Optimize for reach and grouping, not maximum geometric distribution.**

---

# Rule 24 — Use Duo-specific APIs progressively

The agent should classify changes into three tiers.

## Tier 1 — Universal adaptive improvements

Do these first:

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

## Tier 2 — Duo-aware layout improvements

Only when needed:

```text
reserved regions (reservedRegions(kind:options:))
ArrangementView
vertical bar tuning (axisBehavior, visibilityPriority, compression, toolbarVerticalEdge)
toolbarVerticalBehavior(.disabled), only for the documented exceptions
sheet placement (presentationPlacement)
Duo-specific safe-area handling
```

## Tier 3 — Duo-exclusive experiences

Only when the product benefits:

```text
onHingeChange / UIHingeInteraction
hinge angle
CameraCaptureAccessory (scene accessory, camera apps only)
multi-scene workflows (new windows only on the inner display)
```

### Agent rule

**Always complete Tier 1 before proposing Tier 2 or Tier 3 changes.**

---

# Rule 25 — Do not prematurely optimize for untestable behavior

The iPhone Duo simulator ships with Xcode 27.1 (Device Hub can open, close, rotate, and fold the device). It can't verify everything: camera capture and camera capture accessories need a device, and most app extensions can't run in the Duo simulator runtime.

Until the project can be built with Xcode 27.1 and the specific behavior can be checked in the simulator or on a device:

The agent may:

- improve general adaptability
- remove device assumptions
- adopt standard adaptive containers
- prepare code boundaries for Duo-specific APIs
- identify likely fold-sensitive UI

The agent should avoid:

- hardcoding predicted hinge coordinates
- guessing exact Duo dimensions
- adding untested Duo-only layout branches
- restructuring working screens around assumptions

### Agent rule

**If Duo-specific behavior cannot yet be verified in the simulator or on a device, prefer preparation over speculative implementation.**

---

# Rule 26 — Let sheets and presentations adapt

Sheets, popovers, context menus, and alerts adapt to every pose and move away from the fold automatically.

- On the outer display, sheets with a toolbar present it vertically by default.
- On the inner display, sheets are centered by default with horizontal bars. Leading sheets have no vertical bar; trailing sheets get one.
- Choose the placement with `presentationPlacement(_:)` (iOS 27): `.automatic`, `.center`, `.leading`, `.trailing`. Only sheets respect it. UIKit: `UISheetPresentationController.preferredPlacement`.

```swift
.sheet(item: $selection) { item in
    ItemDetailView(item: item)
        .presentationPlacement(.trailing)
}
```

- A sheet with a single control (such as Close) is a documented case for `toolbarVerticalBehavior(.disabled)`.

### Agent rule

**Check every sheet in every pose; set placement deliberately instead of building custom presentations.**

---

# Rule 27 — Gate Duo APIs by availability

Most Duo APIs are new in iOS 27.1: vertical bar behavior and edge, axis behavior, compression behavior, reserved regions, arrangement views, hinge APIs, and `CameraCaptureAccessory`. `visibilityPriority`, `ToolbarOverflowMenu`, `.topBarPinnedTrailing`, `presentationPlacement`, `defaultTabBarPlacement`, `isTabViewSidebarAvailable`, and `sceneAccessory` are iOS 27.0. `ConcentricRectangle` and `backgroundExtensionEffect()` are iOS 26.

- Wrap them in `if #available(iOS 27.1, *)` (or 27.0) when the deployment target is lower.
- Mac Catalyst (Xcode 27.1 known issue): iOS 27.1-specific APIs fail to compile for Mac Catalyst. Isolate them with `#if !targetEnvironment(macCatalyst)`. If a target with iOS 27.1 shows no Mac Catalyst run destination, add a Mac Catalyst 27.0 minimum deployment.

### Agent rule

**Never call a 27.1 Duo API without an availability check that matches the project's deployment target.**

---

# Rule 28 — Check full-bleed media

Hero images and video set to `.scaleAspectFill` or `.aspectRatio(contentMode: .fill)` can lose important content on wider displays. Use the current size classes or aspect ratio to choose between fill and fit, or set a focal point.

For games: fill the screen as the pose changes. Prefer changing the aspect ratio over letterboxing or pillarboxing.

### Agent rule

**Every fill-mode image or video must keep its important content visible from the narrow outer display to the wide inner display.**

---

# Preferred transformation examples

## Instead of

```swift
if isDuo {
    DuoDashboard()
} else {
    Dashboard()
}
```

Prefer one adaptive hierarchy:

```swift
Dashboard()
```

whose internal layout responds to available space.

---

## Instead of

```swift
if width > 700 {
    HStack {
        Preview()
        Inspector()
    }
} else {
    VStack {
        Preview()
        Inspector()
    }
}
```

Prefer:

```swift
ViewThatFits {
    HStack {
        Preview()
        Inspector()
    }

    VStack {
        Preview()
        Inspector()
    }
}
```

or `AnyLayout` where preserving the same child hierarchy is important.

---

## Instead of

```swift
let columns = isDuo ? 3 : 1
```

Prefer:

```swift
GridItem(.adaptive(minimum: 220))
```

---

## Instead of

```swift
let scale = UIScreen.main.scale
let window = UIWindow(frame: UIScreen.main.bounds)
```

Prefer:

```swift
let scale = traitCollection.displayScale
let window = UIWindow(windowScene: windowScene)
```

---

## Instead of

```swift
ToolbarItem(placement: .topBarTrailing) {
    Button("Share") { share() }        // title only: never goes vertical
}
```

Prefer:

```swift
ToolbarItem(placement: .topBarTrailing) {
    Button("Share", systemImage: "square.and.arrow.up") { share() }
}
```

---

## Instead of

```swift
if hinge.angle > ... {
    showSidebar = true
}
```

Prefer:

```text
available width → navigation/layout decision
hinge angle → physical interaction/effect
```

---

# Shipping notes (only when the user asks about release)

- App Store screenshot sizes for iPhone Duo: outer display 1398 × 2034 px (2034 × 1398 landscape), inner display 2007 × 2853 px (2853 × 2007 landscape).
- Apple: "Starting April 2027, any apps or games submitted will need to include screenshots for iPhone Duo." Apple hasn't announced the exact date.
- App Store Connect has a preview tool to see product page assets on iPhone Duo. Featuring nominations have a Helpful Details section where you can state that the app is optimized for iPhone Duo.

---

# Final skill philosophy

When adapting an app for iPhone Duo, the agent should repeatedly ask:

> Can this improvement be expressed as a generally adaptive SwiftUI design rather than a Duo-specific exception?

If yes, use the adaptive solution.

Only introduce Duo-specific APIs when the feature depends on a physical property unique to Duo.

The target is not:

**“Make this app support iPhone Duo.”**

The target is:

**“Make this app excellent at every size, then use Duo's unique hardware where it creates additional value.”**
