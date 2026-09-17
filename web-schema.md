# Web (Semantic HTML) — Table Schema

Despite the title, this schema is not an ARIA authoring guide. The priority order is
semantic HTML first — a real `<button>`, `<nav>`, `<h2>` — with ARIA reached for only
where semantic HTML has no equivalent (a custom switch, a live region, a landmark
label). Calling this table "Web (ARIA)" would tell engineers the deliverable is an
ARIA spec, when the honest summary of what's actually being specified is closer to
"semantic HTML, with ARIA only where noted" — see the [Role Guide](#role-guide) below,
which opens with exactly that preference.

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

## The 11 Columns

**No Layer column on a pure Web spec.** Layer (`Native`/`Web`) only earns its place
when a table actually mixes rows owned by two different teams — the Hybrid combined
table in [SKILL.md § Hybrid Screens](SKILL.md#5-hybrid-screens--fill-in-the-layer-column).
On a pure Web spec every single row is `Web`, so the column carries zero information
and just adds a column engineering has to skim past on every row. If this table is a
*supplementary* Web-detail table referenced from a Hybrid combined table's Notes, it
still doesn't need Layer either — that combined table already carries Layer for the
seam; this table's whole reason for existing is that it's all Web.

| # | Column | What to Write | Examples |
|---|--------|--------------|----------|
| 1 | **Order** | Number each element in **reading order** — the order the virtual cursor visits elements, which is **DOM order**, not visual order. Leave blank for `Merged into parent` rows and `Hidden: Yes` rows. See "Order — DOM vs. Tab Order vs. Visual Order" below before numbering anything. | `1`, `2`, *(blank)* |
| 2 | **Component** | Short name for the element — match the component/dev-tool name when possible | `Close button`, `Page header`, `Price card` |
| 3 | **Role** | The accessible role of a **linear focus stop** — something a user reading straight through the page actually lands on: a real HTML element (preferred) or an explicit `role` attribute. See Role Guide below. **For `heading`, always write the exact level** (`heading 2`, not just `heading`) — see "Heading Levels" below, this is the single highest-value thing to get right on a web spec. **Landmarks (`main`, `navigation`, `banner`, `contentinfo`, `region`) do NOT get their own row here** — a landmark is a structural boundary a user jumps to via a separate navigation index, not a stop in linear reading order, so forcing it into a row with placeholder `Actions: None` / `State: none` misrepresents it as something it isn't. Track landmark membership in a companion **Landmark Map** instead — see "Landmarks" below. | `none`, `button`, `heading 2`, `link` |
| 4 | **Accessible Name** | What the screen reader reads as the element's name. Write `none` if the visible text, `alt`, or associated `<label>` already gives a clear name. | `none`, `Close`, `Free trial offer timeline` |
| 5 | **State** | Current state if it changes. **Leave truly blank** (not `none`) for native form controls — a real `<input type="checkbox">`, `<input type="radio">`, `<select>` — the browser announces their state automatically from the element itself, and adding text risks a double announcement. For any custom ARIA widget (a `div` styled as a switch, a custom combobox), state is **never** automatic — write it explicitly. | *(blank)*, `expanded`, `checked`, `current page` |
| 6 | **Grouping** | Always `Standalone`. Pure Web specs don't use native's `Parent of N (combined)` / `Merged into parent` model — every element a user can land on gets its own row and its own Order number, even inside a card that reads as one visual unit. See "Grouping — Always Standalone on Web" below. | `Standalone` |
| 7 | **Hidden** | `Yes` for decorative elements removed from the accessibility tree. Engineers apply `aria-hidden="true"` (or `display:none`/`visibility:hidden`, which remove it as a side effect). Distinguish this from the opposite pattern — visually-hidden-but-accessible text (a `.sr-only` class) — which is a `Hidden: No` row with an explicit **Accessible Name** and nothing visible on screen. | `No`, `Yes` |
| 8 | **Actions** | Every way to activate the element, in priority order. **Every action must have a keyboard path** — this is non-negotiable on the web (WCAG 2.1.1): if it responds to click or hover, it must also respond to a key. `Enter`/`Space` for buttons, `Enter` for links, arrow keys for composite widgets (tabs, listbox, menu) using a roving-tabindex pattern. Use `None` for non-interactive elements. | `Enter or Space activates`, `Arrow keys move between tabs`, `None` |
| 9 | **Announce on change** | `Polite` = waits for a pause (toasts, confirmations). `Assertive` = interrupts immediately (errors, urgent updates). `None` = static content. | `None`, `Polite`, `Assertive` |
| 10 | **Web example** | The full sentence the screen reader speaks, in order: Accessible Name, Role, State. Read it aloud to check it sounds natural. | `Subscribe now, button.` |
| 11 | **Notes** | Guidance for engineers — design intent no other column captures. Use `•` bullet points. Write just `•` if none needed. | `• Prefer a native <button> over a div with role="button" and a click handler` |

> **Column order matches the iOS and Android templates** wherever the concept exists
> (minus Layer, which Web drops), so specs still review side-by-side. Web shares
> TalkBack's shape most closely: both drop Hint and use Announce on change in its place.

## Role Guide

Prefer a real HTML element over an ARIA role bolted onto a `div` — this is the
["first rule of ARIA use"](https://www.w3.org/TR/using-aria/#rule1): a native `<button>`
gets keyboard support, role, and state for free, while `role="button"` on a `div`
requires you to hand-build all three yourself, and it's easy to miss one.

| Role | Use When... | What's Announced |
|------|------------|-------------------|
| `none` | Plain text — a paragraph, a label, a description | Nothing extra |
| `heading` | A section title (`<h1>`–`<h6>`, or `role="heading" aria-level`) | "heading" plus its **level** — `"heading 2"` — the one thing native headings can't do. Never write a bare `heading` in this schema — always the number. |
| `button` | Performs an in-page action | "button" |
| `link` | Navigates to a new URL or place in the page — in-page, to another page, or off-site | "link" — see "Links" below, the Accessible Name must make sense out of context, in the page's link list |
| `image` | A meaningful `<img>`, `<svg role="img">`, or CSS background promoted via `role="img"` | "image", from its `alt` |
| `textbox` | A text input or `<textarea>` | "edit text" / "text field", plus its label |
| `checkbox` | A checkable option in a multi-select group | "checkbox" + checked state |
| `radio` | An option in a single-select group | "radio button" + selected state |
| `switch` | A custom on/off control styled as a toggle | "switch" + on/off state — **never automatic**, always author `aria-checked` |
| `tab` | A tab in a tablist | "tab" + selected state |

## Heading Levels — The Highest-Value Thing to Get Right on a Web Spec

This is the one column where getting it right matters more than everywhere else in
this schema combined. Headings are how a screen reader user *skims* a web page — most
open a page and immediately pull up a list of every heading to decide where to go,
the same way a sighted user's eye jumps straight to bold section titles. A missing or
wrong level breaks that skim for the entire page, not just one row.

**Unlike a landmark (see below), a heading genuinely is a row in the per-element
table** — a user reading straight through the page actually lands on it, same as a
button or a link. So heading level stays in that table's Role column. But on any page
with more than a handful of headings, also produce a standalone **Heading Outline** —
an indented list (`h1 > h2 > h3...`) built purely by walking the table's Role column
top to bottom and pulling out every heading row. That outline is where a nesting
mistake (a skipped level, two headings competing at the same level) actually becomes
visible; it's easy to miss a single wrong `heading 3` sitting among 50 other rows, and
obvious the moment it's pulled into its own list, the same way it would be obvious to
the screen reader user skimming the real page. Deliver it alongside the row table, not
instead of it — see [SKILL.md](SKILL.md)'s Heading Outline and Landmark Map guidance.

**Levels must nest without skipping**, the same way you wouldn't skip from an `<h1>` to
an `<h4>` in a document outline:

- Exactly **one `h1` per page** — the page's own title. Not the site name/logo (that's
  a landmark's label, not a heading), not the section title of the first module. HTML
  technically permits more than one (each `<section>`/`<article>` could restart at
  `h1` under the old sectioning-based outline algorithm), but no browser or screen
  reader ever implemented that algorithm, and it's since been dropped from the spec —
  every screen reader just flattens all headings into one linear list regardless of
  nesting. Two `h1`s in that list look like two competing "the title of this page,"
  with nothing to say which one actually is. Treat "one `h1`" as a hard rule, not a
  guideline with exceptions.
- Each subsection increases by exactly one level from its parent: an `h1` page title
  with `h2` section titles, each containing `h3` sub-headings, and so on. Never jump
  from `h2` straight to `h4` because it "looks right" visually — visual weight (font
  size, bold) and heading level are two different systems that happen to often
  correlate; a design can use a big bold style for something that is not structurally
  a heading at all (see below), or a small label for something that structurally is.
- A repeated module (a story card, a package of "In Case You Missed It" links) reuses
  the **same level every time it repeats**, whatever level is correct for its place in
  the outline — don't let one instance drift to a different level than its siblings.

**Every row that is visually a bold section title needs a Role/level decision, not an
assumption.** Walk the page and ask, for each candidate: does this genuinely start a
new section a user would want to jump to (`heading N`), or is it styled boldly for
emphasis without functioning as a navigation target (`none`)? A "SPONSOR CONTENT"
eyebrow label above a headline, for instance, is usually **not** its own heading level —
it's more often part of that headline's accessible name or a `none` row directly before
it, not a separate `h3` competing with the headline for the same list position.

**Write the exact level you intend, and say why in Notes when it's not obvious from
position alone** — e.g. `• This card repeats 6 times down the page; every instance is
h3, one level under the "Opinion" package's h2` — so engineering doesn't have to guess
at the outline from a screenshot.

**A card headline is very often both a heading and a link at once** — e.g.
`<h3><a href="...">Headline text</a></h3>` — not one or the other. This schema's Role
column only holds one value, so write `heading 3` (the structural fact that matters
for page skimming) and add `• Also the card's link to the full story` in Notes, rather
than writing `link` and losing the heading level, or vice versa. Don't treat this as
an either/or decision.

## Landmarks — Belongs in a Map, Not a Row

Landmarks let a screen reader user jump straight to `main`, skip repeated navigation,
or ask "what page regions exist here" the way a sighted user's eye does from layout
alone. A page with zero landmarks forces every user to walk the entire DOM linearly
with no way to skip past a large nav or a long list of unrelated cards.

**A landmark is not a row in the per-element table.** The table's Order/Role/Actions
columns describe **linear focus stops** — things a user reading straight through the
page actually lands on. A landmark isn't one of those: it's a structural boundary
announced on entry, and it's the anchor for a *separate*, parallel navigation index
(jump straight to `main`) that exists alongside — not inside — linear reading order. A
landmark row with `Actions: None` and blank `State` misrepresents it as a focus stop
it isn't, and worse, it steals an Order number from something that actually needs one.

Track landmark membership in a standalone **Landmark Map** instead — which region
each part of the page belongs to, independent of the row table. See
[SKILL.md](SKILL.md)'s Heading Outline and Landmark Map guidance for how to structure
and deliver it. What follows here is what a spec author needs to know to build that
map correctly — not row-by-row column guidance.

**Every page needs, at minimum:**

| Landmark | Use for | Notes |
|----------|---------|-------|
| `banner` | The page masthead/header, once per page | Only the *global* site header — see [SKILL.md](SKILL.md)'s shared-web-chrome exclusion; if the masthead itself is out of scope for this spec, its landmark role still matters to note once, since it changes how many `main`-adjacent siblings exist |
| `navigation` | Each distinct block of navigation links — primary nav, footer nav, a package's "jump to section" list | If a page has more than one, each needs a distinct **Accessible Name** (`aria-label="Primary"` vs. `aria-label="Footer"`) — otherwise a screen reader user hears "navigation, navigation" with no way to tell them apart when jumping between landmarks |
| `main` | The primary content of the page, exactly once | If `main` is missing, a screen reader user has no way to skip straight past the header/nav to the actual content — flag this as a real gap, not a nice-to-have |
| `contentinfo` | The page footer, once per page | |
| `region` / `complementary` | A named secondary section worth jumping to directly (a sidebar, a "Related Coverage" box) | Needs an explicit **Accessible Name** via `aria-label` or `aria-labelledby` — an unlabeled `region` is close to useless in landmark navigation, since every unlabeled one just announces "region" with nothing to distinguish it |
| `search` | A search form/widget | |

**Don't over-landmark.** Wrapping every card or every visual box in its own `region`
defeats the purpose — landmark navigation is supposed to jump between a *small* number
of major areas, not replicate the full visual hierarchy. If a page would end up with
15+ landmarks, that's a signal most of them should be `none`/plain divs instead, with
headings (not landmarks) doing the work of marking subsections.

**A repeated module is usually a heading, not a landmark.** A story card that repeats
20 times down a homepage should NOT be 20 `region`s — that many identically-purposed
landmarks makes the landmark list itself the thing a user has to skim through
linearly, defeating the point. Give each card a heading at the right level instead (see
above), and reserve landmarks for the page's small number of major, structurally
distinct areas.

**The NYT homepage template is a real counter-example worth knowing, not a rule to
copy blindly**: each topic-tag row ("U.S. Economy", "2026", "War in the Middle East")
that sits above a package's lead story is, on the live site, its own `navigation`
landmark labeled with that package's name — not plain links with no landmark wrapper.
This is a deliberate exception to "a repeated module is usually a heading, not a
landmark" above: these rows *are* navigation (a set of jump-to-topic links), so
`navigation` is the correct role even though the pattern repeats many times down the
page. Don't assume from the "don't over-landmark" guidance that repetition alone rules
out `navigation` — check what the row actually does (navigate to a different topic
page) rather than just how often it repeats. An interactive embed (a poll chart, a
video carousel) inside a package can also get its own labeled `region` (e.g.
`aria-label="gallery"`) for the same reason a sidebar would.

**A landmark's `aria-label` and its actual role can drift apart — verify, don't
assume.** The live NYT homepage's primary nav bar is a `navigation` landmark whose
`aria-label` is literally `"main"` — a leftover or copy-paste label that has nothing to
do with the `main` landmark role, and collides with it in a landmark reader's list
(two entries that both say "main" with different roles). If you're documenting an
existing live page rather than a new design, get the labels from a real accessibility
inspector or the actual markup — don't infer them from the visual package title, since
the shipped label can be stale, wrong, or copied from something else entirely. Flag
a mismatch like this explicitly in the Landmark Map's notes rather than silently
"fixing" it to what the label should say.

## Links — Getting Them Right Matters as Much as Headings

Just like the heading list, most screen readers let a user pull up a standalone list of
every link on the page. That list is only useful if each entry makes sense **on its
own, out of context** — with no surrounding sentence, no card layout, no visual
proximity to a photo. Get this wrong across a whole page (ten links that all say "Read
more") and the link list becomes useless, the same failure mode as a heading list full
of the wrong levels.

**Every link's Accessible Name must say where it goes, without relying on
surroundings.** This is a real WCAG success criterion (2.4.4, Link Purpose in
Context — and the stronger 2.4.9, Link Purpose, no context, for AA+ work), not a
stylistic preference:

- ❌ `Read more` / `Click here` / `Learn more` repeated across many cards — identical
  in the link list, indistinguishable from each other once pulled out of their cards.
- ✅ `Read more about the government shutdown` — or, more simply, make the **headline
  itself** the link (see the heading-and-link pattern above) so its name already says
  what the story is, and drop the redundant "Read more" text entirely, or `aria-hidden`
  it if it needs to stay visible for design reasons with the real name carried by the
  headline.

**Icon-only links need an explicit Accessible Name, the same as icon-only buttons.** A
bare `<a href="...">` wrapping only an SVG or icon font glyph has no accessible name at
all unless one is authored — screen reader users hear either the raw URL or nothing
useful. `aria-label="Share this article"` on the anchor itself, not just an
adjacent visible label.

**Adjacent links to the same destination need a decision, not an accident.** The story
card pattern from "Grouping — Always Standalone on Web" above — headline link, plus a
separately-linked thumbnail image pointing at the same story — is extremely common in
editorial layouts and is *fine* on the web, unlike native's combined-row model. But
call out explicitly in Notes whether that's the intended pattern (two focus stops,
same destination) or whether the image should NOT independently link (headline is the
only link; image is `Actions: None`) — don't let it default silently one way or the
other, since it changes a real row's Actions value.

**Distinguish link from button by what happens, not by what it looks like.** A
button-styled anchor is a common source of confusion in both directions:
- **Navigates to a new URL or a new place in the page** (even via `#anchor`) → `Role:
  link`, regardless of whether it's styled to look like a button.
  Actions: `Enter activates`.
- **Performs an action in place — submits, toggles, opens a dialog, without changing
  the URL** → `Role: button`, regardless of whether it's styled to look like a text
  link. Actions: `Enter or Space activates`.

Flag it in Notes whenever the visual style and the underlying behavior disagree (a
link-styled "Sign out" that's actually a form-submitting button, or a button-styled
"View all comments" that's actually a same-page anchor jump) — that mismatch is a sign
the design and the correct ARIA role are about to diverge, and it's cheaper to catch
before implementation than after.

**A link opening in a new tab or window needs that stated as part of the name**, not
left implicit — screen reader users get no visual cue (no new browser chrome flash)
the way sighted users do, so losing their place unexpectedly is disorienting. Convention
is a trailing screen-reader-only span: `<a href="...">Subscriber FAQ<span
class="sr-only">, opens in a new tab</span></a>`, giving an Accessible Name like
`Subscriber FAQ, opens in a new tab`. Note this in the row rather than assuming
engineering will remember — it's easy to add `target="_blank"` without it.

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

**A real example of visual order lying about DOM order**: the NYT homepage template
reads top-to-bottom across visual bands, then **right-to-left** within a band — not
the left-to-right a sighted scan would suggest. See
[SKILL.md](SKILL.md#row-order--priority-not-position)'s Row Order exception for the
full rule (including why a wide lead package still outranks right-to-left against a
narrower adjacent module in the same row). Verify this kind of template-specific
convention against the real page rather than assuming a generic default — it's exactly
the sort of thing that's invisible from a screenshot.

**The mismatch can also be deliberate, not accidental — and that's worth designing for.**
A story/article card typically shows the photo above the headline visually. Give the
headline an earlier **Order** number than the photo anyway — put it before the photo
in DOM order, with CSS reordering the photo back on top visually — so a screen reader
user hears what the story is about immediately, instead of sitting through a photo
description first. This holds whether the card exposes one link or several standalone
rows (see "Grouping — Always Standalone on Web" below): either way, order the headline
ahead of the photo. This is the one case where you *want* DOM order to diverge from
visual order — call it out explicitly in Notes so a later reviewer doesn't "fix" it by
matching the two back up.

### Grouping — Always Standalone on Web

Every Grouping cell is `Standalone`. Pure Web specs skip the native `Parent of N
(combined)` / `Merged into parent` model entirely — there's no row that "consumes" a
sibling's Order number. Give every text node, image, and link its own row and its own
Order number, even when several of them make up what looks like one card.

This means a story/article card — photo, headline, dek, read time — is **several
standalone rows**, not one combined row with the rest merged into it:

| Order | Component | Role | Accessible Name | Actions |
|-------|-----------|------|------------------|---------|
| 1 | Story headline | `link` | A 20-Minute Workout to Build Upper-Body Strength | `Enter activates` |
| 2 | Summary / dek | `none` | none | `None` |
| 3 | Read time | `none` | none | `None` |
| 4 | Photo credit | `none` | none | `None` |
| 5 | Thumbnail image | `image` | A man performs a resistance band exercise against a pink background | `None` (or `Enter activates` if the image is also wrapped in a link to the same story) |

Two things carry over from the combined version without needing a Grouping value:

- **Headline before photo, still.** The headline gets the earliest Order number in the
  card (`1`) even though the photo displays above it — same reasoning as the DOM-order
  note above, just expressed as row order instead of word order inside one name.
- **The image keeps a real Accessible Name.** It's informative, not decorative — see
  the story-card row in [SKILL.md](SKILL.md)'s Quick Reference table.

If only the headline (or only the image) is wrapped in a link — the far more common
editorial pattern than one link swallowing the whole card — say so directly in Notes,
since it changes which row(s) carry an Action and whether two rows land on the same
destination (worth a one-line flag, not a redesign: two focus stops that both lead to
the same story is normal on the web, unlike a single card-length link with a
paragraph-long spoken name).

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
- `• This is h3, one level under the "Opinion" package's h2 — every repeat of this card stays h3`
- `• "Primary" and "Footer" nav landmarks need distinct aria-labels so they're not both announced as just "navigation"`
- `• Live region container must exist in the DOM before the confirmation text is inserted`
- `• Opens in a new tab — accessible name needs ", opens in a new tab" appended, e.g. via a visually-hidden span`
- `• This "Read more" link should have a real destination name, not literally "Read more" — it's indistinguishable from every other card's link in the page's link list`
- `• Styled as a button but submits no form and changes no state — this is actually a link (Role: link, Actions: Enter activates), not a button`

Avoid: `• Add aria-label="Close"` (that's the engineer's implementation, not your intent).

## How Web Differs from Native

| Concept | Web | Native (iOS/Android) |
|---------|-----|----------------------|
| What announces the element | Browser builds an accessibility tree from the DOM + ARIA, and the screen reader walks *that* tree | The app's UI framework builds the tree directly from view properties; no intermediate document |
| Reading order | DOM order — can silently diverge from visual order via ordinary CSS (flex/grid `order`, `position`) | Visual position by default; diverging requires a deliberate extra property (`accessibilitySortPriority` / `traversalIndex`) |
| Headings | Carry real levels — `heading 1`–`heading 6`, which most users navigate a page by skimming; a wrong or skipped level breaks that skim page-wide | No levels (Android `heading()` is yes/no; iOS heading trait has no level) |
| Landmarks | `main`/`navigation`/`banner`/`region`/etc. — a distinct jump-navigation layer alongside headings; too many defeats the purpose | No equivalent — iOS's rotor and Android's heading list only expose headings, not a separate landmark layer |
| Automatic state | Only for genuine native HTML controls (`<input>`, `<select>`) — custom ARIA widgets always need explicit state | Native OS controls (`Switch`, `Checkbox`) always auto-announce state regardless of custom styling |
| Hiding | `aria-hidden` (tree only) vs. `display:none`/`visibility:hidden` (tree + visual) — two different tools for two different intents | `accessibilityHidden` / `clearAndSetSemantics` — one tool, tree only |
| Keyboard operability | A first-class, explicit requirement (WCAG 2.1.1) — every interaction needs a keyboard path | Governed by touch gestures and the OS AT's own gesture vocabulary (double-tap, swipe) — not something the app individually re-implements |
| Focus indicator | Designable and stylable (`:focus-visible` outline) — a real design decision that can be gotten wrong | Handled by the OS's own focus highlight — not a per-app design decision |
| Link naming | Must make sense pulled out of context in a standalone link list (WCAG 2.4.4/2.4.9) — a page of "Read more" links is a real failure mode | No page-wide link list exists to pull names out of context for; a native label only has to make sense in place |
| Live/dynamic updates | `aria-live` — declarative, attached to a container, with real cross-browser inconsistency | `liveRegion` (Android, declarative) or `AccessibilityNotification.Announcement` (iOS, imperative) |
