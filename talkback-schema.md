# Android TalkBack — Table Schema

## How TalkBack Reads an Element

When a user swipes to an element, TalkBack speaks in this order:

> **Content Description → State → Role → Actions**

For example, a subscribe button might read:
> *"Subscribe now, button. Double tap to activate. Opens external browser."*

That breaks down as: Content Description ("Subscribe now") → Role ("button") → Action ("Double tap to activate. Opens external browser.").

## The 9 Columns

| # | Column | What to Write | Examples |
|---|--------|--------------|----------|
| 1 | **Order** | Number each element in swipe order (top-to-bottom, left-to-right) | `1`, `2`, `3` |
| 2 | **Component** | Short name for the element — match the design layer name | `Close button`, `Page header`, `Price information` |
| 3 | **Role** | What type of element is this? (see Role Guide below) | `none`, `heading`, `button`, `image` |
| 4 | **Content Description** | What TalkBack reads as the element's name. Write `none` if the visible text is clear enough. | `none`, `Close`, `Free trial offer timeline` |
| 5 | **Actions** | What TalkBack says about available interactions | `none`, `Double tap to activate. Opens external browser.` |
| 6 | **Action Label** | A custom name for the action shown in TalkBack's menu (replaces the generic "Activate") | `none`, `Open in browser`, `Expand section` |
| 7 | **State Description** | The current state of the element, if it changes. Write `none` for static elements. | `none`, `Expanded`, `Collapsed`, `On`, `Off` |
| 8 | **Example** | The **full sentence TalkBack speaks aloud**. Read it aloud to verify. | `Subscribe now, button. Double tap to activate.` |
| 9 | **Notes on Implementation** | Guidance for engineers. Use `•` bullet points. Write just `•` if none needed. | `• Warn user before opening external browser` |

## Role Guide

Roles tell TalkBack what kind of element the user is on.

| Role | Use When... | What TalkBack Announces |
|------|------------|----------------------|
| `none` | Plain text — paragraphs, labels, descriptions | Nothing extra |
| `heading` | Section title. Lets users jump between sections. | Says "heading" |
| `button` | Tappable element that performs an action | Says "button" and "Double tap to activate" |
| `image` | Meaningful image, illustration, or animation | Says "image" |
| `link` | Tappable hyperlink | Says "link" |
| `switch` | Toggle that turns something on or off | Says "switch" and the on/off state |
| `seekbar` | Slider or progress bar the user can adjust | Says current position |
| `list` | Scrollable list or carousel | Announces item count |

## How TalkBack Differs from VoiceOver

If you're creating tables for both platforms, these are the key differences:

| Concept | iOS (VoiceOver) | Android (TalkBack) |
|---------|----------------|-------------------|
| What to call an element | **Label** | **Content Description** |
| Element type | **Trait** | **Role** |
| Current state | **Value** | **State Description** |
| What happens on interaction | **Hint** (spoken automatically) | **Action Label** (shown in menu) |
| Jump between sections | Rotor → Headings | Heading navigation (swipe up/down with reading control set to Headings) |
| Navigate carousel | Swipe up/down on Adjustable trait | Swipe left/right through list items |
| Hide decorative element | `isAccessibilityElement = false` | `importantForAccessibility = no` |

## Column-by-Column Guidance

### Content Description — Same Principles as VoiceOver Label

**Use `none` when** visible text is self-explanatory.
**Write an explicit description when** the visible text alone would be confusing (icons, complex price displays, animations).

### Actions — More Explicit Than iOS

TalkBack is more verbose about actions. Always describe the full interaction:
- Simple button: `Double tap to activate.`
- External link: `Double tap to activate. Opens external browser.`
- Accordion: `Double tap to expand.` / `Double tap to collapse.`
- Toggle: `Double tap to toggle.`

### Action Label — Custom Menu Text

When a user opens TalkBack's local context menu on an element, they see available actions. The **Action Label** replaces the generic "Activate" with something meaningful:

| Element | Default Action | Better Action Label |
|---------|---------------|-------------------|
| External link button | "Activate" | `Open in browser` |
| Accordion trigger | "Activate" | `Expand section` / `Collapse section` |
| Close button | "Activate" | `Dismiss` |
| Simple navigation button | "Activate" | `none` (default is fine) |

### State Description — Same Concept as VoiceOver Value

| Element | State When Active | State When Inactive |
|---------|-----------------|-------------------|
| Accordion | `Expanded` | `Collapsed` |
| Toggle/switch | `On` | `Off` |
| Checkbox | `Checked` | `Not checked` |

Note: TalkBack capitalizes state descriptions by convention.

### Notes — What to Tell Engineers

Same as VoiceOver: focus on **design intent**, not code.

Good:
- `• Warn user before opening external browser`
- `• Group the icon and label into a single focusable element`
- `• Hide decorative dividers from TalkBack`
- `• Announce state change when accordion expands`

Avoid: `• Use ViewCompat.setAccessibilityHeading(view, true)` (engineer's domain)
