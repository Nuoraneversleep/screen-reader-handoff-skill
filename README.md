# Screen Reader Handoff Skill

An agent skill for [Claude Code](https://claude.com/claude-code) and
[Cursor](https://cursor.com) that helps designers generate screen reader
accessibility handoff tables for iOS VoiceOver, Android TalkBack, and the Web
(ARIA).

No screen reader expertise required — the skill guides you through creating structured specs that tell engineers exactly what a screen reader user should hear on every screen.

## What It Does

- Generates **tab-separated (TSV) tables** you can paste directly into Figma or Google Sheets
- Supports **iOS VoiceOver**, **Android TalkBack**, and **Web (ARIA)** with platform-specific schemas
- Reads designs from **Figma URLs** (via MCP) or manual screen descriptions
- **Renders the spec onto the Figma canvas**, beside the screen it documents, so
  reviewers can check an announcement against the design without switching tools
- **Annotates the design itself** with numbered badges and dashed outlines tied to
  each row's Order number, so a reviewer can match a row to its element at a glance
- Orders rows by **announcement priority** rather than visual position — navigation
  chrome first, terms before the CTA they govern
- Handles **hybrid screens** (native shell + WebView) as **one combined table** — a
  single continuous Order sequence with each row marked `Native` or `Web`, instead of
  two disconnected tables that hide whether the seam actually reads in order
- Excludes **OS chrome** (status bar, home indicator, notch) — the platform owns those
- Includes annotated **best-practice examples** explaining common patterns like buttons, accordions, carousels, external links, and more

## Example Output

Each row in the table is one element a screen reader user can focus on:

| Order | Component | Layer | Trait | Label | Value | Grouping | Hidden | Actions | Hint | Example | Notes |
|-------|-----------|-------|-------|-------|-------|----------|--------|---------|------|---------|-------|
| 1 | Close button | Native | button | Close | none | Standalone | No | Double tap to dismiss | none | Close, button | Icon needs explicit label |
| 2 | Page header | Native | header | none | none | Standalone | No | none | none | Subscribe to..., heading | Enables rotor navigation |
| 5 | Learn more | Native | button | Learn more | collapsed | Standalone | No | Double tap to expand | Shows or hides details. | Learn more, collapsed, button. Shows or hides details. | Value tracks state |

## Installation

### Claude Code

```bash
# Personal — available in every project
git clone https://github.com/Nuoraneversleep/screen-reader-handoff-skill.git \
  ~/.claude/skills/screen-reader-handoff

# Or per-project, shared with your team via the repo
git clone https://github.com/Nuoraneversleep/screen-reader-handoff-skill.git \
  .claude/skills/screen-reader-handoff
```

### Cursor

```bash
git clone https://github.com/Nuoraneversleep/screen-reader-handoff-skill.git \
  ~/.cursor/skills/screen-reader-handoff
```

Either way the skill directory must contain `SKILL.md` at its root.

### Reading and writing Figma

To read designs from a Figma URL — and to render specs onto the canvas — you also
need the Figma MCP server. In Claude Code:

```bash
claude plugin install figma@claude-plugins-official
```

Then enable the MCP server in the Figma desktop app under
**Preferences → Enable local MCP server**. Without it, the skill still works from a
screenshot or a written description of the screen.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill instructions — workflow, quality checklist, common UI patterns |
| `voiceover-schema.md` | iOS VoiceOver 12-column table schema and column-by-column guidance |
| `talkback-schema.md` | Android TalkBack 12-column table schema and platform differences |
| `web-schema.md` | Web (ARIA) 12-column table schema — Role, DOM order vs. visual order, live regions, and how it differs from native |
| `examples.md` | 5 fully annotated example tables with best-practice explanations |
| `figma-canvas.md` | Script for rendering a finished table onto the Figma canvas, plus the auto-layout constraints that trip it up |
| `figma-annotations.md` | Script for marking up the design itself with numbered badges tied to each row's Order number |

## Usage

The skill activates automatically when you ask for accessibility handoff work. Try:

- *"Generate a VoiceOver handoff table for this screen"* (with a Figma URL or description)
- *"Create accessibility specs for both iOS and Android"*
- *"Generate a Web handoff table for this page"*
- *"This screen has a native paywall over a WebView — document it"*
- *"What should the screen reader read for this paywall design?"*
- *"Draw the TalkBack spec on the canvas next to the frame"* (needs the Figma MCP server)
- *"Add numbered markers on the design so I can see which row is which element"*

You get a TSV code block to paste into Figma or a spreadsheet — or, if you ask for it
on the canvas, a native Figma table placed beside the frame, with matching numbered
badges drawn directly on top of the design.

Paste a **link to the selection** rather than a plain file URL: right-click the frame
in Figma and choose **Copy link to selection**, so the URL carries a `node-id`.

## Best Practices Covered

The examples file explains key accessibility patterns with rationale:

- **Icons without text** need explicit labels (e.g., "X" icon → "Close")
- **Section headings** should be marked as headers for quick navigation
- **Strikethrough prices** need a combined label ("$30, discounted to $4/month")
- **External links** should warn users they'll leave the app
- **Accordions** need state ("expanded"/"collapsed") and a hint about what they control
- **Carousels** use the Adjustable trait with item count and position
- **Text with embedded links** needs rotor access guidance
- **Decorative elements** (dividers, background shapes) should be hidden from screen readers
- **Navigation chrome** is announced first, even when it sits at the bottom of the screen
- **Terms and consent copy** precede any CTA they govern, regardless of visual position
- **Occluded content** (behind a paywall or overlay) is hidden but keeps its real text
- **Hybrid screens** get one combined table with a continuous Order sequence, each row marked `Native` or `Web` — the layer decides the API and the owning team
- **Web reading order** is DOM order, which can silently diverge from visual order through ordinary CSS (`flex`/`grid` order, `position`) — the biggest web-specific risk with no native equivalent
- **Custom ARIA widgets** (a styled `div role="switch"`) never auto-announce state — only genuine native HTML controls do
- **Annotated Order numbers** on the design surface priority-over-position ordering as a visible fact — a terms block's badge can sit below a CTA's while carrying a lower number

## Contributing

Contributions are welcome! If you have additional examples, platform support, or improvements:

1. Fork the repo
2. Add your changes
3. Submit a pull request

## License

MIT
