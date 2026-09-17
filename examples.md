# Examples & Best Practices

Real-world examples with annotations explaining **why** each decision was made. Use these as references when generating new tables.

---

## Example 1: Subscription Paywall with Accordion (iOS VoiceOver)

A paywall screen with a hero title, an "All Access" card containing an expandable "Learn more" accordion, two purchase options (NYT and Apple), and terms & conditions.

```
Order	Component	Layer	Trait	Label	Value	Grouping	Hidden	Actions	Hint	Example	Notes on Documentation
1	Close button	Native	button	Close	none	Standalone	No	Double tap to dismiss	none	Close, button	• The X icon has no visible text, so an explicit label is required
2	Page header	Native	header	none	none	Standalone	No	none	none	Subscribe to gain unlimited access to all of The Times, heading	• Marked as header so users can jump directly here via the rotor
3	Plan title	Native	none	none	none	Standalone	No	none	none	All Access	•
4	Plan description	Native	none	none	none	Standalone	No	none	none	News, plus Games, Cooking, Audio, Wirecutter and The Athletic.	•
5	Learn more dropdown	Native	button	Learn more	collapsed	Standalone	No	Double tap to expand or collapse	Shows or hides subscription benefit details.	Learn more, collapsed, button. Shows or hides subscription benefit details.	• Value changes between "expanded" and "collapsed" • Hint tells the user what the accordion controls
6	Purchase channel label	Native	header	none	none	Standalone	No	none	none	Buy through The New York Times, heading	• Marked as header — lets users jump between purchase options via rotor
7	NYT purchase button	Native	button	$30, discounted to $4 per month	none	Standalone	No	Double tap opens external checkout	Opens an external website.	$30, discounted to $4 per month, button. Opens an external website.	• Custom label combines strikethrough price and sale price into one clear announcement • Hint warns the user they'll leave the app
8	NYT billing details	Native	none	none	none	Standalone	No	none	none	Introductory offer: billed as $4 every four weeks for your first 6 months, then $30 every four weeks thereafter.	•
9	Purchase channel label	Native	header	none	none	Standalone	No	none	none	Buy through Apple, heading	•
10	Apple purchase button	Native	button	$35, discounted to $5 per month	none	Standalone	No	Double tap to subscribe	none	$35, discounted to $5 per month, button	• No external link hint needed — this stays in the app
11	Apple billing details	Native	none	none	none	Standalone	No	none	none	Introductory offer: billed as $5 every calendar month for your first 6 months, then $35 every calendar month thereafter.	•
12	Terms & Conditions	Native	none	[Terms & Conditions copy]	none	Standalone	No	Rotate two fingers opens rotor	Use the rotor to access links	[T&C copy with [cancel] link callout]	• The cancel link inside the text block should be accessible via rotor • Hint teaches users how to find the link
13	Continue link	Native	button	Continue without subscribing	none	Standalone	No	Double tap to dismiss	none	Continue without subscribing, button	•
```

### Best Practices Highlighted

**Icons without text need explicit labels** (row 1). The close button is just an "X" icon — a sighted user understands it, but VoiceOver needs the word "Close."

**Use headers for section jumps** (rows 2, 7, 10). When a screen has multiple sections, marking titles as headers lets users skip directly to them via the rotor's "Headings" mode, instead of swiping through every element.

**Combine visual price formatting into a spoken label** (rows 8, 11). The design shows "$30" with a strikethrough and "$4/month" — visually clear, but VoiceOver would read them as separate unrelated numbers. The label "$30, discounted to $4 per month" tells the full story in one sentence.

**Warn before leaving the app** (row 8 vs. 11). The NYT button opens Safari, so it has a hint: "Opens an external website." The Apple button stays in-app, so no hint is needed. Users who are blind rely heavily on these warnings.

**Accordion state in the Value column** (row 5). "expanded" or "collapsed" tells the user what mode the accordion is in *before* they tap it. The hint explains what the accordion controls.

**Links inside text blocks** (row 12). When text contains a tappable link (like "cancel"), you can't just double-tap to activate it — the user needs to use the rotor. The hint teaches them how.

**Terms follow their natural visual position** (row 12). On screen, the T&C copy sits at the bottom, below both purchase buttons — the table follows that same order rather than reordering it ahead of the CTAs. A VoiceOver user reaches it in the sequence it was actually built, the same order a sighted user encounters it scanning top to bottom.

---

## Example 2: Free Trial Onboarding Screen (iOS VoiceOver)

A post-account-creation screen with a confirmation banner, a free trial timeline animation, pricing breakdown, value proposition, terms, and two CTAs.

```
Order	Component	Layer	Trait	Label	Value	Grouping	Hidden	Actions	Hint	Example	Notes on Documentation
1	Confirmation	Native	none	none	none	Standalone	No	none	none	Account created as nytgrowth@gmail.com	•
2	Page header	Native	header	none	none	Standalone	No	none	none	Try 7 days free. Your account includes a free trial to explore The Times, heading	•
3	Urgency Badge	Native	none	none	none	Standalone	No	none	none	Limited time offer	•
4	Timeline animation	Native	image	Free trial offer timeline	none	Standalone	No	none	none	Free trial offer timeline, image	• The animation shows a visual timeline — describe it as a single image rather than individual frames
5	Now	Native	none	none	none	Standalone	No	none	none	Now. Limited access to The Times.	•
6	Today	Native	none	none	none	Standalone	No	none	none	Today. Unlock free unlimited access for 7 days.	•
7	After 7 days	Native	none	After 7 days. $4/month for your first 6 months, discounted from $25. [offer subtext]	none	Standalone	No	none	none	After 7 days. $4/month for your first 6 months, discounted from $25. After free trial, billed as $4 every four weeks for your first six months, then $25 thereafter.	• Custom label includes the offer subtext that appears on hover/expansion
8	Value proposition	Native	none	none	none	Standalone	No	none	none	Your free trial includes: Unlimited access to news, Games, Cooking, Audio, Wirecutter and The Athletic.	•
9	Terms & Conditions	Native	none	[Terms & Conditions copy]	none	Standalone	No	Rotate two fingers opens rotor	Use the rotor to access links	[T&C copy: Your subscription will continue until you [cancel], link, use the rotor to access links.]	• Call out the cancel link in the text
10	CTA button	Native	button	Start 7-day free trial in browser	none	Standalone	No	Double tap opens external checkout page	Opens an external website.	Start 7-day free trial in browser, button. Opens an external website.	• Hint warns user they'll leave the app
11	Body text	Native	none	none	none	Standalone	No	none	none	This purchase will be made through The New York Times.	•
12	CTA button	Native	button	Continue without trial	none	Standalone	No	Double tap to proceed	none	Continue without trial, button	•
```

### Best Practices Highlighted

**Animations as images** (row 4). An animated timeline is meaningful content, but VoiceOver can't describe each frame. Treat it as a single image with a descriptive label.

**Collapsing detailed pricing into one label** (row 7). The screen shows "After 7 days" with pricing details that may be partially hidden. The label combines everything into one complete announcement.

**Primary vs. secondary CTA distinction** (rows 10, 12). The primary CTA has a hint about leaving the app; the secondary CTA doesn't. This matches the visual hierarchy — primary actions get more context.

---

## Example 3: Sale Screen with Carousel (iOS VoiceOver)

A sale screen with a value proposition carousel that users can swipe through.

```
Order	Component	Layer	Trait	Label	Value	Grouping	Hidden	Actions	Hint	Example	Notes on Documentation
1	Confirmation	Native	none	none	none	Standalone	No	none	none	Logged in as nytgrowth@gmail.com	•
2	Page header	Native	header	none	none	Standalone	No	none	none	Save on unlimited access with our best offer on The Times, heading	•
3	Urgency Badge	Native	none	none	none	Standalone	No	none	none	Sale offer	•
4	Price information	Native	none	$4/month for your first 12 months, discounted from $25. [offer subtext]	none	Standalone	No	none	none	$4/month for your first 12 months, discounted from $25. Billed as $4 every 4 weeks for your first twelve months. Cancel or pause anytime.	•
5	Value prop carousel	Native	Adjustable	Subscriber value carousel, containing 6 items	1 of 6, essential reporting	Standalone	No	Swipe up to go to next card, swipe down to go to previous card	Swipe up or down with one finger to advance carousel	Subscriber value carousel, containing 6 items, adjustable. 1 of 6, essential reporting. Swipe up or down with one finger to advance carousel.	• Read the focused card's header when user swipes to next card • The Value column updates with each swipe: "2 of 6, daily puzzles" etc.
6	Value proposition	Native	none	none	none	Standalone	No	none	none	Your subscription includes: Unlimited access to news, Games, Cooking, Audio, Wirecutter and The Athletic.	•
7	CTA button	Native	button	Subscribe now	none	Standalone	No	Double tap opens checkout page	Opens an external website.	Subscribe now, button. Opens an external website.	•
8	Body text	Native	none	none	none	Standalone	No	none	none	This purchase will be made through The New York Times.	•
9	CTA button	Native	button	Continue without subscribing	none	Standalone	No	Double tap to dismiss	none	Continue without subscribing, button	•
10	Terms & Conditions	Native	none	[Terms & Conditions copy]	none	Standalone	No	Rotate two fingers opens rotor	Use the rotor to access links	[T&C copy with [cancel] link callout]	• Call out the cancel link
```

### Best Practices Highlighted

**Carousels use the Adjustable trait** (row 5). This is the only way screen reader users can navigate carousel content — swiping left/right moves to the *next element*, but swiping up/down on an Adjustable changes its value (next/previous card).

**Carousel labels include item count** (row 5). "containing 6 items" tells users how much content is in the carousel before they start navigating.

**Value updates with each swipe** (row 5). The note explains that the Value column is dynamic — "1 of 6, essential reporting" becomes "2 of 6, daily puzzles" as the user swipes.

---

## Example 4: Same Screen, Android TalkBack

The same subscription paywall from Example 1, but for Android TalkBack. Rows are listed in announcement order, which follows the screen's visual order — Terms & Conditions is announced where it sits on screen, after the purchase buttons.

```
Order	Component	Layer	Element Type	Description	State	Grouping	Hidden	Action	Announce on change	TalkBack example	Notes
1	Close button	Native	Button	Close	none	Standalone	No	Dismiss	None	Close, button. Double tap to activate.	• On dismiss, return focus to whatever opened the paywall
2	Page header	Native	Heading	none	none	Standalone	No	None	None	Subscribe to gain unlimited access to all of The Times, heading	•
3	Plan title	Native	None	none	none	Standalone	No	None	None	All Access	•
4	Plan description	Native	None	none	none	Standalone	No	None	None	News, plus Games, Cooking, Audio, Wirecutter and The Athletic.	•
5	Learn more dropdown	Native	Button	Learn more	Collapsed	Standalone	No	Expand	None	Learn more, Collapsed, button. Double tap to expand.	• Announce expanded/collapsed state change politely
6	Purchase channel label	Native	Heading	none	none	Standalone	No	None	None	Buy through The New York Times, heading	•
7	NYT purchase button	Native	Button	$30, discounted to $4 per month	none	Standalone	No	Open in browser	None	$30, discounted to $4 per month, button. Double tap to activate. Opens external browser.	• Warn the user before leaving the app
8	NYT billing details	Native	None	none	none	Standalone	No	None	None	Introductory offer: billed as $4 every four weeks for your first 6 months, then $30 every four weeks thereafter.	•
9	Purchase channel label	Native	Heading	none	none	Standalone	No	None	None	Buy through Apple, heading	•
10	Apple purchase button	Native	Button	$35, discounted to $5 per month	none	Standalone	No	Subscribe	None	$35, discounted to $5 per month, button. Double tap to activate.	•
11	Apple billing details	Native	None	none	none	Standalone	No	None	None	Introductory offer: billed as $5 every calendar month for your first 6 months, then $35 every calendar month thereafter.	•
12	Terms & Conditions	Native	None	[Terms & Conditions copy with cancel link]	none	Parent of 1	No	None	None	[T&C copy]. Cancel, link.	• Cancel link must be reachable via linear swipe
13	Continue link	Native	Button	Continue without subscribing	none	Standalone	No	Dismiss	None	Continue without subscribing, button. Double tap to activate.	•
```

### Key Differences from iOS Version

**TalkBack announces actions explicitly** — iOS says "button" and users *know* to double-tap, but TalkBack says "Double tap to activate" out loud, so the Action column reads as a verb that matches the on-screen intent ("Expand", "Open in browser", "Dismiss").

**One Action column instead of Actions + Hint + Action Label** — iOS uses `Hint` (auto-spoken) for "Opens an external website." TalkBack has no hint equivalent; that information goes into the Action column ("Open in browser") and engineers wire the first item to `onClickLabel`. Any extras land in the swipe up/down menu.

**State for binary toggles is left blank** — for `Switch`, `Checkbox`, `Radio button`, and selectable cells, TalkBack auto-announces the on/off/checked state. Writing it in State would cause double announcements ("On, on"). Only fill State for *measured* values like `Collapsed`, `Playing, 2:15 of 4:30`, or `3 of 5`.

**State capitalization** — iOS convention is lowercase ("collapsed"), TalkBack convention is capitalized ("Collapsed").

**Grouping is explicit** — Plan title and Plan description are `Merged into parent` because they belong to the All Access card and should be read as one block. Android has no drill-in container, so grouping is binary: merged or standalone.

---

## Example 5: Paywalled Article with Bottom Action Bar (Android TalkBack)

A paywalled article screen: headline and summary preview, hero image, a bottom action
bar (NYT's "charm bracelet"), and a native paywall covering the article body. The story
page is a WebView; the bar and paywall are native. This example exercises four things
the others don't — navigation chrome announced first, content occluded by an overlay, a
hybrid native/web split, and a toggle button.

```
Order	Component	Layer	Element Type	Description	State	Grouping	Hidden	Action	Announce on change	TalkBack example	Notes
1	Charm bracelet / Back	Native	Button	Back	none	Standalone	No	Go back	None	Back, button. Double tap to activate.	• Announced first even though the bar sits at the bottom — it is navigation • Needs traversalIndex + isTraversalGroup; layout order would place it last
2	Charm bracelet / Save	Native	Toggle button	Save		Standalone	No	Save article	None	Save, button, off. Double tap to activate.	• State left blank — TalkBack auto-announces the saved condition
3	Charm bracelet / Share	Native	Button	Share	none	Standalone	No	Share article	None	Share, button. Double tap to activate.	•
4	Charm bracelet / More options	Native	Button	More options	none	Standalone	No	Open more options menu	None	More options, button. Double tap to activate.	• Icon-only overflow button needs an explicit description
5	Headline	Web	Heading	none	none	Standalone	No	None	None	Wonder and Awe at the Natural History Museum’s New Wing. Butterflies, Too., heading 1	• Announces the level because web headings carry h1-h6
6	Summary	Web	None	none	none	Standalone	No	None	None	The stunning $465 million Richard Gilder Center for Science is destined to become a colossal attraction in New York City, our critic writes.	•
7	Media / Image	Web	Image	[Alt text]	none	Standalone	No	None	None	[Alt text], image	• Alt text is an alt attribute owned by the CMS / article page — the app team cannot add it • Visible above the paywall, so it is content the reader is meant to perceive
	Photo credit	Web	None	Peter Fisher for The New York Times	none	Standalone	Yes	None	None	(not spoken — behind the paywall)	• Occluded: sits at y=705, inside the paywall band at 575-787 • Keep the real text — a subscriber sees no paywall and this flips to Hidden: No • Must be hidden in the web layer with aria-hidden; clearAndSetSemantics cannot reach into a WebView
8	Paywall header	Native	Heading	none	none	Standalone	No	None	None	Support independent journalism with a subscription., heading	• No level — native headings have none
9	Paywall CTA button	Native	Button	none	none	Standalone	No	Open subscription options	None	View subscription options, button. Double tap to activate.	• Stays in the app but opens a new page, so no external-browser warning • Move focus to the new screen on navigation; Android will not do it automatically
	Paywall gradient	Native	None	Decorative	none	Standalone	Yes	None	None	(not spoken)	• Carries no meaning, unlike the occluded photo credit
```

### Best Practices Highlighted

**The Layer column splits this screen across two teams** (rows 1-4 and 8-9 `Native`,
rows 5-7 plus the photo credit `Web`). That boundary decides who implements each row.
The image's alt text is an `alt` attribute owned by the CMS or article-rendering team,
and hiding the occluded photo credit has to happen in the web layer with `aria-hidden` —
a native overlay does not clear the WebView's accessibility tree. Note also that the
headline announces as "heading 1" because web headings carry levels, while the native
paywall header just says "heading."

**Navigation chrome is announced first** (rows 1-4). The charm bracelet sits at the
*bottom* of the screen, below the paywall. It is still announced before the headline,
because it is the screen's navigation. Android will not do this by default — layout
order puts it last — so it needs `traversalIndex` plus `isTraversalGroup` on a shared
parent.

**Occluded content is hidden but keeps its text** (photo credit). It sits at y=705,
inside the paywall's 575-787 band, so it is invisible and marked `Hidden: Yes`. The
Description keeps the real credit rather than `Decorative`, because a subscriber sees no
paywall and that same element flips to `Hidden: No`. Content behind an overlay stays in
the accessibility tree unless it is cleared explicitly — this is how paywalled text
leaks to screen reader users.

**Occluded rows take no Order number.** Like `Merged into parent` rows, they are listed
so the decision is documented, but they are not focus stops.

**Toggle button State stays blank** (row 2). Save flips between saved and unsaved, so it
is a `Toggle button` and TalkBack announces the condition itself. Writing anything in
State — including `none` — risks "off, off."

**Visible content above a paywall still needs alt text** (row 7). Only about 30% of the
hero image is covered; the rest is deliberately shown to entice the reader. Hiding it
would leave a blind user with silence where a sighted user sees a large photograph.
Check the geometry before assuming a paywall covers something.

**Switched-off layers are left out of the table.** Four layers in this frame are
`hidden="true"` in Figma — `7 MIN READ`, the paywall `Subheader`, the device nav, and a
`Paywall Topper` reading "Already a subscriber? Log in." They are not on screen, so they
get no rows. The topper is still worth flagging: it may be the only escape route for a
logged-out subscriber, and its absence looks like an oversight rather than a decision.

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
