# Rendering the Table onto the Figma Canvas

When the user wants the spec in the file rather than as pasted TSV, build it with
`use_figma`. Load the `figma-use` skill first — it is a mandatory prerequisite for
that tool.

The script below renders one table as a native auto-layout frame: a title block, a
header row, and one row per element. Every cell is real text, so reviewers can correct
a label in place.

## Read this first — three constraints that will bite you

Auto-layout sizing has rules that produce confusing errors rather than obvious ones.
All three cost real debugging time:

1. **`layoutSizing*` requires a parent.** You cannot set `layoutSizingHorizontal` on a
   node before `appendChild` puts it inside an auto-layout frame. Create → append →
   *then* size. Setting it at creation time throws.
2. **`FILL` requires the parent to be `FIXED` on that axis.** A child cannot fill an
   axis its parent hugs. To make cells share a uniform height in a row, append them
   all, then set `row.counterAxisSizingMode = 'FIXED'` (which locks the hugged
   height), and only then set each cell to `FILL`.
3. **Load fonts before writing text.** `figma.loadFontAsync` for every family/style
   you use, before setting `.characters` or `.fontName`. Note Inter's style strings
   have spaces: `"Semi Bold"`, not `"SemiBold"`.

A fourth, cosmetic one: `figma.createText()` starts at a fixed 100px width, so text
appears clipped unless you set sizing after appending (`FILL` inside a fixed-width
cell, `HUG` for a title).

## The script

Pass the finished rows in as `ROWS` — an array of 12-string arrays matching the
platform's column order. Set `TITLE`, `SUBTITLE`, and `COLS` per platform.

```js
const COLS = ['Order','Component','Layer','Trait','Label','Value','Grouping','Hidden','Actions','Hint','Example','Notes on Documentation'];
const WIDTHS = [50, 140, 60, 100, 190, 110, 130, 60, 150, 150, 280, 240];
const TITLE = 'iOS VoiceOver — <screen name>';
const SUBTITLE = 'Label → Value → Trait → Hint · review before handoff';
const ROWS = [ /* ['1','Close button','Native','button', ...], ... */ ];

const REG = { family: 'Inter', style: 'Regular' };
const BOLD = { family: 'Inter', style: 'Semi Bold' };
await figma.loadFontAsync(REG);
await figma.loadFontAsync(BOLD);

const rgb = (hex) => {
  const n = parseInt(hex.slice(1), 16);
  return { r: ((n >> 16) & 255) / 255, g: ((n >> 8) & 255) / 255, b: (n & 255) / 255 };
};

function text(chars, opts) {
  const t = figma.createText();
  t.fontName = opts.bold ? BOLD : REG;          // font is loaded, so this is safe
  t.fontSize = opts.size || 11;
  t.characters = chars == null ? '' : String(chars);
  t.lineHeight = { value: 140, unit: 'PERCENT' };
  t.fills = [{ type: 'SOLID', color: rgb(opts.color || '#1A1A1A') }];
  return t;                                      // sizing happens after append
}

function cell(value, width, opts) {
  const f = figma.createFrame();
  f.name = 'Cell';
  f.layoutMode = 'VERTICAL';
  f.primaryAxisSizingMode = 'AUTO';              // height hugs the wrapped text
  f.counterAxisSizingMode = 'FIXED';             // width is the column
  f.resize(width, f.height);
  f.paddingLeft = 10; f.paddingRight = 10; f.paddingTop = 8; f.paddingBottom = 8;
  f.fills = opts.bg ? [{ type: 'SOLID', color: rgb(opts.bg) }] : [];
  f.strokes = [{ type: 'SOLID', color: rgb('#E3E3E3') }];
  f.strokeWeight = 1;
  f.strokeAlign = 'INSIDE';

  const t = text(value, opts);
  f.appendChild(t);                              // append BEFORE sizing
  t.layoutSizingHorizontal = 'FILL';             // legal: parent is FIXED width
  t.layoutSizingVertical = 'HUG';
  return f;
}

function row(values, opts) {
  const r = figma.createFrame();
  r.name = opts.header ? 'Header row' : 'Row';
  r.layoutMode = 'HORIZONTAL';
  r.primaryAxisSizingMode = 'AUTO';
  r.counterAxisSizingMode = 'AUTO';               // hug height while filling
  r.counterAxisAlignItems = 'MIN';
  r.itemSpacing = 0;
  r.fills = [];

  values.forEach((v, i) => r.appendChild(cell(v, WIDTHS[i] || 150, {
    bold: !!opts.header,
    bg: opts.header ? '#F2F2F2' : (opts.muted ? '#FAFAFA' : '#FFFFFF'),
    color: opts.muted ? '#6B6B6B' : '#1A1A1A'
  })));

  // Lock the hugged height, THEN let cells fill it, so borders line up.
  r.counterAxisSizingMode = 'FIXED';
  r.children.forEach((c) => { c.layoutSizingVertical = 'FILL'; });
  return r;
}

const table = figma.createFrame();
table.name = TITLE;
table.layoutMode = 'VERTICAL';
table.primaryAxisSizingMode = 'AUTO';
table.counterAxisSizingMode = 'AUTO';
table.itemSpacing = 0;
table.paddingTop = 24; table.paddingBottom = 24;
table.paddingLeft = 24; table.paddingRight = 24;
table.fills = [{ type: 'SOLID', color: rgb('#FFFFFF') }];
table.cornerRadius = 8;

const head = figma.createFrame();
head.name = 'Title';
head.layoutMode = 'VERTICAL';
head.primaryAxisSizingMode = 'AUTO';
head.counterAxisSizingMode = 'AUTO';
head.itemSpacing = 4;
head.paddingBottom = 16;
head.fills = [];
table.appendChild(head);                          // stays HUG — table hugs its width

const h1 = text(TITLE, { bold: true, size: 18 });
head.appendChild(h1);
h1.layoutSizingHorizontal = 'HUG';
const h2 = text(SUBTITLE, { size: 11, color: '#6B6B6B' });
head.appendChild(h2);
h2.layoutSizingHorizontal = 'HUG';

table.appendChild(row(COLS, { header: true }));
// A blank Order means a Merged-into-parent or Hidden row — grey it so the reader
// can see at a glance which rows are not focus stops.
ROWS.forEach((r) => table.appendChild(row(r, { muted: !String(r[0]).trim() })));

// Park it to the right of the frame being documented.
const target = figma.currentPage.selection[0];
if (target) {
  table.x = target.x + target.width + 160;
  table.y = target.y;
}
figma.currentPage.selection = [table];
figma.viewport.scrollAndZoomIntoView([table]);
'Rendered ' + ROWS.length + ' rows';
```

## Notes

- **Both platforms:** run the script twice, changing `TITLE`, `SUBTITLE`, `COLS` and
  `ROWS`. Offset the second table (`table.y = target.y + 900`, say) so they don't
  overlap, or wrap both in a vertical auto-layout frame.
- **TalkBack columns:** `['Order','Component','Layer','Element Type','Description','State','Grouping','Hidden','Action','Announce on change','TalkBack example','Notes']`,
  and a subtitle of `Description → State → Element Type → Action hint`.
- **Empty Android State cells stay genuinely empty** — for Switch, Checkbox, Radio,
  Toggle, selectable cells and disabled controls, pass `''`, not `'none'`. Writing
  anything there risks a double announcement.
- **Keep cells on one line.** Join multi-part notes with ` • `; a literal newline
  inside a cell breaks the row rhythm.
- **Long tables are slow to render.** Past ~40 rows, mention it will take a moment
  rather than looking hung.
- **Pair with the design itself.** A table alone still leaves the reader to match rows
  to shapes by eye. See [figma-annotations.md](figma-annotations.md) for a script that
  draws the same Order numbers as badges directly on the frame.
