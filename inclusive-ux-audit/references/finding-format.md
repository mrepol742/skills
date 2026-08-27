# Finding Format & Severity

## Finding Format

```
[SEVERITY] Title
Track: Inclusive Design | Accessibility (WCAG SC #) | Both
Where: page/component/file (or "site-wide" for a pattern seen throughout)
Who's affected: <the specific kind of user this blocks or slows down, concretely — not just "some users">
Issue: <what's missing or wrong>
Why it matters: <the concrete scenario where this trips someone up>
Fix: <specific recommendation — reference implementation-patterns.md or design-style-rules.md where applicable>
```

For items reviewed and found fine, use a short form so the review's coverage stays visible:

```
✅ [Inclusive Design — Touch ergonomics] Tap targets throughout the mobile nav measured ~48px with adequate spacing — no issues found.
```

## Severity rubric

- **Critical** — blocks a core task entirely for a meaningful chunk of the audience (e.g. a checkout flow with no keyboard access at all, or a required voice-only interaction with no text alternative).
- **High** — significantly harder or more error-prone for a meaningful chunk of the audience, but a workaround exists (e.g. low contrast that's readable in good lighting but not in bright sun or for low-vision users; a destructive action with no undo).
- **Medium** — a real gap that adds friction or excludes a smaller group, or a "present but hard to find" issue (e.g. a font-size control that exists but is buried in settings).
- **Low** — a polish-level improvement or a rule-of-thumb violation with no demonstrated real-world friction yet (e.g. a decorative gradient that doesn't sit behind any text).

## Summary table

Close the report with:

| # | Area | Track | Severity | Status |
|---|---|---|---|---|

One row per finding, most severe first. "Status" is "Open" for a first pass; use "Fixed"/"Open"/"Deferred" for a re-review.
