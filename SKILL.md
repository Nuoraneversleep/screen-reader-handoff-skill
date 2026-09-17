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

### 1. Determine Platform and Delivery Format

Ask both up front, in the same round of questions, before reading the design:

- **Platform** — ask this explicitly; don't try to infer it from the design. A hybrid
  screen usually looks completely seamless in the mock, with nothing visually marking
  a region as a WebView, so guessing from the frame alone is unreliable:
  - **iOS** → VoiceOver table (see [voiceover-schema.md](voiceover-schema.md))
  - **Android** → TalkBack table (see [talkback-schema.md](talkback-schema.md))
  - **Web** → Web/ARIA table (see [web-schema.md](web-schema.md))
  - **Hybrid** (native app shell containing a WebView) → generate **one** combined
    table, not one per layer — see Step 5. Picking this ask three more things before
    reading the design, since they change what you need from it:
    1. *"Which region is the WebView, and which is native?"* — get the exact
       boundary; it's rarely a clean top/bottom split (a native paywall can float
       over web content with no visible seam).
    2. *"Do you have the actual web markup, or just this visual?"* — Figma shows
       layout, not DOM structure. Without markup, Role/heading-level/DOM-order calls
       for the Web rows are informed guesses from the visual — say so in Notes
       rather than stating them as fact.
    3. *"Is there a known reading-order requirement that spans the boundary?"* —
       e.g. terms that must legally precede a CTA on the other side of the seam.
       Ordering across the native/web boundary is coarse to fix after the fact (see
       Step 5), so it's worth knowing before assigning Order numbers, not after.
- **Delivery format** (always ask, even if the platform is obvious):
  - **Spec file (TSV)** → a tab-separated code block the user pastes into Figma, Sheets, or Notion
  - **Rendered in Figma** → a native table built with `use_figma` plus numbered badges annotating the frame itself (see Step 7)
  - **Both** → produce the TSV and also render it on the canvas

Knowing the delivery format up front matters because it changes what you need from the
design: rendering onto the canvas requires the frame's own coordinate space for
annotation placement (Step 7), not just the visual judgment calls TSV-only delivery
needs. Skip re-asking this in Step 7 — you already have the answer.

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

**On the web, the same exclusion applies to existing shared site chrome** — a global
masthead, nav, search, and footer that are already implemented and maintained by a
separate team, sitting in the mock only for context around whatever this specific
component or page redesign actually is. Mockups for a single component or section
often paste in a flattened screenshot of the real site header for orientation (look
for layer names like a URL or `Screenshot ...`) rather than a real breakdown of nav
links — that flattening is itself a signal it's reference chrome, not new work. Same
rule as OS chrome: no row at all, not even `Hidden: Yes`.

**Watch for off-canvas duplicates and overflowing nested content.** Figma files
assembled from shared component libraries sometimes carry a leftover copy of a module
positioned outside the frame's visible bounds (negative `y`, or far past the frame's
edge) — a stray duplicate, not part of the composition. Separately, a nested
instance's children can report `y` positions that exceed its own parent's declared
height, meaning that content doesn't actually render where the layer tree implies.
Neither is an occlusion case (Step above) — do the same "compare positions, don't
eyeball it" arithmetic, and when a node's numbers don't add up, ask rather than
guessing whether it's live content.

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
- Set **Layer** on every row to `Native` or `Web` — **native tables only**. On an
  all-native screen every row is `Native`. On an all-web screen, skip this column
  entirely — you're using [web-schema.md](web-schema.md)'s 11 columns, which drop
  Layer since every row would be `Web` anyway and the column would carry zero
  information. On a hybrid screen, the *combined* table keeps Layer, since it decides
  which API and which team owns each row — read the hybrid-screens step before filling
  it in.
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

**Exception — the NYT homepage (and similarly-built multi-column module grids):**
reading order is top-to-bottom across visual bands first, then **right-to-left**
within a band — not left-to-right. A top band with a wide lead story on the left and a
narrower module on the right (e.g. the lead news story beside a smaller feature card)
announces the **left** story first, because top-to-bottom between bands outranks the
right-to-left rule — the two stories are in the same visual row, but the lead package
still reads as the first band. Right-to-left only decides order **between modules that
are genuine same-row siblings** at the same structural level, not between a lead
package and an adjacent rail. This is a DOM-order fact about how the homepage template
is actually built (see [web-schema.md](web-schema.md)'s Order section on DOM vs.
visual order) — verify it against the real page or a screen reader test rather than
assuming standard left-to-right, and say so explicitly in Notes wherever a homepage
spec's Order column relies on this.

### 5. Hybrid Screens — Fill In the Layer Column

This is the step Step 1 flagged if the user picked **Hybrid**: a story page in a
WebView with a native paywall over it is one screen to the reader but two
accessibility trees to the system — the WebView bridges its DOM tree into virtual
nodes, and the screen reader walks the merged result. Semantics on each side are
authored with different APIs and usually owned by different teams, so mark every row
`Native` or `Web`.

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
while native headings have no level. [web-schema.md](web-schema.md) covers ARIA
concepts (Role, DOM order, live regions) in full — use it as the reference for what
each Web-layer row's Notes should tell the web team, even though the row itself lives
in the combined table below.

**Everything in [web-schema.md](web-schema.md)'s Heading Levels, Landmarks, and Links
sections applies unchanged inside a WebView** — the DOM inside a WebView is still a
DOM, read by the same browser accessibility layer, regardless of the native shell
wrapping it. Don't let the shell distract from the same checks that apply to a pure
web page:

- **Heading levels and nesting still matter, and still need the exact level kept.**
  Even though the combined table's Trait/Element Type column uses native vocabulary
  for everything else (see below), a Web-layer heading row still needs its `h`-level
  written down somewhere — don't let it collapse to a bare "header" the way a native
  header row does. Keep the level in the Component name or Notes if the Trait column
  itself can't carry it (`Component: Story headline (h3)`).
- **Landmarks inside the WebView still need identifying**, the same way `main`/
  `navigation`/`banner` matter on a pure web page — a long article body inside a
  WebView still benefits from a `main` landmark so a user can skip past a native
  header straight to the content, even though the *seam* between native and web is
  handled by the native ordering properties, not by landmarks.
- **Link naming rules are unchanged.** A "Read more" link inside WebView story content
  is exactly as broken pulled into the page's link list whether that WebView sits
  inside a native app or a browser tab — the web accessibility tree doesn't know or
  care that a native shell wraps it.
- **One exception: Grouping in the combined table below still follows the native
  model, not pure Web's always-`Standalone` rule.** The combined table's shape is the
  native platform's shape, and a Web-layer story card there can still be authored as
  one `Parent of N (combined)` row if that's how the app team wants that particular
  seam to announce — pure Web's "never group" rule from
  [web-schema.md](web-schema.md) is about a table using Web's own 12 columns, which
  the combined table isn't.

#### Deliver One Combined Table, Not One Per Layer

A hybrid screen is one screen to the user — one continuous swipe sequence, not two
separate trees. So the handoff artifact should be **one table**, not a native table
plus a separate web table:

- **Use the native platform's table shape** (VoiceOver or TalkBack columns) as the
  single artifact, since the native shell is what stitches the two trees into one
  reading order and is usually the audience that needs to see the seam.
- **Number every row in one continuous Order sequence** across both layers — this is
  the entire point. Two separate tables each starting at `1` hide the thing most likely
  to break: whether the web content lands in the right place in the native reading
  order.
- **Tag each row's Layer** (`Native` or `Web`) as already described. For a Web row's
  **Trait**/**Element Type** column, use the closest native-vocabulary term (`link`,
  `header`, `button`, `image`, `none`) rather than a raw ARIA role — the table's
  primary audience is reading it as one consistent list, not switching vocabularies
  mid-table.
- **Push ARIA-specific authoring detail into that row's Notes**, and point to
  [web-schema.md](web-schema.md) for the exact Role/state/DOM-order conventions —
  e.g. `• Web layer — see web-schema.md. Role: switch, needs aria-checked authored
  explicitly (custom widget, not a native input).`
- **If the WebView content is complex enough to need its own detailed spec** — a long
  article page with many interactive elements, say — produce a *supplementary* Web
  table using the full [web-schema.md](web-schema.md) columns for that content's own
  team, and cross-reference it by name from the Notes column of its corresponding row
  in the combined table. The combined table stays the single source of truth for
  reading **order** across the seam; the supplementary table is for that layer's own
  implementation detail — this is exactly where exact heading levels, landmark
  identification, and per-link naming (see above) actually get specified, since the
  combined table's simplified vocabulary has nowhere to carry that detail.

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
- [ ] Every row has a **Layer** on native and hybrid tables, and anything owned by the
      web layer is flagged as such in the Notes so it routes to the right team. A pure
      Web table has no Layer column at all — see [web-schema.md](web-schema.md)
- [ ] The **Example** column reads naturally — read it aloud to check
- [ ] No blank cells — use `none`, except Android **State** for auto-announced conditions
- [ ] **On Web specifically**: every `heading` row has an exact level (`heading 2`, never
      a bare `heading`), levels nest without skipping, and a repeated module reuses the
      same level on every instance — see [web-schema.md](web-schema.md)'s Heading Levels
      section, this is the single highest-value thing to get right on a web spec. On any
      page with more than a handful of headings, verify this with a **Heading Outline**
      (see below) rather than eyeballing the row table
- [ ] **On Web specifically**: the page's major regions (`main`, `navigation`, `banner`,
      `contentinfo`) are identified in a standalone **Landmark Map** (see below), NOT as
      rows in the per-element table — a landmark isn't a linear focus stop, so it doesn't
      belong in a table of Order/Role/Actions. Any repeated `navigation` landmarks have
      distinct names, and a repeated card module is a heading, not its own landmark — see
      [web-schema.md](web-schema.md)'s Landmarks section
- [ ] **On Web specifically**: every link's Accessible Name makes sense pulled out of
      context, in the page's own link list — no bare "Read more"/"Click here" repeated
      across cards, no icon-only link missing a name, and any link that opens in a new
      tab says so as part of its name — see [web-schema.md](web-schema.md)'s Links section

### 6.5. On Web — Add a Heading Outline and Landmark Map

These are two small companion documents, not a replacement for the row table — the row
table is still the only place carrying implementation detail (accessible names,
actions, link destinations) that engineering builds against. Produce them alongside
the row table whenever the page has more than a handful of headings, more than one or
two landmarks, or a reviewer needs to verify page-level structure at a glance rather
than reading 50 rows top to bottom.

**Why not just add landmark rows to the table?** A landmark isn't a linear focus
stop — nobody reading straight through the page "lands on" a `main` boundary the way
they land on a button. It's the anchor for a *separate*, parallel jump-navigation
index. Forcing it into a row gives it a fake `Actions: None` / blank `State` it
doesn't actually have, and worse, it can eat an Order number that should belong to a
real focus stop. Headings are different: a heading genuinely is a focus stop *and* a
member of the heading-skim index, so heading level correctly stays in the row table's
Role column — the outline below is a second view of that same data, not a place to
move it to.

**Heading Outline** — walk the row table's Role column top to bottom, pull out every
heading row, and render it as a plain indented list:

```
h1  Discover all that's new. With all of The Times.
  h2  Subscribers enjoy more with New York Times All Access.
    h3  News
    h3  Games
    h3  Cooking
    h3  Audio
    h3  Wirecutter
    h3  The Athletic
  h2  Make The Times part of your day. At your own pace.
```

A skipped level or a heading competing at the wrong level is far easier to spot in
this shape than buried among 50 other rows — the same reason it's the first thing a
screen reader user's own heading list surfaces.

**Landmark Map** — a three-column table of every landmark on the page, independent of
the row table. This mirrors what a real accessibility inspector (e.g. a browser
extension's landmarks panel) reports, so it reads the same way a reviewer running that
tool against the shipped page would see it:

| Element | Role | Label |
|---------|------|-------|
| `header` | `banner` | *(none)* |
| `main` | `main` | *(none)* |
| `nav` | `navigation` | New York Times All Access |
| `nav` | `navigation` | Other subscriptions |
| `footer` | `contentinfo` | *(none)* |

- **Element** — the real HTML tag the landmark is authored with. Most landmark roles
  come from a dedicated tag (`<header>` → `banner`, `<nav>` → `navigation`, `<main>` →
  `main`, `<footer>` → `contentinfo`), but `<section>`/`<div role="region">` both
  surface as `region` in the Role column, so Element is what distinguishes them for
  engineering.
- **Role** — the landmark role as a screen reader announces it. Same vocabulary as
  [web-schema.md](web-schema.md)'s Landmarks section.
- **Label** — the landmark's accessible name (`aria-label` or `aria-labelledby`),
  exactly as it would be spoken. Leave `*(none)*` only when a page truly has just one
  of that role — the moment two landmarks share a role, at least one needs a label,
  or they announce as indistinguishable duplicates. Flag any duplicate label the same
  way a real landmarks-inspector tool does: as a warning, not a silent gap.

Call out gaps explicitly in a note below the table — a missing `main`, two landmarks
of the same role sharing one label (or both unlabeled), more landmarks than the page
actually needs (see [web-schema.md](web-schema.md)'s Landmarks section on
over-landmarking).

Deliver both as plain text/markdown blocks alongside the row table — they don't need
`use_figma` rendering or canvas annotation the way the row table does in Step 7, since
they aren't tied to individual on-screen elements the same way.

### 7. Deliver — TSV, Rendered Table, or Annotated Design

Use the delivery format chosen in Step 1 — don't re-ask here.

**Spec file (TSV)**: output a TSV code block the user can paste into Figma, Sheets, or
Notion.

**Rendered in Figma**: render it as a native table using `use_figma`. See
[figma-canvas.md](figma-canvas.md) for a ready-to-run script and the auto-layout
constraints that otherwise cause hard-to-debug failures. Also **annotate the frame
itself**: a numbered badge and dashed outline on top of every element that has a row,
using that row's **Order** number. A table next to the frame still leaves the reader to
match rows to shapes by eye — the numbers close that gap, and they make
priority-over-position order (nav chrome first, terms before the CTA) visible on the
canvas instead of only asserted in the Notes column. See
[figma-annotations.md](figma-annotations.md) for the script. Skip a badge for any row
with a blank Order (merged-into-parent or Hidden rows).

Rendering has a real advantage over pasted TSV: the rows sit beside the screen they
document, so a reviewer can check an announcement against the design without
switching tools, and engineers see the spec in the same file they are already
building from.

**Both**: do both of the above.

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
| A story/article card (photo + headline + dek + metadata, one clickable unit) | The photo illustrates the story, so it is **not** decorative — give it a real descriptive label, not `Hidden: Yes`. **Put the headline ahead of the photo in reading order either way**, regardless of which one displays first visually — a screen reader user should hear "what is this story" before sitting through a photo description, not after. On **native** (iOS/Android), if the whole card is one control, model it as **Parent of N (combined)**: one Order number on the card itself, every piece listed below it as `Merged into parent` with a blank Order, headline first inside that combined announcement and the photo last. On pure **Web**, skip grouping entirely — [web-schema.md](web-schema.md) always uses `Standalone`, so the same card is several standalone rows, each with its own Order number, headline's Order number earlier than the photo's. Either way this is the DOM-vs-visual-order concept in practice — implemented with CSS reordering (flex/grid `order`), not a change to what displays — so flag it clearly in Notes so a later reviewer doesn't "fix" it by matching order back to the visual layout. On native, also flag in Notes if the combined name reads as too long overall — that's a real trade-off, not something to silently fix by hiding the photo. |

## Platform Schema References

- **iOS VoiceOver**: [voiceover-schema.md](voiceover-schema.md)
- **Android TalkBack**: [talkback-schema.md](talkback-schema.md)
- **Web (ARIA)**: [web-schema.md](web-schema.md)
- **Full examples with best-practice annotations**: [examples.md](examples.md)
- **Rendering the table onto the Figma canvas**: [figma-canvas.md](figma-canvas.md)
- **Annotating the design with numbered markers**: [figma-annotations.md](figma-annotations.md)
