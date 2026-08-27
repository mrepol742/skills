---
name: inclusive-ux-audit
description: Audit a web app's UX for real, possibly non-technical, users, not code quality or performance. Two tracks, kept separate, inclusive/universal-usability design regardless of tech literacy, age, or culture (missing features like text-to-speech, speech-to-text, font-size controls, plain-language content, high-contrast mode), and formal WCAG/accessibility compliance (ARIA, contrast, keyboard nav, screen readers). Also reviews visual design, layout, typography, and color against rules of thumb (avoid gradients, avoid em dashes in UI copy, tap-target sizes, contrast ratios, font legibility). Use when the user wants a UX/usability/design review, asks if something is accessible or easy for anyone to use, asks what's missing for elderly, non-technical, or international users, or wants a typography/color review. Produces a findings report with severity, then offers to implement fixes.
---

# Inclusive UX & Area-of-Improvement Audit

The lens for this skill is different from a code-quality or performance audit: the question throughout is **"if someone dropped into this app with no assumptions — doesn't know tech jargon, is 70 years old, speaks a different first language, has never seen this exact UI pattern before — could they figure out what to do and recover from a mistake?"** Findings should read as "here's where a real, unprepared person gets stuck" rather than "here's a code smell."

## Two tracks — keep them separate in the report

1. **Inclusive / universal-usability design** — `references/checklist-inclusive-design.md`. This is the primary focus: missing adaptive features (text-to-speech, speech-to-text, font scaling, plain-language content, high-contrast/simplified modes), onboarding clarity, touch ergonomics, cognitive load, and cultural adaptability. Most of this is about *default* behavior for users who never open a settings menu.
2. **Formal accessibility (WCAG) compliance** — `references/checklist-accessibility-wcag.md`. This is the standards-based track: ARIA roles/labels, contrast ratio math, keyboard operability, screen reader semantics. It overlaps with track 1 in places (both care about contrast, for instance) but is checked against the actual WCAG 2.2 success criteria rather than general usability judgment.

Report these as two clearly labeled sections. A gap can appear in both if it matters for both reasons (e.g. low contrast fails WCAG AA *and* makes text hard to read for an older user with common presbyopia) — say so rather than picking one bucket arbitrarily.

## Design, structure, typography & color review

`references/design-style-rules.md` has the concrete rules of thumb, with the reasoning behind each one — including the two the person specifically called out (avoid gradients, avoid em dashes in UI copy) and the rest of the set (contrast ratios, font sizing/weight/line-height, color palette discipline, layout consistency, avoiding cognitive-heavy patterns like deep nested menus or infinite scroll for goal-oriented tasks). Apply this section to both visual design and any UI copy/microcopy you can see.

## Workflow

1. **Scope the review.** If given a live URL or screenshots, review what's actually rendered — layout, copy, visible controls, apparent flows. If given a repo, also check the codebase for **absence** of the adaptive features in the checklist (e.g. grep for any speech-synthesis/speech-recognition usage, font-size CSS custom properties, a language switcher, a `prefers-reduced-motion` media query) — a feature genuinely missing from the code is a stronger finding than one you can't confirm either way from a screenshot.
2. **Walk both checklists.** For each item, give an explicit verdict — present/absent/partial — with a one-line reason. Don't skip items silently.
3. **Walk the design/style rules** against the actual visual design and copy.
4. **Write the report** using `references/finding-format.md` — each finding gets a severity, which track(s) it belongs to, why it matters for the "clueless/unprepared user" framing, and a concrete fix.
5. **Ask before implementing.** After the report, ask whether to implement the missing features/fixes. If yes, use `references/implementation-patterns.md` for ready-to-adapt code (Web Speech API for TTS/STT, a persisted font-size control, a high-contrast toggle, `prefers-reduced-motion` handling, skip-links, plain-language copy rewrites) rather than inventing these from scratch each time.

## Ground rules

- **Don't assume the person's target users match you.** Default to the broadest reasonable audience (a range of ages, first languages, and tech comfort levels) unless the person describes a narrower one — ask if genuinely unclear which audience matters most, since a banking app for retirees and a developer tool have very different bars for "easy."
- **Distinguish "missing" from "present but weak."** A feature that exists but is hard to find (e.g. font-size control buried three menus deep) is a real but different finding than one that doesn't exist at all — say which it is.
- **Don't recommend a feature just because it's on the checklist.** If speech-to-text genuinely adds little value for a given app (e.g. a data-entry tool with mostly numeric fields), say so rather than padding the report — the goal is real improvement, not checklist completion.
- **Respect what the person already decided.** If they explain a deliberate design choice (e.g. a minimalist single-language MVP), note the trade-off rather than re-litigating it as a "finding."

## Reference files

- `references/checklist-inclusive-design.md` — universal usability checklist and the missing-features catalog (TTS, STT, font scaling, high-contrast/simplified modes, plain-language content, onboarding, touch ergonomics, cultural adaptability, cognitive load).
- `references/checklist-accessibility-wcag.md` — WCAG 2.2-mapped compliance checklist (perceivable/operable/understandable/robust).
- `references/design-style-rules.md` — concrete typography, color, and layout rules of thumb with rationale.
- `references/implementation-patterns.md` — ready-to-adapt code for the most commonly-missing features.
- `references/finding-format.md` — report format, severity rubric, and summary table.
