# iOS VoiceOver — Table Schema

## How VoiceOver Reads an Element

When a user swipes to an element, VoiceOver speaks in this order:

> **Label → Value → Type → Hint**

For example, a subscribe button might read:
> *"Subscribe now, button. Opens an external website."*

That breaks down as: Label ("Subscribe now") → Type ("button") → Hint ("Opens an external website").

## The 9 Columns

| # | Column | What to Write | Examples |
|---|--------|--------------|----------|
| 1 | **Order** | Number each element in the order a user swipes through the screen (top-to-bottom, left-to-right) | `1`, `2`, `3` |
| 2 | **Component** | A short name for the element — match the Figma layer name when possible | `Close button`, `Page header`, `Price information` |
| 3 | **Trait** | What type of element is this? (see Trait Guide below) | `none`, `header`, `button`, `image` |
| 4 | **Label** | What VoiceOver reads as the element's name. Write `none` if the visible text on screen is already clear enough. | `none`, `Close`, `Free trial offer timeline` |
| 5 | **Actions** | What happens when the user double-taps? Write `none` for non-interactive text. | `none`, `Double tap opens external checkout page` |
| 6 | **Hint** | Extra context spoken after the type — usually what will happen. Write `none` if unnecessary. | `none`, `Opens an external website.`, `Shows or hides subscription details.` |
| 7 | **Value** | The current state of the element, if it changes. Write `none` for static elements. | `none`, `expanded`, `collapsed`, `1 of 6` |
| 8 | **Example** | The **full sentence VoiceOver speaks aloud**. This is the most important column — read it aloud to verify it sounds natural. | `Subscribe now, button. Opens an external website.` |
| 9 | **Notes on Documentation** | Guidance for engineers. Use `•` bullet points. Write just `•` if none needed. | `• Warn user before opening external browser` |

## Trait Guide

Think of traits as telling VoiceOver "what kind of thing is this?"

| Trait | Use When... | What VoiceOver Adds |
|-------|------------|-------------------|
| `none` | It's plain text — a paragraph, a label, a description | Nothing extra |
| `header` | It's a section title. This lets users jump between sections using the rotor. | Says "heading" after the text |
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

### Value — Communicating State

Values tell users "where am I?" or "what mode is this in?"

| Element | Value When Active | Value When Inactive |
|---------|-----------------|-------------------|
| Accordion | `expanded` | `collapsed` |
| Toggle/switch | `on` | `off` |
| Carousel item | `1 of 6` | (changes per item) |
| Tab | `selected` | (not focused) |

### Example — Assembling the Announcement

Combine the non-`none` values from Label, Value, Trait, and Hint:

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
