# Screen Reader Handoff Skill

A [Cursor](https://cursor.com) Agent Skill that helps designers generate screen reader accessibility handoff tables for iOS VoiceOver and Android TalkBack.

No screen reader expertise required — the skill guides you through creating structured specs that tell engineers exactly what a screen reader user should hear on every screen.

## What It Does

- Generates **tab-separated (TSV) tables** you can paste directly into Figma or Google Sheets
- Supports **iOS VoiceOver** and **Android TalkBack** with platform-specific schemas
- Reads designs from **Figma URLs** (via MCP) or manual screen descriptions
- Includes annotated **best-practice examples** explaining common patterns like buttons, accordions, carousels, external links, and more

## Example Output

Each row in the table is one element a screen reader user can focus on:

| Order | Component | Trait | Label | Actions | Hint | Value | Example | Notes |
|-------|-----------|-------|-------|---------|------|-------|---------|-------|
| 1 | Close button | button | Close | Double tap to dismiss | none | none | Close, button | Icon needs explicit label |
| 2 | Page header | header | none | none | none | none | Subscribe to..., heading | Enables rotor navigation |
| 5 | Learn more | button | Learn more | Double tap to expand | Shows or hides details. | collapsed | Learn more, collapsed, button. Shows or hides details. | Value tracks state |

## Installation

### Option A: Personal skill (available across all your projects)

```bash
git clone https://github.com/Nuoraneversleep/screen-reader-handoff-skill.git ~/.cursor/skills/screen-reader-handoff
```

### Option B: Project skill (shared with your team via the repo)

```bash
# From your project root
git clone https://github.com/Nuoraneversleep/screen-reader-handoff-skill.git .cursor/skills/screen-reader-handoff
```

Or copy the files manually into either location. The skill directory must contain `SKILL.md` at its root.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill instructions — workflow, quality checklist, common UI patterns |
| `voiceover-schema.md` | iOS VoiceOver 9-column table schema and column-by-column guidance |
| `talkback-schema.md` | Android TalkBack 9-column table schema and platform differences |
| `examples.md` | 4 fully annotated example tables with best-practice explanations |

## Usage

Once installed, Cursor will automatically activate this skill when you ask for accessibility handoff work. Try prompts like:

- *"Generate a VoiceOver handoff table for this screen"* (with a Figma URL or description)
- *"Create accessibility specs for both iOS and Android"*
- *"What should the screen reader read for this paywall design?"*

The skill produces a TSV code block you can copy and paste directly into Figma or any spreadsheet tool.

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

## Contributing

Contributions are welcome! If you have additional examples, platform support, or improvements:

1. Fork the repo
2. Add your changes
3. Submit a pull request

## License

MIT
