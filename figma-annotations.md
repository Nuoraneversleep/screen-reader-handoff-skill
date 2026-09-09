# Annotating the Design with Order Numbers

A rendered table sitting next to a frame still makes the reader match rows to shapes
by eye. This script closes that gap: it draws a numbered badge and a dashed outline
directly on top of the frame, one per row, using the same number as that row's
**Order** column. A reviewer can then point at any element on screen and read off
which row documents it — and see priority-over-position ordering (nav chrome first,
terms before the CTA) as a fact about the canvas instead of a claim in the Notes
column, since a term's badge can sit visually below a CTA's while still carrying a
lower number.

Load the `figma-use` skill first — it is a mandatory prerequisite for `use_figma`.

## When to use this

Pair it with [figma-canvas.md](figma-canvas.md) whenever the user wants the spec "in
the file" — render the table beside the frame, then run this script on the frame
itself, so the two reference each other by number. It also works standalone if the
user only wants markers on the design without a rendered table.

Skip a row entirely — no badge, no outline — for anything with a blank **Order**:
`Merged into parent` rows (no separate focus stop) and, per the platform schemas,
`Hidden: Yes` rows (not a swipe stop either). Optionally give `Hidden: Yes` rows a
grey dashed outline with **no number**, so a reviewer can see the occluded element is
tracked without implying it's part of the swipe sequence.

## What you need before running the script

You already have this from Step 2 (reading the design) — the same node walk that
built the table. For each numbered row, keep:

- The **Order** number (matches the table).
- The element's bounding box **relative to the frame you're annotating** — the same
  coordinate space used for the occlusion check: child coordinates are relative to
  their immediate parent, so sum offsets down to the frame, not the page.
- Whether the row is `Hidden: Yes` (for the muted-outline treatment).

If a row merges several layers (`Parent of N (combined)`), use the union bounding box
of the merged group, not any single child.

## The script

```js
// One entry per numbered row, in any order. x/y/width/height are relative to
// `target` — the same frame you built the table from.
const ANNOTATIONS = [
  // { order: '1', x: 125, y: 80, width: 212, height: 55 },
  // { order: '4', x: 56, y: 704, width: 359, height: 140, hidden: true },
];

const target = figma.currentPage.selection[0];
if (!target) throw new Error('Select the frame you documented, then run this script.');

const rgb = (hex) => {
  const n = parseInt(hex.slice(1), 16);
  return { r: ((n >> 16) & 255) / 255, g: ((n >> 8) & 255) / 255, b: (n & 255) / 255 };
};

const BADGE_W = 32, BADGE_H = 28;
const BLUE = '#2F6BFF', NAVY = '#2B3A67', GREY = '#9AA5B1';
const FONT = { family: 'Inter', style: 'Bold' };
await figma.loadFontAsync(FONT);

const overlay = figma.createFrame();
overlay.name = 'Annotations — ' + target.name;
overlay.x = target.x;
overlay.y = target.y;
overlay.resize(target.width, target.height);
overlay.clipsContent = false;   // badges are allowed to sit outside the frame's edge
overlay.fills = [];

// Same parent as target, so target.x/y and ANNOTATIONS coordinates share one origin.
target.parent.appendChild(overlay);

ANNOTATIONS.forEach((a) => {
  const outlineColor = a.hidden ? GREY : BLUE;

  const box = figma.createRectangle();
  box.name = 'Highlight ' + a.order;
  box.x = a.x; box.y = a.y;
  box.resize(a.width, a.height);
  box.fills = [];
  box.strokes = [{ type: 'SOLID', color: rgb(outlineColor) }];
  box.strokeWeight = 1.5;
  box.dashPattern = [4, 3];
  box.cornerRadius = 4;
  overlay.appendChild(box);

  if (a.hidden) return;   // grey outline only — no badge, matches its blank Order

  const badge = figma.createFrame();
  badge.name = 'Badge ' + a.order;
  badge.x = a.x - BADGE_W;   // flush against the left edge of its element
  badge.y = a.y;             // top-aligned with the element, not vertically centered
  badge.resize(BADGE_W, BADGE_H);
  badge.cornerRadius = 6;
  badge.fills = [{ type: 'SOLID', color: rgb(NAVY) }];
  overlay.appendChild(badge);

  const label = figma.createText();
  label.fontName = FONT;
  label.characters = String(a.order);
  label.fontSize = 13;
  label.fills = [{ type: 'SOLID', color: rgb('#FFFFFF') }];
  label.textAlignHorizontal = 'CENTER';
  label.textAlignVertical = 'CENTER';
  badge.appendChild(label);
  label.textAutoResize = 'NONE';   // required before a manual resize
  label.resize(BADGE_W, BADGE_H);
  label.x = 0; label.y = 0;
});

figma.currentPage.selection = [overlay];
figma.viewport.scrollAndZoomIntoView([overlay, target]);
'Annotated ' + ANNOTATIONS.filter(a => !a.hidden).length + ' numbered rows onto ' + target.name;
```

## Notes

- **Badges sit flush left of their element, top-aligned** — `badge.x = element.x -
  BADGE_W` with no gap, `badge.y = element.y`. For elements near the frame's own left
  edge this pushes the badge into negative x, spilling outside the frame into empty
  canvas space. That's intentional and matches the reference pattern; `clipsContent =
  false` on the overlay is what allows it.
- **Top-align, don't vertically center, for tall elements.** A badge centered against
  a 140px-tall terms paragraph drifts far from where the reader's eye lands first.
  Aligning to the top edge keeps the number next to the start of the content.
- **The overlay is a sibling of `target`, not a child.** Appending into `target`'s own
  parent — rather than the page — keeps the coordinate math correct even when `target`
  sits inside a section or another frame; page-relative math would be wrong in that
  case, since `x`/`y` on a node are relative to its immediate parent.
- **Re-running the script leaves the old overlay behind.** Delete the previous
  `Annotations — <name>` frame first if the row set changed, or you'll get two sets of
  badges stacked on the same frame.
- **Keep numbers in sync with the rendered table.** If you regenerate the table after
  edits, regenerate the annotations from the same `ANNOTATIONS` list so the two never
  drift apart.
- **Verify with `node.screenshot()`, not the `get_screenshot` tool.** Immediately after
  writing the overlay, `get_screenshot` on `target` can return a stale cached render —
  same bytes as before the badges existed. `await target.screenshot({ contentsOnly:
  false })` inside `use_figma` reads the live canvas and is the reliable check.
