# Android TalkBack — Table Schema

## How TalkBack Reads an Element

When a user swipes to an element, TalkBack speaks in this order:

> **Description → State → Element Type → Action hint**

For example, a subscribe button might read:
> *"Subscribe now, button. Double tap to activate. Opens external browser."*

That breaks down as: Description ("Subscribe now") → Element Type ("button") → Action hint ("Double tap to activate. Opens external browser.").

## The 10 Columns

| # | Column | What to Write | Examples |
|---|--------|--------------|----------|
| 1 | **Order** | Number each element in swipe order (top-to-bottom, left-to-right). **Only number what TalkBack actually focuses on:** standalone elements and parents. Leave blank for `Merged into parent` rows and `Hidden: Yes` rows — those aren't read as their own focus stops. | `1`, `2`, *(blank)* |
| 2 | **Component** | Your design system name for the element. Helps engineers locate it in code. | `Featured card`, `Save button`, `Close button` |
| 3 | **Element Type** | What type of thing it is. One of: `Button`, `Toggle button`, `Switch`, `Checkbox`, `Radio button`, `Image`, `Heading`, `Tab`, `Link`, `None`. Use `None` for decorative or live-region-only elements. | `Button`, `Heading`, `Image`, `None` |
| 4 | **Description** | The spoken label. Write in sentence case. Use `[brackets]` for dynamic content. Write `Decorative` for elements that should be hidden. | `Close`, `Free trial offer timeline`, `[User's first name]`, `Decorative` |
| 5 | **State** | Custom state like `Playing, 2:15 of 4:30`. **Leave blank for Switch / Checkbox / Radio / selectable cells / disabled controls** — the system auto-announces the binary state and adding it causes double announcements. | `none`, `Playing, 2:15 of 4:30`, `3 of 5`, `75 percent` |
| 6 | **Grouping** | `Standalone` for solo elements. `Parent of N` for containers that merge children into one announcement. `Merged into parent` for children that should not be swiped to individually. | `Standalone`, `Parent of 3`, `Merged into parent` |
| 7 | **Hidden** | `Yes` for decorative icons, redundant labels, background images. Engineers apply `clearAndSetSemantics`. | `No`, `Yes` |
| 8 | **Action** | Every action a user can perform on this element, in priority order. The first item becomes the default tap action; the rest go to TalkBack's swipe up/down menu. Use `None` for elements with no actions. | `Subscribe`, `Open article, Save for later, Share`, `None` |
| 9 | **Announce on change** | `Polite` = waits for the user to pause (toasts, confirmations). `Assertive` = interrupts immediately (errors, breaking news). `None` = no announcement. | `None`, `Polite`, `Assertive` |
| 10 | **TalkBack example** | The full sentence TalkBack speaks aloud, in order: description, state, element type, action hint. Read it aloud to sanity-check — if it sounds awkward, rewrite the description. | `Subscribe now, button. Double tap to activate.` |

### Order — Only Number What's Focusable

Number rows in the sequence a TalkBack user swipes through them. Give a number only to rows that are their own focus stop — `Standalone` elements and `Parent of N` containers. **Leave Order blank** for `Merged into parent` and `Hidden: Yes` rows: they're listed for engineer clarity but TalkBack never lands on them, so they take no position in the swipe sequence.

## Element Type Guide

| Type | Use When... | What TalkBack Announces |
|------|------------|----------------------|
| `None` | Plain text — paragraphs, labels, descriptions. Also for decorative elements and live regions where you only want content updates spoken. | Nothing extra |
| `Heading` | Section title. Lets users jump between sections with the heading reading control. | Says "heading" |
| `Button` | Tappable element that performs an action | Says "button" and "Double tap to activate" |
| `Toggle button` | Button that flips between two states (e.g. Follow / Following) | Says "button" + on/off state |
| `Switch` | iOS-style switch that turns something on or off | Says "switch" + on/off state |
| `Checkbox` | Selectable box in a multi-select list | Says "checkbox" + checked state |
| `Radio button` | Selectable option in a single-select group | Says "radio button" + selected state |
| `Tab` | Tab in a tab bar | Says "tab" + selected state |
| `Image` | Meaningful image, illustration, or animation | Says "image" |
| `Link` | Tappable hyperlink | Says "link" |

## Column-by-Column Guidance

### Description — Don't Repeat What TalkBack Already Says

Write what the user should hear as the element's name. **Don't include the element type or state in the description** — TalkBack announces those itself, so duplicating them causes things like "Subscribe button, button" or "Notifications on, on."

❌ `Subscribe button` → reads as "Subscribe button, button"
✅ `Subscribe` → reads as "Subscribe, button"

❌ `Notifications on` → reads as "Notifications on, switch, on"
✅ `Notifications` → reads as "Notifications, switch, on"

**Use the visible text when** it's self-explanatory ("Subscribe to gain unlimited access…" reads fine as-is).

**Write an explicit description when** the visible text alone would be confusing — an "X" icon should be `Close`, an animation needs `Free trial offer timeline`, a strikethrough price needs `$30, discounted to $4 per month`.

**Write `Decorative`** for elements you want hidden. Engineers apply `clearAndSetSemantics { }`.

### State — When to Leave It Blank

For **Switch, Checkbox, Radio button, Toggle button, selectable cells, and disabled controls**, leave State blank. TalkBack announces the binary condition automatically — adding it causes "On, on" / "Checked, checked" / "Disabled, disabled."

**Disabled is one of these auto-announced conditions.** Leave State blank and note that engineering should set `enabled = false` (Compose) / `setEnabled(false)` (View) — don't hand-write `stateDescription = "Disabled"`. TalkBack appends "disabled" itself, so the word still appears in the example (e.g. `Save article, button, disabled`), and a disabled control's Action becomes `None`. This is a TalkBack-only convention: on iOS VoiceOver, `dimmed` goes in the Value column the normal way.

For **custom measured states**, write exactly what TalkBack should say. You're writing `stateDescription` verbatim:

| Element | State |
|---------|-------|
| Audio player | `Playing, 2:15 of 4:30` |
| Stepper | `3 of 5` |
| Slider | `75 percent` |
| Date picker | `Today, 9:30 AM` |
| Accordion | `Expanded` / `Collapsed` |

(Sliders also need `progressBarRangeInfo` so TalkBack's adjustment gestures work — that's the engineer's job, but worth noting.)

### Grouping — How Children Behave

| Value | Meaning | When to Use |
|-------|---------|-------------|
| `Standalone` | Element stands alone | Solo buttons, single text labels |
| `Parent of N` | Merges N children into one announcement | A card with title + price + image — read as one block |
| `Merged into parent` | This element is consumed by its parent | The title, price, image inside the card above |

Note: Android has no equivalent to iOS's "drill-in" container. Either children merge (`mergeDescendants = true`) or they don't.

### Hidden — Remove from TalkBack

Mark `Yes` for:
- Decorative icons that have a labeled sibling (a heart icon next to "Save" button text)
- Redundant labels
- Background images and dividers

Engineers will apply `clearAndSetSemantics { }` to clear the element and all of its semantic info.

### Action — One List, Priority Order

List every action a user can perform on this element. The **first item becomes the default tap action**; the rest become secondary actions in TalkBack's swipe up/down menu.

For most elements, **one action is enough** — engineering binds it to the tap gesture by default. Only list multiple actions when the element supports operations beyond tap (long-press menus, swipe gestures that need a screen-reader alternative).

| Element | Action(s) |
|---------|-----------|
| External link button | `Open in browser` |
| Article card with long-press menu | `Open article, Save for later, Share, Hide` |
| Accordion trigger | `Expand` (when collapsed) / `Collapse` (when expanded) |
| Plain text paragraph | `None` |

Use `None` (not blank) for elements with no actions — it keeps the decision explicit.

### Announce on Change — Dynamic Content

For elements whose content updates after the screen loads:

| Value | When to Use | Example |
|-------|-------------|---------|
| `Polite` | Confirmation messages, toasts, non-urgent updates | "Article saved" toast |
| `Assertive` | Errors, breaking news, urgent updates that must interrupt | "Connection lost" banner |
| `None` | Static content (default) | Most elements |

This maps to Android's `liveRegion` — a **declarative** API attached to the element itself, so it auto-announces every time the content changes. iOS handles the same thing imperatively (`AccessibilityNotification.Announcement`), which is one reason iOS and Android specs diverge here.

### TalkBack Example — The Full Spoken Sentence

Assemble the announcement in order: **Description, State, Element Type, Action hint**.

| Scenario | TalkBack Example |
|----------|------------------|
| Plain text | `Limited time offer` |
| Section heading | `Your Benefits, heading` |
| Close button | `Close, button. Double tap to activate.` |
| External link button | `Subscribe now, button. Double tap to activate. Opens external browser.` |
| Image | `Free trial offer timeline, image` |
| Accordion (collapsed) | `Learn more, Collapsed, button. Double tap to expand.` |
| Switch (on) | `Notifications, switch, on. Double tap to toggle.` (State left blank — auto-announced) |
| Audio player | `Now playing, Playing, 2:15 of 4:30, button. Double tap to pause.` |

Read each example aloud. If it sounds awkward, rewrite the description.

## How TalkBack Differs from VoiceOver

If you're creating tables for both platforms, here are the key differences. Use this to decide when an iOS spec can copy across and when it needs a different decision on Android.

| Concept | iOS (VoiceOver) | Android (TalkBack) | Parity |
|---------|----------------|-------------------|--------|
| What to call an element | `accessibilityLabel` — same text fills both specs | `contentDescription` — same text, but **never** include element type or state | Same decision |
| Current state | `accessibilityValue` — measured values only; binary states (Selected, Disabled) live in traits | `stateDescription` — handles measured values **and** binary conditions in one field | Differs |
| Element type | `accessibilityTraits` — one list mixing type tags (`.button`) and condition tags (`.selected`) | Split: `Role` ("Button") + `stateDescription` ("Selected") | Differs |
| Heading | `accessibilityHeading` trait — supports H1–H6 levels | `heading()` semantic — no levels, just yes/no | Differs |
| Actions | `accessibilityCustomActions` — single list, user cycles via rotor; no primary/extras distinction | `onClickLabel` (first action, bound to tap) + `customActions` (rest, in swipe menu). Engineer splits the list. | Differs |
| Hint | `accessibilityHint` — spoken after a pause; users can disable globally | **No equivalent** — fold the hint into the description or actions, or drop it | iOS only |
| Grouping | `accessibilityElement(children:)` — three modes: `.combine`, `.contain` (drill-in), `.ignore` | `mergeDescendants = true` — single boolean. No native equivalent to iOS's drill-in container. | Differs |
| Hidden | `accessibilityHidden` — hides element and descendants | `clearAndSetSemantics { }` — empty block clears all semantic info | Same effect |
| Announce on change | `AccessibilityNotification.Announcement` — posted imperatively when content changes (plus `.screenChanged`, `.layoutChanged`) | `liveRegion = Polite / Assertive` — declarative; attached to the element, auto-announces on change | Differs |
| Reading order | `accessibilitySortPriority` — higher reads first, default 0. Rare to override. | `traversalIndex + isTraversalGroup` — lower reads first, default 0f. Needs `isTraversalGroup = true` on a common parent for reliable behavior. | Differs |
| Focus on appearance | `accessibilityFocused` + `UIAccessibility.post` — explicit, can target any element | `requestFocus()` on a `FocusRequester` — same capability, different API shape. Easy to forget on Android. | Differs |
| Gesture for secondary actions | Rotor (two-finger twist) | Swipe up or down | Differs |
