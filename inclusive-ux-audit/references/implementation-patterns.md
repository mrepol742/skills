# Implementation Patterns for Commonly-Missing Features

Use these as a starting point, adapted to the actual codebase's framework and conventions — don't paste verbatim without checking it fits the existing component structure and state-management approach.

## Text-to-speech (read content aloud)

Browser-native, no library needed, via the Web Speech API's `SpeechSynthesis`:

```javascript
function speak(text, { rate = 1, lang = 'en-US' } = {}) {
  if (!('speechSynthesis' in window)) return false; // feature-detect, fail silently or show a fallback
  window.speechSynthesis.cancel(); // stop any previous utterance first
  const utterance = new SpeechSynthesisUtterance(text);
  utterance.rate = rate;
  utterance.lang = lang;
  window.speechSynthesis.speak(utterance);
  return true;
}
```

UI pattern: a small speaker icon **with a text label** ("Read aloud") next to long-form content or instructions — not icon-only, per the inclusive-design checklist. Provide a way to stop/pause playback, not just start it. Support is broad but voice quality/language coverage varies by OS/browser — don't promise more languages than the user's browser actually provides (`speechSynthesis.getVoices()` to check).

## Speech-to-text (voice input)

Web Speech API's `SpeechRecognition` (prefixed as `webkitSpeechRecognition` in some browsers; **not supported in Firefox or older Safari** as of this writing — feature-detect and provide a normal text input as the fallback, never as the only input method):

```javascript
function startVoiceInput(onResult, { lang = 'en-US' } = {}) {
  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
  if (!SpeechRecognition) return null; // fall back to normal typing — this must always work regardless
  const recognition = new SpeechRecognition();
  recognition.lang = lang;
  recognition.interimResults = false;
  recognition.onresult = (event) => onResult(event.results[0][0].transcript);
  recognition.start();
  return recognition; // caller can call .stop() on it
}
```

UI pattern: a microphone icon **with a text label or tooltip** next to the text input it complements, with a clear visual state for "listening" (so the user knows to speak) and a way to cancel.

## Font-size control (independent of OS/browser zoom)

Use a CSS custom property scaled at the root, persisted in `localStorage`, so all `rem`-based sizing scales together:

```css
:root { --font-scale: 1; }
body { font-size: calc(1rem * var(--font-scale)); }
```
```javascript
function setFontScale(scale) {
  document.documentElement.style.setProperty('--font-scale', scale);
  localStorage.setItem('fontScale', scale);
}
// on load:
const saved = localStorage.getItem('fontScale');
if (saved) document.documentElement.style.setProperty('--font-scale', saved);
```
Expose as 3-4 discrete steps (e.g. "Small / Default / Large / Extra large") rather than a fine-grained slider — easier to use for anyone unfamiliar with the control, and easier to test that layouts don't break at each step.

## High-contrast / theme toggle

Respect the OS-level preference by default, but always offer an explicit override — don't assume `prefers-contrast`/`prefers-color-scheme` alone is enough, since not every OS exposes an easy way to change it and the user may want a different choice per-app:

```css
@media (prefers-color-scheme: dark) { /* dark defaults */ }
@media (prefers-contrast: more) { /* higher-contrast overrides */ }
[data-theme="high-contrast"] { /* explicit override, controlled by the in-app toggle */ }
```
Store the explicit choice (if any) in `localStorage` and apply it via a `data-theme` attribute on `<html>`, checked before the media-query defaults on load.

## Reduced motion

Respect `prefers-reduced-motion` for any animation beyond simple opacity/color transitions — this one should generally be automatic, not require an in-app toggle, since it reflects an OS-level accessibility setting:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

## Skip-to-content link

Visually hidden until focused, first focusable element on the page:

```html
<a href="#main-content" class="skip-link">Skip to main content</a>
<!-- ... -->
<main id="main-content" tabindex="-1">...</main>
```
```css
.skip-link {
  position: absolute; left: -9999px; top: 0;
}
.skip-link:focus {
  left: 0; z-index: 100; padding: 1rem; background: white; /* make it visibly land in place */
}
```

## Plain-language copy rewrites

When rewriting flagged copy, apply this pattern rather than just shortening: **name the concrete action or object, not the abstract process.**

- "An error occurred during submission." → "We couldn't save your changes. Check your internet connection and try again."
- "Please provide valid credentials." → "Enter the email and password you used to sign up."
- "This action is irreversible." → "This will permanently delete your account and all your data — this can't be undone."

Each rewrite: states what happened in concrete terms, and states the next action the user can take.
