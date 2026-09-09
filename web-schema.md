# Web (ARIA) — Table Schema

## How a Screen Reader Reads a Web Element

When a user's virtual cursor lands on an element, a screen reader speaks in this order:

> **Accessible Name → Role → State**

For example, a subscribe button might read:
> *"Subscribe now, button."*

A toggle might read:
> *"Notifications, switch, on."*

Unlike iOS's Hint, there's no authored "spoken after a pause" slot on the web — the
closest equivalent, `aria-describedby`, is read as a description after a short pause
in most AT/browser combinations, but support is inconsistent enough that it belongs in
**Notes** as implementation guidance rather than its own column. That's also why this
schema drops Hint and adds **Announce on change**, matching the TalkBack template.

## The 12 Columns

| # | Column | What to Write | Examples |
|---|--------|--------------|----------|
| 1 | **Order** | Number each element in **reading order** — the order the virtual cursor visits elements, which is **DOM order**, not visual order. Leave blank for `Merged into parent` rows and `Hidden: Yes` rows. See "Order — DOM vs. Tab Order vs. Visual Order" below before numbering anything. | `1`, `2`, *(blank)* |
| 2 | **Component** | Short name for the element — match the component/dev-tool name when possible | `Close button`, `Page header`, `Price card` |
| 3 | **Layer** | `Native` or `Web`. On an all-web screen every row is `Web`; on a hybrid screen (native shell + WebView) this decides which team and which API owns the row — see [SKILL.md § Hybrid Screens](SKILL.md#5-hybrid-screens--fill-in-the-layer-column). | `Web`, `Native` |
| 4 | **Role** | The accessible role — from a real HTML element (preferred) or an explicit `role` attribute. See Role Guide below. | `none`, `button`, `heading`, `link` |
| 5 | **Accessible Name** | What the screen reader reads as the element's name. Write `none` if the visible text, `alt`, or associated `<label>` already gives a clear name. | `none`, `Close`, `Free trial offer timeline` |
| 6 | **State** | Current state if it changes. **Leave truly blank** (not `none`) for native form controls — a real `<input type="checkbox">`, `<input type="radio">`, `<select>` — the browser announces their state automatically from the element itself, and adding text risks a double announcement. For any custom ARIA widget (a `div` styled as a switch, a custom combobox), state is **never** automatic — write it explicitly. | *(blank)*, `expanded`, `checked`, `current page` |
| 7 | **Grouping** | `Standalone`, `Parent of N (labelledby group)`, or `Merged into parent`. A landmark or `role="group"`/`role="region"` with `aria-labelledby` merges its children's names into one announcement, the web equivalent of iOS's `.combine`. | `Standalone`, `Parent of 3 (labelledby group)` |
| 8 | **Hidden** | `Yes` for decorative elements removed from the accessibility tree. Engineers apply `aria-hidden="true"` (or `display:none`/`visibility:hidden`, which remove it as a side effect). Distinguish this from the opposite pattern — visually-hidden-but-accessible text (a `.sr-only` class) — which is a `Hidden: No` row with an explicit **Accessible Name** and nothing visible on screen. | `No`, `Yes` |
| 9 | **Actions** | Every way to activate the element, in priority order. **Every action must have a keyboard path** — this is non-negotiable on the web (WCAG 2.1.1): if it responds to click or hover, it must also respond to a key. `Enter`/`Space` for buttons, `Enter` for links, arrow keys for composite widgets (tabs, listbox, menu) using a roving-tabindex pattern. Use `None` for non-interactive elements. | `Enter or Space activates`, `Arrow keys move between tabs`, `None` |
| 10 | **Announce on change** | `Polite` = waits for a pause (toasts, confirmations). `Assertive` = interrupts immediately (errors, urgent updates). `None` = static content. | `None`, `Polite`, `Assertive` |
| 11 | **Web example** | The full sentence the screen reader speaks, in order: Accessible Name, Role, State. Read it aloud to check it sounds natural. | `Subscribe now, button.` |
| 12 | **Notes** | Guidance for engineers — design intent no other column captures. Use `•` bullet points. Write just `•` if none needed. | `• Prefer a native <button> over a div with role="button" and a click handler` |

> **Column order matches the iOS and Android templates** wherever the concept exists,
> so specs review side-by-side. Web shares TalkBack's shape most closely: both drop
> Hint and use Announce on change in its place.

## Role Guide

Prefer a real HTML element over an ARIA role bolted onto a `div` — this is the
["first rule of ARIA use"](https://www.w3.org/TR/using-aria/#rule1): a native `<button>`
gets keyboard support, role, and state for free, while `role="button"` on a `div`
requires you to hand-build all three yourself, and it's easy to miss one.

| Role | Use When... | What's Announced |
|------|------------|-------------------|
| `none` | Plain text — a paragraph, a label, a description | Nothing extra |
| `heading` | A section title (`<h1>`–`<h6>`, or `role="heading" aria-level`) | "heading" plus its **level** — `"heading 2"` — the one thing native headings can't do |
| `button` | Performs an in-page action | "button" |
| `link` | Navigates — in-page, to another page, or off-site | "link" |
| `image` | A meaningful `<img>`, `<svg role="img">`, or CSS background promoted via `role="img"` | "image", from its `alt` |
| `textbox` | A text input or `<textarea>` | "edit text" / "text field", plus its label |
| `checkbox` | A checkable option in a multi-select group | "checkbox" + checked state |
| `radio` | An option in a single-select group | "radio button" + selected state |
| `switch` | A custom on/off control styled as a toggle | "switch" + on/off state — **never automatic**, always author `aria-checked` |
| `tab` | A tab in a tablist | "tab" + selected state |
| `region` / `navigation` / `main` / `banner` | A landmark — lets users jump between page sections, the web's version of iOS's rotor / Android's heading navigation | Announces the landmark type and its label |

## Column-by-Column Guidance

### Order — DOM vs. Tab Order vs. Visual Order

Three different "orders" exist on the web, and they can diverge without anyone intending it:

| Order | Who uses it | What controls it |
|-------|------------|-------------------|
| **Reading order** (what this column numbers) | Screen reader virtual cursor | DOM order |
| **Tab order** | Sighted keyboard users | DOM order, unless a positive `tabindex` overrides it |
| **Visual order** | Sighted mouse/touch users | CSS (`flex-direction`, `order`, `grid-template-areas`, `position: absolute`) |

**This is the biggest risk on the web that has no real native equivalent.** On iOS or
Android, reordering focus takes a deliberate extra property
(`accessibilitySortPriority`, `traversalIndex`) — someone has to choose to do it. On
the web, an ordinary CSS technique used purely for visual polish — a flexbox `order`,
a CSS Grid area, an absolutely-positioned element — silently changes nothing about the
DOM, so **the design can look correct while the reading order is scrambled**, and no
one notices without inspecting the accessible tree. Flag any place where the visual
layout was achieved through CSS reordering rather than DOM order in the Notes column,
so engineers know to double check.

Avoid `tabindex` values greater than `0` — they create a second, competing order for
keyboard users on top of the DOM order and are almost always a sign something should
have been reordered in markup instead.

### Accessible Name — Semantic HTML First

**Use `none` when** the visible text, a `<label for>` association, or `alt` text is
already the right name.

**Write an explicit name when:**
- An icon-only button needs a name: `aria-label="Close"` on a `<button>` with only an X icon
- Meaningful `alt` text is needed on an `<img>` — and `alt=""` (not omitted) for decorative images
- A price with strikethrough needs a combined name: `$30, discounted to $4 per month`
- Screen-reader-only helper text exists via a visually-hidden-but-accessible class (not
  `display:none`/`aria-hidden`, which would hide it from AT too)

### State — When It's Automatic and When It Isn't

The web's version of the native "leave it blank, the system announces it" rule is
**narrower** than it looks: it only holds for genuine native HTML controls.

| Element | State column | Why |
|---------|-------------|-----|
| `<input type="checkbox">` | *(blank)* | Browser announces checked/unchecked from the element itself |
| `<input type="radio">` | *(blank)* | Same — native semantics |
| `<select>` | *(blank)* | Native semantics |
| `<div role="switch">` (custom-styled toggle) | `on` / `off` | **Not automatic** — a custom widget only announces state if `aria-checked` is explicitly authored and kept in sync with the visual state |
| `<div role="checkbox">` | `checked` / `unchecked` | Same — custom widgets always need explicit state |

This is the most common web accessibility bug for exactly the controls designers care
about most (custom switches, custom checkboxes): they look right, but nobody wired
`aria-checked`, so the state is silently missing. Call it out in Notes whenever a
control in the design is a custom-styled native input.

### Hidden — Two Opposite Techniques, Don't Confuse Them

| Technique | Effect | When to use |
|-----------|--------|-------------|
| `aria-hidden="true"` | Removes the element (and descendants) from the accessibility tree — still visible on screen | Decorative icons, redundant labels, background images |
| Visually-hidden-but-accessible class (`.sr-only`, clip-based CSS) | Removes the element from the **visual** layout but keeps it in the accessibility tree | Extra context a sighted user doesn't need but a screen reader user does — e.g. `<span class="sr-only">, opens in a new tab</span>` after a link |

Never use `display:none` or `visibility:hidden` to mean "hide from screen reader only"
— both remove the element from *everyone's* experience, sighted and non-sighted alike.

**Occluded content follows the same rule as native**: anything sitting under an overlay
or paywall stays in the DOM and in the accessibility tree by default. Hiding it takes
an explicit `aria-hidden="true"` (or removing the markup outright) — a CSS-only overlay
does not hide the content underneath from a screen reader.

### Actions — Keyboard Is Not Optional

List every action, first-item-is-default. For each one, confirm there's a keyboard path
— WCAG 2.1.1 makes this a hard requirement, not a nice-to-have:

| Element | Actions |
|---------|---------|
| Button | `Enter or Space activates` |
| Link | `Enter activates` |
| Tabs | `Arrow keys move between tabs, Enter/Space activates the focused tab` |
| Custom dropdown/combobox | `Enter or Space opens, Arrow keys navigate options, Enter selects, Escape closes` |
| Modal/dialog trigger | `Enter opens the dialog; focus moves into it; Escape closes and returns focus to the trigger` |

Anything that only responds to `mouseover`/`mouseout` (a hover-triggered tooltip or
menu) needs a keyboard-equivalent trigger (focus/blur) called out explicitly in Notes.

### Announce on Change — Live Regions

| Value | When to Use | Example |
|-------|-------------|---------|
| `Polite` | Confirmation messages, non-urgent updates | "Item added to cart" |
| `Assertive` | Errors, urgent interrupts | "Session expired" |
| `None` | Static content (default) | Most elements |

Maps to `aria-live="polite"/"assertive"`. **Note in Notes when a live region is used**:
the container needs to exist in the DOM *before* its content changes for some
AT/browser combinations to pick up the update reliably — swapping in a brand-new
`aria-live` container at the same time as its content is a common bug that silently
fails only for screen reader users.

### Web Example — Assembling the Announcement

Combine Accessible Name, Role, and State, in that order.

| Scenario | Web Example |
|----------|--------------|
| Plain text | `Limited time offer` |
| Section heading (h2) | `Your Benefits, heading 2` |
| Icon-only close button | `Close, button` |
| Link | `Subscribe now, link` |
| Custom switch (on) | `Notifications, switch, on` |
| Landmark | `Main content, region` |
| Live region update | `Item added to cart` (spoken automatically on change, not on focus) |

### Notes — What to Tell Engineers

- `• Use a native <button> here, not a styled <div> with an onClick handler`
- `• This toggle is a custom widget — needs aria-checked kept in sync with its visual state`
- `• Visual order here comes from CSS Grid, not DOM order — verify reading order matches Column 1 before shipping`
- `• Icon-only — needs aria-label, the icon itself has no accessible name`
- `• Live region container must exist in the DOM before the confirmation text is inserted`

Avoid: `• Add aria-label="Close"` (that's the engineer's implementation, not your intent).

## How Web Differs from Native

| Concept | Web | Native (iOS/Android) |
|---------|-----|----------------------|
| What announces the element | Browser builds an accessibility tree from the DOM + ARIA, and the screen reader walks *that* tree | The app's UI framework builds the tree directly from view properties; no intermediate document |
| Reading order | DOM order — can silently diverge from visual order via ordinary CSS (flex/grid `order`, `position`) | Visual position by default; diverging requires a deliberate extra property (`accessibilitySortPriority` / `traversalIndex`) |
| Headings | Carry real levels — `heading 1`–`heading 6` | No levels (Android `heading()` is yes/no; iOS heading trait has no level) |
| Automatic state | Only for genuine native HTML controls (`<input>`, `<select>`) — custom ARIA widgets always need explicit state | Native OS controls (`Switch`, `Checkbox`) always auto-announce state regardless of custom styling |
| Hiding | `aria-hidden` (tree only) vs. `display:none`/`visibility:hidden` (tree + visual) — two different tools for two different intents | `accessibilityHidden` / `clearAndSetSemantics` — one tool, tree only |
| Keyboard operability | A first-class, explicit requirement (WCAG 2.1.1) — every interaction needs a keyboard path | Governed by touch gestures and the OS AT's own gesture vocabulary (double-tap, swipe) — not something the app individually re-implements |
| Focus indicator | Designable and stylable (`:focus-visible` outline) — a real design decision that can be gotten wrong | Handled by the OS's own focus highlight — not a per-app design decision |
| Live/dynamic updates | `aria-live` — declarative, attached to a container, with real cross-browser inconsistency | `liveRegion` (Android, declarative) or `AccessibilityNotification.Announcement` (iOS, imperative) |
