# Design, Typography & Color Rules of Thumb

Concrete rules to check the visual design and copy against, with the reasoning — apply judgment, since every rule has legitimate exceptions, but a violation should be justified rather than accidental.

## Color

- **Avoid gradients on backgrounds behind text or as the primary surface color.** A gradient's contrast against overlaid text varies across its span — text that passes contrast checks at one end of the gradient can fail at the other. Gradients also read as more "decorative/trendy" than "functional," which works against the goal of an interface that feels calm and predictable to someone unfamiliar with current design trends. Flat, single colors (or very subtle, low-contrast-shift gradients used purely for texture, never behind text) are safer.
- **Maintain contrast ratios**: 4.5:1 for body text, 3:1 for large text/UI boundaries (see the WCAG checklist for the formal citation) — but treat this as a baseline, not a ceiling; higher contrast generally helps older users and anyone with common age-related vision changes (presbyopia, reduced contrast sensitivity).
- **Don't rely on color alone to convey meaning** — pair it with an icon, label, or pattern, both for color-blind users and because color conventions (red = danger, green = success) aren't universal across cultures.
- **Keep the palette disciplined**: 2-3 primary/brand colors plus a neutral scale, rather than many saturated colors competing for attention — a busy palette increases cognitive load and makes it harder to tell what's actually important (a button vs. decoration).
- **Avoid pure black (#000) on pure white (#FFF) for large blocks of body text** — the extreme contrast can cause visual "vibration"/halation for some readers, especially with certain fonts at small sizes. A very dark gray (e.g. #1a1a1a) on off-white is usually more comfortable while still passing contrast requirements.
- **Be consistent about what a color means** — if red means "error" in one place, it shouldn't mean "featured" or "urgent-but-good" elsewhere in the same app.

## Typography

- **Body text ≥ 16px**, with generous line-height (1.5 is a reasonable default) — smaller text disproportionately excludes older users and anyone reading on a small or lower-resolution screen.
- **Sans-serif for functional UI text** in most digital contexts (generally more legible on-screen at small sizes than serif or decorative fonts); reserve decorative/display fonts for large headings only, never for body copy, form labels, or anything functional.
- **Avoid thin font weights (< 400) for small text** — they lose legibility on many displays, especially for users with any visual impairment.
- **Limit to 1-2 font families** — more adds visual noise without adding clarity.
- **Avoid full text justification** (justified both left and right) for body copy — it creates uneven word spacing ("rivers" of white space) that's harder to read, particularly for people with dyslexia. Left-aligned (ragged-right) is generally more legible.
- **Avoid long unbroken paragraphs** — break into shorter paragraphs, use headings and lists, so the content is scannable rather than requiring a full read to find what's relevant.

## Copy & microcopy

- **Avoid em dashes in UI copy.** Prefer a period, comma, or restructuring the sentence into two shorter ones. Em dashes are a stylistic choice that reads as more literary/formal than functional UI copy usually needs, can render inconsistently in some fonts, and add a small parsing burden — a shorter sentence with standard punctuation is easier to scan at a glance, which is the whole point of UI copy. This is a style rule, not a hard technical requirement — apply it to interface text, error messages, labels, and onboarding copy; it matters less in long-form editorial or marketing content the person has full authorial control over.
- **Avoid jargon and unexplained acronyms** in button labels, form fields, and error messages — this is the single highest-leverage clarity fix for non-technical users.
- **Prefer verbs on buttons** ("Save changes", "Delete account") over vague labels ("OK", "Submit", "Continue") so the action's consequence is clear without prior context.

## Structure & layout

- **Keep primary navigation and key actions in a consistent position across pages** — predictability reduces the learning curve to near zero after the first page.
- **Avoid deeply nested menus** (more than 2-3 levels) — each additional level is a place a user can get lost or give up.
- **Always provide a visible way back** (breadcrumb, back button, persistent nav) rather than relying on the browser back button as the only escape hatch.
- **Prefer pagination with a clear end over infinite scroll for goal-oriented tasks** (e.g. searching for a specific item, reviewing a list to make a decision) — infinite scroll works for passive browsing but makes it hard to gauge progress or return to a specific spot for a task with a defined goal.
- **Use skeleton loaders or progress indicators instead of blank white screens** during load — a blank screen reads as "broken" to an unfamiliar user, while a skeleton communicates "something is happening."
- **Avoid auto-playing audio/video** — it's disorienting, especially for users unfamiliar with the interface, and can be genuinely distressing for some users (autoplaying sound in a public/quiet setting, sudden motion for vestibular-sensitive users).
