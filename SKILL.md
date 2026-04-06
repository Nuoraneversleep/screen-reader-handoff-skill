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
Step 1: Call Figma MCP `get_file` or `get_node` to retrieve the screen structure.
Step 2: If that fails, try `get_metadata` for an overview, then `get_design_context`
        with forceCode: true for detailed node data.
Step 3: Walk the node tree top-to-bottom, left-to-right to identify every visible
        element: text, buttons, images, inputs, toggles, carousels, links.
```

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
- Order rows by **navigation sequence**: top-to-bottom, left-to-right (how a user would swipe through the screen).
- Use `none` (lowercase) for empty cells — never leave cells blank.
- Use `[brackets]` for content that changes at runtime.
- Keep multi-line notes on a single line using ` • ` as separator.

### 5. Quality Check

After generating, verify:
- [ ] Every tappable element is marked as a button or link
- [ ] Section headings are marked as headers (enables "jump to heading" navigation)
- [ ] Every button describes what happens when tapped
- [ ] External links warn the user they'll leave the app
- [ ] Toggles/accordions communicate their state (expanded/collapsed, on/off)
- [ ] Decorative images are excluded (not in the table)
- [ ] The **Example** column reads naturally — read it aloud to check
- [ ] No blank cells — use `none` where not applicable

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
