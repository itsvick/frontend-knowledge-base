# Accessibility

> Part of: 03-HTML

## Overview

Web accessibility (a11y) means building UIs that people using assistive technology — screen readers, keyboard-only navigation, switch devices — can perceive, operate, and understand, not just people using a mouse and looking at a screen. It combines semantic HTML, ARIA attributes, keyboard support, sufficient visual contrast, and testing with real assistive tools rather than relying on automated linters alone.

## Key Concepts

- ARIA (`role`, `aria-label`, `aria-labelledby`, `aria-describedby`, `aria-live`) supplements semantic HTML — it doesn't replace it. Prefer a native semantic element over adding ARIA to a `<div>` when one exists ("no ARIA is better than bad ARIA").
- `tabindex` controls keyboard focus order: `tabindex="0"` inserts a naturally-non-focusable element into the normal tab order; `tabindex="-1"` makes an element programmatically focusable (e.g. via `.focus()`) but removes it from the tab sequence; positive values override natural DOM order and should generally be avoided.
- Sufficient color contrast (WCAG AA/AAA ratios) and never relying on color alone to convey meaning or state.
- Visible focus indicators — never `outline: none` without supplying a replacement focus style.
- Testing: automated tools (axe DevTools, Lighthouse accessibility audit) catch common issues, but manual testing — navigating with just a keyboard, and with a screen reader (VoiceOver/NVDA) — catches problems automated tools miss.

## Interview Questions & Answers

### Q1. How do you make web applications accessible?

#### Answer

Accessibility is built in layers, not bolted on afterward. Start with semantic HTML — native elements (`<button>`, `<nav>`, `<label>`, etc.) come with built-in keyboard support, roles, and states for free, so reach for them before adding ARIA. ARIA attributes (`role`, `aria-label`, `aria-labelledby`, `aria-describedby`, `aria-live` for dynamic content announcements) supplement semantic HTML for cases no native element covers, but they never replace an available native element — adding `role="button"` to a `<div>` doesn't give it keyboard behavior or focus management for free. The rule of thumb: no ARIA is better than bad ARIA.

Keyboard support matters as much as markup: `tabindex="0"` inserts an element that isn't natively focusable (e.g. a `<div>` acting as a button) into the normal tab order; `tabindex="-1"` makes an element programmatically focusable via `.focus()` (useful for moving focus to a modal or an error message) but removes it from the tab sequence; positive `tabindex` values should generally be avoided since they override the natural DOM-order tab sequence and quickly become confusing to maintain.

Visual/perceptual accessibility requires sufficient color contrast (meeting WCAG AA/AAA ratios) and never using color alone to convey meaning (e.g. pairing a red error state with an icon or text, not just a color change). Focus indicators must stay visible — never set `outline: none` without providing a clear replacement focus style, or keyboard users lose track of where they are on the page.

Finally, verify with both automated and manual testing: run axe DevTools or a Lighthouse accessibility audit to catch common, mechanically-detectable issues (missing labels, contrast failures, missing alt text), then actually navigate the page using only a keyboard (Tab/Shift+Tab/Enter/Space/Esc) and with a screen reader (VoiceOver on macOS, NVDA on Windows) — this catches problems automated tools structurally can't, like illogical focus order, unannounced dynamic content, or a modal that doesn't trap focus.

#### Code Example

```html
<!-- Custom toggle built from a div: needs ARIA + tabindex + manual key handling -->
<div role="button" tabindex="0" aria-pressed="false" id="toggle">Toggle</div>
<script>
  const toggle = document.getElementById("toggle");
  const press = () => {
    const pressed = toggle.getAttribute("aria-pressed") === "true";
    toggle.setAttribute("aria-pressed", String(!pressed));
  };
  toggle.addEventListener("click", press);
  toggle.addEventListener("keydown", (e) => {
    if (e.key === "Enter" || e.key === " ") press();
  });
</script>

<!-- Prefer this instead — a native <button> gets all of the above for free -->
<button aria-pressed="false" id="toggle-native">Toggle</button>

<!-- aria-live announces dynamically inserted content to screen readers -->
<div aria-live="polite" id="status"></div>
```

#### Follow-up Questions

- What's the difference between `aria-live="polite"` and `aria-live="assertive"`?
- Why is `outline: none` considered a common anti-pattern, and how would you replace it?

## Common Pitfalls

- Adding ARIA roles to a `<div>`/`<span>` instead of using the native semantic element that already provides the needed behavior (keyboard support, focus, states) for free.
- Using positive `tabindex` values, which override natural DOM order and make the tab sequence unpredictable.
- Removing focus outlines (`outline: none`) without providing a replacement focus style, leaving keyboard users with no visual indication of where focus is.
- Relying on color alone to convey state or meaning (e.g. only red text for an error) without an icon or text alternative.
- Testing only with automated tools (axe/Lighthouse) and skipping manual keyboard/screen-reader testing, missing issues like broken focus order or unannounced dynamic updates.
