Good interfaces look like someone made decisions. AI slop looks like nobody did: every default accepted, every surface decorated, nothing prioritized. Your job is to make a small number of strong, specific choices and then apply them with discipline.

Before writing any CSS

Answer these in one line each (in your head or a comment). If you can't, you're not ready to style.

What is the one thing the user comes here to do? That element gets the most visual weight. Everything else steps back.
What's the density? A tool used 8 hours a day (tables, editors, dashboards) should be compact. A page read once (landing, onboarding) can breathe. Don't give a data table landing-page padding.
What's the one distinctive decision? Pick ONE: an unusual typeface, a committed color, a strong grid, a monospace accent, an editorial layout. Not all of them. Restraint everywhere else is what makes the one choice land.
Core principles

Hierarchy comes from type and space, not boxes. Before adding a card, border, or background, try solving it with size, weight, color (gray levels), and whitespace. Most "sections" don't need a container.

One accent color, used rarely. Neutrals do 90% of the work. The accent marks the primary action, the current selection, and little else. If everything is colored, nothing is.

Borders over shadows. Use 1px borders in a subtle neutral for structure. Reserve shadows for things that actually float (menus, popovers, modals), and keep them soft and small.

Align to something. Pick a spacing scale (4px base) and a content width, and stick to them. Left-align text by default. Centered layouts are for short hero moments, not whole pages.

Write real copy. Placeholder marketing language is the loudest slop signal. Use specific nouns, real-looking data, and labels a product team would actually ship. "3 invoices overdue" beats "Unlock powerful insights."

Design every state. Empty, loading, error, long text, one item, 500 items. A well-designed empty state is a sign of care; a blank white void is not.

Small details compound. Tabular numbers in tables, aligned decimals, consistent icon size and stroke, visible focus rings, correct hover/active states, truncation with ellipsis, sensible line length (60–75ch for prose).

The slop list: do not do these unless there's a specific reason
Purple→blue or pink→orange gradients, especially on hero backgrounds or text
Gradient text on headings
Glassmorphism / backdrop blur panels for no reason
Emoji as icons or section markers (🚀 ✨ 💡)
"✨ AI-powered" style pill badges above headings
The three-column "icon + bold title + two lines" feature grid as the default page section
Every element in a card with rounded-2xl shadow-lg; cards nested inside cards
Huge border radius on everything (keep it 4–8px for app UI unless the brand demands otherwise)
Hover effects that scale, lift, or glow every card
Dark mode as navy + neon cyan/purple
Low-contrast light gray body text on white (keep body text ≥ 4.5:1)
Generic copy: "Unlock," "Supercharge," "Seamless," "Elevate," "Revolutionize," "at your fingertips"
Stock layout: centered hero → logo strip → 3 features → testimonial carousel → pricing → CTA banner, with no deviation
Five different accent colors for "visual interest"
Decorative blobs, floating orbs, grain-gradient backgrounds as filler
Default tokens

Start here, then make the one distinctive change.

css
:root {
  /* Type — one family is enough; a second only for a clear role (mono for data, serif for editorial) */
  --font-sans: "Inter", system-ui, sans-serif;   /* swap for something with character: Geist, IBM Plex Sans, Söhne-like alternatives, Instrument Sans */
  --font-mono: "JetBrains Mono", ui-monospace, monospace;

  --text-xs: 12px;  --text-sm: 13px;  --text-base: 14px;   /* app UI: 14px base */
  --text-lg: 16px;  --text-xl: 20px;  --text-2xl: 24px;  --text-3xl: 32px;
  --leading-tight: 1.2;  --leading-normal: 1.5;

  /* Space — 4px base, use only these */
  --s1: 4px; --s2: 8px; --s3: 12px; --s4: 16px; --s6: 24px; --s8: 32px; --s12: 48px; --s16: 64px;

  /* Neutrals carry the design */
  --bg: #fafaf9;
  --surface: #ffffff;
  --border: #e7e5e4;
  --text: #1c1917;
  --text-muted: #57534e;
  --text-subtle: #a8a29e;   /* only for placeholders/meta, never body */

  /* One accent */
  --accent: #2f5bea;
  --accent-fg: #ffffff;

  /* Semantic, used only for status */
  --danger: #c4321c; --success: #1f7a4d; --warning: #a15c07;

  --radius: 6px;
  --shadow-pop: 0 4px 16px rgb(0 0 0 / 0.08), 0 1px 2px rgb(0 0 0 / 0.06);
}

Use warm or cool neutrals deliberately (stone vs. slate), not a random gray. Pick weights 400/500/600; avoid 700+ in app UI except for very large display text. Use font-variant-numeric: tabular-nums anywhere numbers line up.

Worked examples
1. Dense tool (dashboard, admin, table-heavy)

Feel: calm, compact, information-first. Think of the best issue trackers and payment dashboards.

13–14px base, 32–36px row height, 1px row dividers, no zebra striping needed
Sidebar in --bg, content on --surface, separated by a single border
Page title 20px/600, no hero, no subtitle paragraph unless it says something useful
Numbers right-aligned, tabular, muted units ($1,240 with .00 in --text-muted)
Status as small text with a 6px colored dot, not a saturated pill
Primary button: solid accent, 32px tall. Secondary: border + surface. Only one primary per view.
css
.table td { padding: 0 var(--s3); height: 36px; border-bottom: 1px solid var(--border); font-size: var(--text-sm); }
.table td.num { text-align: right; font-variant-numeric: tabular-nums; }
.status::before { content: ""; display: inline-block; width: 6px; height: 6px; border-radius: 50%; background: currentColor; margin-right: 6px; }
2. Editorial landing page

Feel: confident, typographic, like a well-made magazine or a design studio's site.

Large display type does the work: 56–80px headline, tight leading (1.05), slight negative tracking (-0.02em)
Left-aligned, asymmetric grid; the headline might span 8 of 12 columns with a short paragraph offset to the right
A serif or distinctive display face for headlines, plain sans for body
One real product screenshot or concrete visual, not abstract blobs
Sections separated by space and thin rules, not alternating colored backgrounds
Copy is short and specific: what it is, who it's for, what changes for them
3. Form / settings page

Feel: quiet and trustworthy.

Single column, max-width ~640px, labels above inputs
Group related fields under a 14px/600 section label with a one-line muted description; separate groups with a border-top and --s8 of space, not cards
Inputs: 36px tall, 1px border, radius 6px, clear focus ring (outline: 2px solid var(--accent); outline-offset: 1px)
Helper and error text directly under the field, 12–13px
Save button bottom-right of the form, disabled until something changes
Destructive actions isolated at the bottom in their own section, with a bordered (not filled) danger button
Interaction and motion
Transitions 120–200ms, ease-out, on color/background/opacity. Don't animate layout for decoration.
Hover: a subtle background or border change. No scaling, no glow.
Every interactive element needs visible :focus-visible styles.
Respect prefers-reduced-motion.
Dark mode

Not inverted light mode. Use a near-black neutral (
#0c0c0d–
#161618), raise surfaces slightly lighter rather than darker, drop border contrast, desaturate the accent a little, and keep body text off pure white (
#e7e7e9).

Final review checklist

Before finishing, look at the result and check:

 Squint test: is the primary action/content obviously the most prominent thing?
 Could any card, border, or background be removed without losing meaning? Remove it.
 Is the accent used in more than ~3 places per screen? Reduce.
 Anything from the slop list present? Justify it or remove it.
 Is all copy specific and plausible, no placeholder marketing phrases?
 Spacing values all from the scale? Alignment edges consistent?
 Empty, loading, error, and overflow states handled?
 Works at 375px wide with no horizontal scroll?
 Contrast passes, focus states visible?
 Would a good designer believe a human made deliberate choices here?
