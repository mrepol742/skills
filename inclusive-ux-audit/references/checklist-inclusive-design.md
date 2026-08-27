# Inclusive Design & Universal Usability Checklist

The framing for every item: someone with no prior exposure to this app, who may not be comfortable with technology, may be older, may not read the interface's primary language fluently, and may come from a culture with different visual/color conventions. Would they get stuck, and could they recover if they made a mistake?

## 1. Adaptive / assistive features (the "missing features" catalog)

Check for presence, absence, or partial implementation of each. These are the features that let the *app* adapt to the user, rather than requiring the user to adapt to the app:

- **Text-to-speech (read content aloud)** — for long-form content, instructions, or forms; especially valuable for low-literacy users, visual fatigue, or users who process audio better than text.
- **Speech-to-text (voice input)** — for search bars, forms, and especially anywhere typing is the only input method offered; valuable for users with limited typing ability, motor difficulty, or who simply find talking easier than typing.
- **Font-size / zoom control independent of the OS/browser** — a visible, easy-to-find control (not just relying on the user knowing Ctrl/Cmd+ works).
- **High-contrast mode / theme toggle** — beyond just light/dark, a genuinely higher-contrast option for low-vision users.
- **Simplified / "reader" mode** — strips non-essential visual clutter (ads, sidebars, decorative elements) for users who find dense interfaces overwhelming.
- **Language/localization switcher** — visible and easy to find (not buried in settings), if the app serves multiple regions.
- **Reduced-motion toggle** (or automatic respect for the OS-level `prefers-reduced-motion` setting) — for users sensitive to animation, including vestibular disorders.
- **Autosave / draft recovery** — so a mistake, accidental navigation, or connection drop doesn't destroy work; especially important for users unsure whether "did that save?"
- **Clear error recovery, not just error detection** — an error message that says what happened *and* what to do next, not just "Something went wrong."
- **Undo** for any destructive or hard-to-reverse action, not just a confirmation dialog before it.
- **Visible help / contextual guidance** — tooltips, inline hints, or a persistent "?" affordance near complex controls, rather than requiring the user to already know what something does.

For each, if absent: note it as a finding. If present but hard to discover (buried in a menu, no visible affordance): note it as a "present but weak" finding — see the Ground Rules in SKILL.md.

## 2. Onboarding & wayfinding

- Is the primary action on any given screen obvious without reading a manual? (Visually distinct, labeled with a verb, not just an icon.)
- Is there a clear, persistent way to know "where am I" (breadcrumbs, page titles, highlighted nav state) and "how do I get back"?
- Are icon-only controls also labeled with text, or at minimum have a text tooltip/accessible name — icon meaning is not universal across cultures or age groups (a "hamburger" menu, a floppy-disk "save" icon, or a share icon shaped differently per platform can all be unfamiliar).
- Is there a confirmation step before destructive or hard-to-reverse actions (delete, cancel subscription, submit payment)?
- Does the app avoid relying on hover-only interactions to reveal necessary information or controls? (Hover doesn't exist on touchscreens, and isn't discoverable for someone who's never used a mouse.)

## 3. Language & content clarity

- Is the copy in plain language — short sentences, common words, jargon explained or avoided, acronyms spelled out on first use?
- Are idioms, culture-specific references, or humor that don't translate well avoided in core flows (not marketing copy, where more voice is fine)?
- Are numbers, dates, and currency formatted appropriately for the audience, not just one region's convention hardcoded?
- Is error and instructional text specific and actionable ("Enter a valid email like name@example.com" rather than "Invalid input")?
- Is the reading level appropriate for a broad audience (roughly a plain-language / 8th-grade-reading-level target for consumer apps, unless the audience is specifically technical/professional)?

## 4. Touch & interaction ergonomics

- Are tap targets at least ~44×44px (iOS HIG) / ~48×48dp (Material) with adequate spacing between adjacent targets, so imprecise taps (common for older users or anyone on a moving vehicle) don't misfire?
- Are gestures (swipe, pinch, long-press) never the *only* way to perform an important action — is there always a visible, tappable alternative?
- Is there enough time given for time-limited interactions (session timeouts, "are you still there" prompts), or an easy way to extend them, since reaction speed varies a lot by age and context?

## 5. Cognitive load & information structure

- Is information chunked and progressively disclosed rather than presented as a wall of text or an overwhelming single form?
- Are choices limited to what's actually needed at each step (avoiding decision paralysis from too many simultaneous options)?
- Are sensible defaults provided so a user can proceed without understanding every option?
- Is visual hierarchy clear enough that the eye is guided to what matters most on the screen, without needing to read everything to find it?

## 6. Cultural & age adaptability

- Does the app avoid assuming one culture's color conventions (e.g. red/green for good/bad isn't universal; white/red have different connotations across cultures)?
- Does it support right-to-left layouts if serving RTL-language audiences, rather than assuming left-to-right is universal?
- Does it avoid slang or generational-specific references in core flows?
- If imagery/icons depict people, are they reasonably diverse rather than implicitly assuming one demographic is "the user"?
