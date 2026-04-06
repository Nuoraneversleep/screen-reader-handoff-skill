# Examples & Best Practices

Real-world examples with annotations explaining **why** each decision was made. Use these as references when generating new tables.

---

## Example 1: Subscription Paywall with Accordion (iOS VoiceOver)

A paywall screen with a hero title, an "All Access" card containing an expandable "Learn more" accordion, two purchase options (NYT and Apple), and terms & conditions.

```
Order	Component	Trait	Label	Actions	Hint	Value	Example	Notes on Documentation
1	Close button	button	Close	Double tap to dismiss	none	none	Close, button	• The X icon has no visible text, so an explicit label is required
2	Page header	header	none	none	none	none	Subscribe to gain unlimited access to all of The Times, heading	• Marked as header so users can jump directly here via the rotor
3	Plan title	none	none	none	none	none	All Access	•
4	Plan description	none	none	none	none	none	News, plus Games, Cooking, Audio, Wirecutter and The Athletic.	•
5	Learn more dropdown	button	Learn more	Double tap to expand or collapse	Shows or hides subscription benefit details.	collapsed	Learn more, collapsed, button. Shows or hides subscription benefit details.	• Value changes between "expanded" and "collapsed" • Hint tells the user what the accordion controls
6	Purchase channel label	header	none	none	none	none	Buy through The New York Times, heading	• Marked as header — lets users jump between purchase options via rotor
7	NYT purchase button	button	$30, discounted to $4 per month	Double tap opens external checkout	Opens an external website.	none	$30, discounted to $4 per month, button. Opens an external website.	• Custom label combines strikethrough price and sale price into one clear announcement • Hint warns the user they'll leave the app
8	NYT billing details	none	none	none	none	none	Introductory offer: billed as $4 every four weeks for your first 6 months, then $30 every four weeks thereafter.	•
9	Purchase channel label	header	none	none	none	none	Buy through Apple, heading	•
10	Apple purchase button	button	$35, discounted to $5 per month	Double tap to subscribe	none	none	$35, discounted to $5 per month, button	• No external link hint needed — this stays in the app
11	Apple billing details	none	none	none	none	none	Introductory offer: billed as $5 every calendar month for your first 6 months, then $35 every calendar month thereafter.	•
12	Continue link	button	Continue without subscribing	Double tap to dismiss	none	none	Continue without subscribing, button	•
13	Terms & Conditions	none	[Terms & Conditions copy]	Rotate two fingers opens rotor	Use the rotor to access links	none	[T&C copy with [cancel] link callout]	• The cancel link inside the text block should be accessible via rotor • Hint teaches users how to find the link
```

### Best Practices Highlighted

**Icons without text need explicit labels** (row 1). The close button is just an "X" icon — a sighted user understands it, but VoiceOver needs the word "Close."

**Use headers for section jumps** (rows 2, 6, 9). When a screen has multiple sections, marking titles as headers lets users skip directly to them via the rotor's "Headings" mode, instead of swiping through every element.

**Combine visual price formatting into a spoken label** (rows 7, 10). The design shows "$30" with a strikethrough and "$4/month" — visually clear, but VoiceOver would read them as separate unrelated numbers. The label "$30, discounted to $4 per month" tells the full story in one sentence.

**Warn before leaving the app** (row 7 vs. 10). The NYT button opens Safari, so it has a hint: "Opens an external website." The Apple button stays in-app, so no hint is needed. Users who are blind rely heavily on these warnings.

**Accordion state in the Value column** (row 5). "expanded" or "collapsed" tells the user what mode the accordion is in *before* they tap it. The hint explains what the accordion controls.

**Links inside text blocks** (row 13). When text contains a tappable link (like "cancel"), you can't just double-tap to activate it — the user needs to use the rotor. The hint teaches them how.

---

## Example 2: Free Trial Onboarding Screen (iOS VoiceOver)

A post-account-creation screen with a confirmation banner, a free trial timeline animation, pricing breakdown, value proposition, terms, and two CTAs.

```
Order	Component	Trait	Label	Actions	Hint	Value	Example	Notes on Documentation
1	Confirmation	none	none	none	none	none	Account created as nytgrowth@gmail.com	•
2	Page header	header	none	none	none	none	Try 7 days free. Your account includes a free trial to explore The Times, heading	•
3	Urgency Badge	none	none	none	none	none	Limited time offer	•
4	Timeline animation	image	Free trial offer timeline	none	none	none	Free trial offer timeline, image	• The animation shows a visual timeline — describe it as a single image rather than individual frames
5	Now	none	none	none	none	none	Now. Limited access to The Times.	•
6	Today	none	none	none	none	none	Today. Unlock free unlimited access for 7 days.	•
7	After 7 days	none	After 7 days. $4/month for your first 6 months, discounted from $25. [offer subtext]	none	none	none	After 7 days. $4/month for your first 6 months, discounted from $25. After free trial, billed as $4 every four weeks for your first six months, then $25 thereafter.	• Custom label includes the offer subtext that appears on hover/expansion
8	Value proposition	none	none	none	none	none	Your free trial includes: Unlimited access to news, Games, Cooking, Audio, Wirecutter and The Athletic.	•
9	Terms & Conditions	none	[Terms & Conditions copy]	Rotate two fingers opens rotor	Use the rotor to access links	none	[T&C copy: Your subscription will continue until you [cancel], link, use the rotor to access links.]	• Call out the cancel link in the text
10	CTA button	button	Start 7-day free trial in browser	Double tap opens external checkout page	Opens an external website.	none	Start 7-day free trial in browser, button. Opens an external website.	• Hint warns user they'll leave the app
11	Body text	none	none	none	none	none	This purchase will be made through The New York Times.	•
12	CTA button	button	Continue without trial	Double tap to proceed	none	none	Continue without trial, button	•
```

### Best Practices Highlighted

**Animations as images** (row 4). An animated timeline is meaningful content, but VoiceOver can't describe each frame. Treat it as a single image with a descriptive label.

**Collapsing detailed pricing into one label** (row 7). The screen shows "After 7 days" with pricing details that may be partially hidden. The label combines everything into one complete announcement.

**Primary vs. secondary CTA distinction** (rows 10, 12). The primary CTA has a hint about leaving the app; the secondary CTA doesn't. This matches the visual hierarchy — primary actions get more context.

---

## Example 3: Sale Screen with Carousel (iOS VoiceOver)

A sale screen with a value proposition carousel that users can swipe through.

```
Order	Component	Trait	Label	Actions	Hint	Value	Example	Notes on Documentation
1	Confirmation	none	none	none	none	none	Logged in as nytgrowth@gmail.com	•
2	Page header	header	none	none	none	none	Save on unlimited access with our best offer on The Times, heading	•
3	Urgency Badge	none	none	none	none	none	Sale offer	•
4	Price information	none	$4/month for your first 12 months, discounted from $25. [offer subtext]	none	none	none	$4/month for your first 12 months, discounted from $25. Billed as $4 every 4 weeks for your first twelve months. Cancel or pause anytime.	•
5	Value prop carousel	Adjustable	Subscriber value carousel, containing 6 items	Swipe up to go to next card, swipe down to go to previous card	Swipe up or down with one finger to advance carousel	1 of 6, essential reporting	Subscriber value carousel, containing 6 items, adjustable. 1 of 6, essential reporting. Swipe up or down with one finger to advance carousel.	• Read the focused card's header when user swipes to next card • The Value column updates with each swipe: "2 of 6, daily puzzles" etc.
6	Value proposition	none	none	none	none	none	Your subscription includes: Unlimited access to news, Games, Cooking, Audio, Wirecutter and The Athletic.	•
7	Terms & Conditions	none	[Terms & Conditions copy]	Rotate two fingers opens rotor	Use the rotor to access links	none	[T&C copy with [cancel] link callout]	• Call out the cancel link
8	CTA button	button	Subscribe now	Double tap opens checkout page	Opens an external website.	none	Subscribe now, button. Opens an external website.	•
9	Body text	none	none	none	none	none	This purchase will be made through The New York Times.	•
10	CTA button	button	Continue without subscribing	Double tap to dismiss	none	none	Continue without subscribing, button	•
```

### Best Practices Highlighted

**Carousels use the Adjustable trait** (row 5). This is the only way screen reader users can navigate carousel content — swiping left/right moves to the *next element*, but swiping up/down on an Adjustable changes its value (next/previous card).

**Carousel labels include item count** (row 5). "containing 6 items" tells users how much content is in the carousel before they start navigating.

**Value updates with each swipe** (row 5). The note explains that the Value column is dynamic — "1 of 6, essential reporting" becomes "2 of 6, daily puzzles" as the user swipes.

---

## Example 4: Same Screen, Android TalkBack

The same subscription paywall from Example 1, but for Android TalkBack.

```
Order	Component	Role	Content Description	Actions	Action Label	State Description	Example	Notes on Implementation
1	Close button	button	Close	Double tap to activate	Dismiss	none	Close, button. Double tap to activate.	• The X icon needs an explicit content description
2	Page header	heading	none	none	none	none	Subscribe to gain unlimited access to all of The Times, heading	•
3	Plan title	none	none	none	none	none	All Access	•
4	Plan description	none	none	none	none	none	News, plus Games, Cooking, Audio, Wirecutter and The Athletic.	•
5	Learn more dropdown	button	Learn more	Double tap to expand	Expand section	Collapsed	Learn more, Collapsed, button. Double tap to expand.	• State changes to "Expanded" when open • Action label changes to "Collapse section"
6	Purchase channel label	heading	none	none	none	none	Buy through The New York Times, heading	•
7	NYT purchase button	button	$30, discounted to $4 per month	Double tap to activate. Opens external browser.	Open in browser	none	$30, discounted to $4 per month, button. Double tap to activate. Opens external browser.	• Action label "Open in browser" appears in TalkBack menu
8	NYT billing details	none	none	none	none	none	Introductory offer: billed as $4 every four weeks for your first 6 months, then $30 every four weeks thereafter.	•
9	Purchase channel label	heading	none	none	none	none	Buy through Apple, heading	•
10	Apple purchase button	button	$35, discounted to $5 per month	Double tap to activate	none	none	$35, discounted to $5 per month, button. Double tap to activate.	•
11	Apple billing details	none	none	none	none	none	Introductory offer: billed as $5 every calendar month for your first 6 months, then $35 every calendar month thereafter.	•
12	Continue link	button	Continue without subscribing	Double tap to activate	Dismiss	none	Continue without subscribing, button. Double tap to activate.	•
13	Terms & Conditions	none	[Terms & Conditions copy]	none	none	none	[T&C copy]	• The cancel link should be a ClickableSpan • Group the text block as a single focusable element
```

### Key Differences from iOS Version

**TalkBack announces actions explicitly** — iOS says "button" and users *know* to double-tap, but TalkBack says "Double tap to activate" out loud.

**Action Labels appear in the context menu** — iOS uses Hints (spoken automatically), while TalkBack uses Action Labels (shown when the user opens the local context menu). This is why row 7 has `Open in browser` as an Action Label instead of a Hint.

**State Description uses capitalization** — iOS convention is lowercase ("collapsed"), TalkBack convention is capitalized ("Collapsed").

---

## Prompt Template

Use this when requesting a handoff table:

> Generate a **[VoiceOver / TalkBack / both]** accessibility handoff table for the **[Screen Name]** screen.
>
> **Screen description:** [List all visible elements top-to-bottom, or provide a Figma URL]
>
> **Context:** [App name, platform, any special interactions like carousels or accordions]
>
> Follow the screen-reader-handoff skill format. Output as TSV I can paste into Figma.
