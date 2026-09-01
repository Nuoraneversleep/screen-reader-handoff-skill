 ---
name: screen-reader-handoff
description: >-
  Generate screen reader accessibility handoff tables for iOS VoiceOver and
  Android TalkBack from design screens. Designed for designers who need to
  create clear accessibility specs for engineering handoff. Outputs
  tab-separated (TSV) tables ready to paste into Figma or spreadsheets.
  Use when the user asks for accessibility handoff, VoiceOver table,
  TalkBack table, screen reader documentation, a11y handoff, or
  accessibility specs for a design.
---

# Screen Reader Handoff for Designers

Create clear, structured accessibility specs that tell engineers exactly what a screen reader user should hear on every screen. No screen reader expertise required.

## What This Skill Produces

A **tab-separated table** (TSV) where each row is one element a screen reader user can land on, in the order they'd encounter it. The table tells engineers:
- What the user **hears** for each element
- What **type** of element it is (button, heading, image, etc.)
- What **happens** when they interact with it
- Any **guidance** for implementation

## Workflow

### 1. Determine Platform

Ask if not obvious:
- **iOS** → VoiceOver table (see [voiceover-schema.md](voiceover-schema.md))
- **Android** → TalkBack table (see [talkback-schema.md](talkback-schema.md))
- **Both** → Generate one table per platform

### 2. Read the Design

**If the user provides a Figma URL**, extract the design using MCP tools:

```
Step 1: Call `get_metadata` for the structural overview — node IDs, layer names,
        types, positions, and sizes. Positions are what establish swipe order.
Step 2: Call `get_screenshot` to see the screen. You need the visual to judge what
        is decorative, what reads as a heading, and what groups together.
Step 3: ONLY IF text is unreadable in the screenshot or a component's variant is
        ambiguous, call `get_design_context`. Usually steps 1-2 are enough — Figma
        auto-names text layers after their content, and the screenshot supplies the
        rest. Skip it by default.
Step 4: Walk the node tree and identify every visible element: text, buttons,
        images, inputs, toggles, carousels, links.
```

Extract the `fileKey` and `nodeId` from the Figma URL: for
`figma.com/design/:fileKey/:fileName?node-id=1-2`, the `fileKey` is `:fileKey` and
the `nodeId` is `1:2`. If the URL has no `node-id`, ask for a node-specific link —
right-click the frame in Figma and choose **Copy link to selection**.

**Watch for `hidden="true"` in the metadata.** Those layers are switched off in this
variant and are not on screen — leave them out of the table entirely. That is a
different thing from the `Hidden` column, which means *visible on screen but removed
from the screen reader*. Do call out any switched-off layer that matters, though: a
hidden "Already a subscriber? Log in." link may be the only escape route for a
logged-out subscriber, and its absence is worth a question.

**Check for occlusion.** Compare positions: an element can be present in the tree but
sit underneath an overlay, gradient, or paywall. Anything covered should be `Hidden:
Yes` — content behind a paywall stays in the accessibility tree by default, so a
screen reader user can read paywalled text unless engineering explicitly clears it.
Do the arithmetic rather than eyeballing it; child coordinates are relative to the
parent frame, so add the offsets before comparing against the overlay's bounds.

**Leave out OS chrome entirely — no row, not even `Hidden: Yes`.** The status bar
clock, battery, cellular and wifi indicators, the home indicator, the notch and the
Dynamic Island belong to the platform, not the app. iOS and Android expose them
through their own accessibility layers, so a row for them would tell engineers to
implement something they do not own. This is different from a decorative divider,
which the app *does* own and which therefore earns a `Hidden: Yes` row.

Mockups usually include this chrome from a UI kit, so it will be in the metadata.
Two ways to spot it:

- **By name** — `Status bar`, `StatusBar`, `status_bar/dark`, `Home indicator`,
  `Notch`, `Dynamic Island`, `Battery`, `Wifi`, `Signal`, `Carrier`.
- **By shape, when the layer is named something useless like `Group 12`** — a
  full-width strip flush with the top of the frame, under ~60px tall, containing
  clock-like text (`9:41`). The clock is the tell: a nav bar sits in the same place
  and looks similar, but has a title and back button instead of a time.

Prune the whole subtree, not just the parent. If you only drop the `Status bar`
frame and keep walking its children, `9:41`, the battery and the wifi icon each
arrive as their own row.

**If no Figma URL**, ask the user to describe the screen or provide a screenshot.

### 3. Think Like a Screen Reader User

For every visible element on screen, ask:

| Question | Why It Matters |
|----------|---------------|
| Can the user tap or interact with it? | Determines if it's a button, link, toggle, etc. |
| Is it a section heading? | Headings let users jump between sections quickly |
| Is it a meaningful image or just decorative? | Decorative images should be hidden from screen readers |
| Does it change state? (expand/collapse, on/off) | Users need to hear the current state |
| Does tapping it leave the app? | Users should be warned before leaving |
| Would reading just the visible text make sense? | If not, write a custom label |

### 4. Generate the Table

- Output as **TSV** (tab-separated values).
- One header row, one row per element a screen reader user can focus on.
- Order rows by **announcement priority**, not visual position — see below.
- Set **Layer** on every row to `Native` or `Web`. On an all-native screen every row is
  `Native`; on a hybrid screen this decides which API and which team owns the row, so
  read the hybrid-screens step before filling it in.
- Use `none` (lowercase) for empty cells, with one exception: leave the Android
  **State** cell truly blank for Switch, Checkbox, Radio button, Toggle button,
  selectable cells, and disabled controls. The system announces those binary states
  itself, so writing anything — including `none` — risks a double announcement. Every
  other empty cell gets `none`.
- Use `[brackets]` for content that changes at runtime.
- Keep multi-line notes on a single line using ` • ` as separator.

#### Row Order — Priority, Not Position

Screen reader users move through a screen strictly sequentially. Anything placed
after a decision point may never be heard. So visual position does not drive
announcement order:

1. **Navigation chrome first** — toolbars, app bars, and bottom action bars (NYT's
   "charm bracelet") come first regardless of where they sit on screen. A bottom bar
   at y=787 is still announced before the headline at y=43.
2. **Terms, legal, and consent copy next** — always before the CTA it governs. A
   sighted user catches terms below a Subscribe button in peripheral vision before
   deciding; a screen reader user will activate the button as soon as they hear it
   and never reach the terms.
3. **Everything else** — top-to-bottom, left-to-right.

When priority order differs from visual order, say so in the Notes column. Android
needs `traversalIndex` plus `isTraversalGroup` on a shared parent; iOS needs
`accessibilitySortPriority`. Neither happens by default.

### 5. Hybrid Screens — Fill In the Layer Column

Many screens are not all-native. A story page in a WebView with a native paywall over
it is one screen to the reader but two accessibility trees to the system: the WebView
bridges its DOM tree into virtual nodes, and the screen reader walks the merged result.
Semantics on each side are authored with different APIs and usually owned by different
teams, so mark every row `Native` or `Web`.

| | Native | Web |
|---|---|---|
| Label | `contentDescription` / `accessibilityLabel` | `alt`, `aria-label`, text content |
| State | `stateDescription` | `aria-expanded`, `aria-checked` |
| Hide | `clearAndSetSemantics` / `accessibilityHidden` | `aria-hidden="true"`, `display:none` |
| Order | `traversalIndex` / `accessibilitySortPriority` | DOM order |

Three constraints to spell out in the handoff:

- **Native modifiers do not reach inside a WebView, and web attributes do not reach
  out.** Alt text for an image in the story page is an `alt` attribute owned by the
  CMS or article-rendering team — not something the app team can add.
- **Hiding occluded web content has to happen in the web layer.** A native overlay does
  not clear the WebView's accessibility tree, so a screen reader can reach content
  sitting behind a paywall. The blunt native fix (`importantForAccessibility =
  NO_HIDE_DESCENDANTS` on the WebView) hides *all* web content including the visible
  preview, so it is usually wrong. Cleanest is not sending paywalled markup at all.
- **Ordering across the boundary is coarse.** You can order native elements relative to
  each other and to the WebView as a whole, but you cannot interleave a native element
  between two web nodes. If terms live in the web layer while the CTA is native, they
  cannot be reordered across that seam — the terms have to move into the native
  component.

Also note that web headings carry `h1`–`h6` levels and are announced as "heading 1",
while native headings have no level.

### 6. Quality Check

After generating, verify:
- [ ] Every tappable element is marked as a button or link
- [ ] Section headings are marked as headers (enables "jump to heading" navigation)
- [ ] Every button describes what happens when tapped
- [ ] External links warn the user they'll leave the app
- [ ] Toggles/accordions communicate their state (expanded/collapsed, on/off)
- [ ] Decorative images are excluded (not in the table)
- [ ] OS chrome (status bar, home indicator, notch) has no row at all — and none of
      its children leaked in as rows either
- [ ] Navigation chrome is first; terms precede any CTA they govern
- [ ] Content occluded by an overlay or paywall is `Hidden: Yes`, with its real text kept
- [ ] Every row has a **Layer**, and anything owned by the web layer is flagged as such
      in the Notes so it routes to the right team
- [ ] The **Example** column reads naturally — read it aloud to check
- [ ] No blank cells — use `none`, except Android **State** for auto-announced conditions

### 7. Deliver — TSV, Rendered Table, or Annotated Design

Default to a **TSV code block** the user can paste into Figma, Sheets, or Notion.

If the user asks for the spec *in* the file — "draw it next to the frame", "put it on
the canvas", "add it to the Figma file" — render it as a native table instead, using
`use_figma`. See [figma-canvas.md](figma-canvas.md) for a ready-to-run script and the
auto-layout constraints that otherwise cause hard-to-debug failures.

When you render onto the canvas, also **annotate the frame itself**: a numbered badge
and dashed outline on top of every element that has a row, using that row's **Order**
number. A table next to the frame still leaves the reader to match rows to shapes by
eye — the numbers close that gap, and they make priority-over-position order (nav
chrome first, terms before the CTA) visible on the canvas instead of only asserted in
the Notes column. See [figma-annotations.md](figma-annotations.md) for the script.
Skip a badge for any row with a blank Order (merged-into-parent or Hidden rows).

Rendering has a real advantage over pasted TSV: the rows sit beside the screen they
document, so a reviewer can check an announcement against the design without
switching tools, and engineers see the spec in the same file they are already
building from.

Offer the canvas version — table plus annotations — alongside the TSV when the user
has not said which they want.

## Key Concept: The Example Column

The **Example** column is the most important column in the table. It's the exact words a screen reader speaks aloud. Engineers and QA use it to verify their implementation is correct.

Read each Example aloud. If it sounds awkward or confusing, revise the Label, Hint, or Value.

## Quick Reference — Common UI Patterns

| What You See in the Design | What to Put in the Table |
|---------------------------|-------------------------|
| Plain text paragraph | One row, no special type. VoiceOver reads it as-is. |
| Bold section title (e.g., "Your Benefits") | Mark as **header** so users can jump to it |
| A button that stays in the app | Type: **button**. Describe what tapping does. |
| A button with an external link icon | Type: **button**. Add hint: "Opens an external website." |
| A hero image or illustration | Type: **image**. Write a short description as the label. |
| A decorative divider or background shape | **Skip it** — don't include in the table |
| A toggle or switch | Type: **button/switch**. Include state: On/Off |
| An accordion (expand/collapse) | Type: **button**. Include state: expanded/collapsed. Add hint about what it shows/hides. |
| A carousel or swipeable content | Type: **adjustable/list**. Describe swipe behavior. Include position: "1 of 6" |
| Text with a hyperlink inside it | Include the full text. Note that there's a link and how to access it. |
| Price with strikethrough + discounted price | Combine into one label: "$30, discounted to $4 per month" |

## Platform Schema References

- **iOS VoiceOver**: [voiceover-schema.md](voiceover-schema.md)
- **Android TalkBack**: [talkback-schema.md](talkback-schema.md)
- **Full examples with best-practice annotations**: [examples.md](examples.md)
- **Rendering the table onto the Figma canvas**: [figma-canvas.md](figma-canvas.md)
- **Annotating the design with numbered markers**: [figma-annotations.md](figma-annotations.md)
