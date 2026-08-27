# Formal Accessibility Checklist (WCAG 2.2)

Organized by WCAG's four principles. Cite the specific success criterion (SC) number where you can — it makes findings verifiable and gives the person a standard to point to.

## Perceivable

- **Text alternatives (1.1.1)**: images, icons-as-buttons, and non-text content have meaningful `alt` text (or `alt=""` for genuinely decorative images) — not filenames or "image123.png".
- **Captions/transcripts (1.2.x)**: video has captions; audio-only content has a transcript.
- **Contrast ratio (1.4.3, 1.4.11)**: body text ≥ 4.5:1 against its background; large text (≥18pt or 14pt bold) ≥ 3:1; UI component boundaries/icons ≥ 3:1. Check actual rendered colors, not just the design file's intended palette.
- **Resize text (1.4.4)**: text can be resized to 200% via browser zoom without loss of content or functionality (no fixed-height containers clipping scaled text).
- **Text spacing (1.4.12)**: content doesn't break when a user applies custom line-height/letter-spacing/word-spacing overrides.
- **Reflow (1.4.10)**: content reflows to a single column at 320px width / 400% zoom without horizontal scrolling for anything other than genuinely 2D content (maps, data tables).
- **Don't rely on color alone (1.4.1)**: status/error/success states have a non-color indicator too (icon, text label, pattern), for color-blind users (~8% of men).

## Operable

- **Keyboard accessible (2.1.1)**: every interactive element (including custom dropdowns, modals, date pickers) can be reached and operated with keyboard alone — Tab, Shift+Tab, Enter/Space, Escape, arrow keys where conventional.
- **No keyboard trap (2.1.2)**: focus can always move away from any component via keyboard (common failure: a modal or embedded widget that swallows Tab).
- **Focus visible (2.4.7, 2.4.11)**: a clear, visible focus indicator on every focusable element — don't strip `outline` without providing a replacement.
- **Skip link (2.4.1)**: a "skip to main content" link (visually hidden until focused) so keyboard users don't have to tab through the entire nav on every page.
- **Target size (2.5.8)**: interactive targets at least 24×24px minimum (44×44px recommended — overlaps with the inclusive-design checklist's touch ergonomics item, cite both if relevant).
- **No seizure risk (2.3.1)**: nothing flashes more than 3 times per second.
- **Timing adjustable (2.2.1)**: session timeouts and time-limited content can be extended, turned off, or adjusted, unless truly essential (e.g. a live auction).

## Understandable

- **Labels/instructions (3.3.2)**: every form field has a visible, programmatically-associated label (not just a placeholder, which disappears on input and often fails contrast requirements).
- **Error identification (3.3.1)** and **error suggestion (3.3.3)**: errors are identified in text (not color alone), associated with the specific field, and suggest how to fix them.
- **Consistent navigation (3.2.3)** and **consistent identification (3.2.4)**: nav structure and recurring components (icons, buttons) stay consistent across pages.
- **Language of page (3.1.1)**: the page declares its language (`<html lang="en">`) so screen readers use correct pronunciation rules, and `lang` is updated for embedded content in a different language.

## Robust

- **Valid semantic HTML / ARIA (4.1.2)**: custom interactive components (dropdowns, tabs, modals, sliders) expose the correct role, name, and state to assistive tech — either via native elements (`<button>`, `<select>`) or correct ARIA roles/attributes when a custom implementation is unavoidable. Flag `<div onClick>` used as a button with no role/keyboard handling.
- **Status messages (4.1.3)**: dynamic updates (form submitted, item added to cart, search results loaded) are announced to screen readers via `aria-live` regions, not just a visual toast that a screen reader user would never know appeared.
- **Name/role/value correctness**: for custom widgets, confirm the accessible name matches the visible label (don't rely on `aria-label` alone when visible text exists — that can create a mismatch between what's seen and what's announced).
