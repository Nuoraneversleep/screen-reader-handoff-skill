# iOS VoiceOver — Table Schema

## How VoiceOver Reads an Element

When a user swipes to an element, VoiceOver speaks in this order:

> **Label → Value → Type → Hint**

For example, a subscribe button might read:
> *"Subscribe now, button. Opens an external website."*

That breaks down as: Label ("Subscribe now") → Type ("button") → Hint ("Opens an external website").

`Grouping` and `Hidden` don't add words to the announcement — they decide *whether* and *how* an element is focused in the first place.

## The 11 Columns

| # | Column | What to Write | Examples |
|---|--------|--------------|----------|
| 1 | **Order** | Number each element in swipe order (top-to-bottom, left-to-right). **Only number what VoiceOver actually focuses on:** standalone elements and parents. Leave blank for `Merged into parent` rows and `Hidden: Yes` rows — those don't get read as their own focus stops. | `1`, `2`, *(blank)* |
| 2 | **Component** | Short name for the element — match the Figma layer name when possible | `Close button`, `Page header`, `Price information` |
| 3 | **Trait** | What type of element is this? (see Trait Guide below) | `none`, `header`, `button`, `image` |
| 4 | **Label** | What VoiceOver reads as the element's name. Write `none` if the visible text is already clear. | `none`, `Close`, `Free trial offer timeline` |
| 5 | **Value** | Current state of the element if it changes. `none` for static elements. | `none`, `expanded`, `collapsed`, `1 of 6` |
| 6 | **Grouping** | `Standalone`, `Parent of N (combined)`, `Parent of N (drill-in)`, or `Merged into parent`. See Grouping below. | `Standalone`, `Parent of 3 (combined)` |
| 7 | **Hidden** | `Yes` for decorative elements that should be removed from VoiceOver. Engineers apply `accessibilityHidden = true`. | `No`, `Yes` |
| 8 | **Actions** | What happens on double-tap. `none` for non-interactive text. | `none`, `Double tap opens external checkout page` |
| 9 | **Hint** | Extra context spoken after the type — usually what will happen. `none` if unnecessary. | `none`, `Opens an external website.`, `Shows or hides subscription details.` |
| 10 | **Example** | The **full sentence VoiceOver speaks aloud**. Read it aloud to verify it sounds natural. | `Subscribe now, button. Opens an external website.` |
| 11 | **Notes on Documentation** | Guidance for engineers. Use `•` bullet points. Write just `•` if none needed. | `• Warn user before opening external browser` |

> **Column order matches the TalkBack template.** Same name → same column position. Both templates share the `Order` column (number focus stops, blank for merged/hidden rows). The Android version drops `Hint` and adds `Announce on change` in its place; `Notes` is the last column in both. Everything else lines up so you can review iOS and Android specs side-by-side.

## Trait Guide

Think of traits as telling VoiceOver "what kind of thing is this?"

| Trait | Use When... | What VoiceOver Adds |
|-------|------------|-------------------|
| `none` | It's plain text — a paragraph, a label, a description | Nothing extra |
| `header` | It's a section title. Lets users jump between sections using the rotor. | Says "heading" after the text |
| `button` | The user can tap it to do something | Says "button" after the label |
| `image` | It's a meaningful image, illustration, or animation | Says "image" after the label |
| `link` | It's a tappable hyperlink | Says "link" after the text |
| `Adjustable` | It's a carousel, slider, or stepper where swiping up/down changes the value | Says "adjustable" and enables swipe gestures |

You can combine traits when needed: `button, header` for a tappable section header.

## Column-by-Column Guidance

### Label — When to Write One vs. Use `none`

**Use `none` when** the text visible on screen is self-explanatory:
- "Subscribe to gain unlimited access to all of The Times." → `none` (perfectly clear)
- "Limited time offer" → `none` (reads fine as-is)

**Write an explicit label when:**
- The visible text alone is confusing: an "X" icon should be labeled `Close`
- An animation needs description: `Free trial offer timeline`
- A price button shows "$30 ~~$4/month~~" — label it `$30, discounted to $4 per month`
- Dynamic content: use brackets like `[Terms & Conditions copy]`

### Hint — Best Practices

Hints give users a preview of what will happen. Keep them short and actionable.

| Scenario | Good Hint |
|----------|-----------|
| Button opens Safari/external browser | `Opens an external website.` |
| Accordion expand/collapse | `Shows or hides subscription benefit details.` |
| Text block contains tappable links | `Use the rotor to access links` |
| Carousel / adjustable | `Swipe up or down with one finger to navigate` |
| Simple in-app navigation | `none` (no hint needed — tapping a button is expected) |

**Don't over-hint.** If tapping a "Continue" button just goes to the next screen, `none` is fine. Hints are for *unexpected* outcomes.

> **Android has no hint.** When porting iOS specs to TalkBack, fold the hint information into the Description or Action column, or drop it.

### Value — Communicating State

Values tell users "where am I?" or "what mode is this in?"

| Element | Value When Active | Value When Inactive |
|---------|-----------------|-------------------|
| Accordion | `expanded` | `collapsed` |
| Toggle/switch | `on` | `off` |
| Carousel item | `1 of 6` | (changes per item) |
| Tab | `selected` | (not focused) |

> **iOS convention is lowercase** ("collapsed", "on"). Android convention is capitalized ("Collapsed", "On"). Same meaning, different convention.

### Grouping — How Children Behave

Maps to `accessibilityElement(children:)`. iOS supports more options than Android — the drill-in mode is iOS only.

| Value | iOS API | Meaning | When to Use |
|-------|---------|---------|-------------|
| `Standalone` | (default) | Element stands alone | Solo buttons, single text labels |
| `Parent of N (combined)` | `.combine` | Merges N children into one announcement | A card with title + price + image — read as one block |
| `Parent of N (drill-in)` | `.contain` | Group is a single focus stop, but children stay individually navigable inside | A complex card where the user might want to drill in to a specific element |
| `Merged into parent` | (child of `.combine`) | This element is consumed by its parent | The title, price, image inside the card above |

**iOS only**: `Parent of N (drill-in)` has no Android equivalent. When porting to TalkBack, fall back to `Parent of N` (combined) or `Standalone` and explain the trade-off in the Notes column.

**Order numbers**: only the parent row gets an Order number. `Merged into parent` rows skip Order — they're listed for engineer clarity but aren't separate focus stops.

| Order | Component | Trait | Label | Grouping |
|-------|-----------|-------|-------|----------|
| 7 | Plan card | none | All Access. News, plus Games, Cooking, Audio, Wirecutter and The Athletic. | Parent of 2 (combined) |
|   | Plan title | none | none | Merged into parent |
|   | Plan description | none | none | Merged into parent |

### Hidden — Remove from VoiceOver

Mark `Yes` for:
- Decorative icons that have a labeled sibling (a heart icon next to "Save" button text)
- Redundant labels
- Background images and dividers

Engineers will apply `accessibilityHidden = true`, which hides the element and all its descendants. Same effect as Android's `clearAndSetSemantics { }`.

**Order numbers**: leave blank for `Hidden: Yes` rows — VoiceOver skips them, so they don't take a position in the swipe sequence.

### Example — Assembling the Announcement

Combine the non-`none` values from Label, Value, Trait, and Hint. **Grouping and Hidden don't appear in the Example column** — they affect what's focusable, not what's spoken.

| Scenario | Example |
|----------|---------|
| Plain text | `Limited time offer` |
| Section header | `Your Benefits, heading` |
| Close button | `Close, button` |
| External link button | `Subscribe now, button. Opens an external website.` |
| Image | `Free trial offer timeline, image` |
| Accordion (collapsed) | `Learn more, collapsed, button. Shows or hides subscription benefit details.` |
| Carousel | `Subscriber value carousel, containing 6 items, adjustable. 1 of 6, essential reporting. Swipe up or down with one finger to navigate.` |
| Text with link | `Your subscription will continue until you cancel, link, use the rotor to access links.` |

### Notes — What to Tell Engineers

Keep notes focused on **design intent**, not code. Engineers know their APIs — they need to know **what you want to happen**.

Good notes:
- `• Warn user before opening external browser`
- `• Treat the animation as a single image, not individual frames`
- `• The cancel link inside the Terms text should be accessible via rotor`
- `• Read the full carousel card header when user swipes to next card`
- `• Decorative checkmark icons should not be read aloud`

Avoid: `• Use accessibilityLabel = "Close"` (that's the engineer's job)
