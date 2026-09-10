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

# Rule 1 — Design for available space, not device identity

Never make layout decisions primarily from:

```swift
UIDevice.current.userInterfaceIdiom
UIScreen.main.bounds
device model
interface orientation
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

### Agent rule

**Never use `UIScreen.main.bounds` for SwiftUI layout unless there is an exceptional non-layout requirement.**

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

Do not use fold geometry for ordinary system layouts that already adapt correctly.

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

Use split-style arrangement when both views deserve dedicated space.

Conceptually:

```swift
ArrangementView {
    PrimaryView()
} secondary: {
    SecondaryView()
}
.arrangementViewStyle(.split)
```

Use overlay style when one view is supplementary to another:

```swift
.arrangementViewStyle(.overlay)
```

### Agent rule

**Use `ArrangementView` only when the two views form one adaptive experience; do not substitute it for app navigation.**

Avoid placing navigation containers inside it unless Apple explicitly supports that configuration.

---

# Rule 12 — Let system navigation and bars adapt vertically

On Duo, system bars may move to vertical edges.

Prefer actual SwiftUI toolbars and tab/navigation APIs over custom bar implementations.

Use:

```swift
.toolbar
ToolbarItem
ToolbarItemGroup
ToolbarOverflowMenu
```

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

Relevant APIs include:

```swift
.visibilityPriority(...)
ToolbarOverflowMenu
.topBarPinnedTrailing
.axisBehavior(...)
```

Use vertical behavior APIs rather than building a second toolbar.

### Agent rule

**Primary actions must remain visible; secondary actions may move into overflow as available bar space decreases.**

---

# Rule 14 — Prefer adaptive sidebar navigation where it improves wide layouts

For tab-based apps, consider adaptive sidebar presentation on the inner display.

Relevant concepts include:

```swift
TabView
.sidebarAdaptable
defaultTabBarPlacement
isTabViewSidebarAvailable
tabBarPlacement
```

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
- fold geometry
- system bars
- multitasking

Keep important controls and readable content inside safe areas.

Decorative backgrounds may extend using:

```swift
.ignoresSafeArea()
```

when appropriate.

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

Conceptually:

```swift
.onHingeChange { previous, context in
    let hinge = context.hinge
}
```

Possible states include closed, partially open, and fully open.

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

as independent canvases.

For supplementary content on the second display, use the system-provided scene-accessory APIs.

Use cases include:

- camera subject preview
- countdown
- teleprompter
- presenter information
- supplementary status

### Agent rule

**Use scene accessories for intentionally secondary display content; keep the main experience attached to the primary scene.**

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
Reserved Regions
ArrangementView
vertical toolbar behavior
Duo-specific safe-area handling
```

## Tier 3 — Duo-exclusive experiences

Only when the product benefits:

```text
onHingeChange
hinge angle
scene accessories
multi-display interactions
multi-scene Duo workflows
```

### Agent rule

**Always complete Tier 1 before proposing Tier 2 or Tier 3 changes.**

---

# Rule 25 — Do not prematurely optimize for untestable behavior

Until the project can be tested with the iPhone Duo simulator in Xcode 27.1:

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

**If Duo-specific behavior cannot yet be verified in the simulator, prefer preparation over speculative implementation.**

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

# Final skill philosophy

When adapting an app for iPhone Duo, the agent should repeatedly ask:

> Can this improvement be expressed as a generally adaptive SwiftUI design rather than a Duo-specific exception?

If yes, use the adaptive solution.

Only introduce Duo-specific APIs when the feature depends on a physical property unique to Duo.

The target is not:

**“Make this app support iPhone Duo.”**

The target is:

**“Make this app excellent at every size, then use Duo's unique hardware where it creates additional value.”**
