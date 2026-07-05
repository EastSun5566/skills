---
name: web-interface-guidelines
description: Review and improve web interfaces for usability, accessibility, interaction behavior, responsive design, and performance. Use for UI/UX reviews, accessibility audits, or frontend interface improvements.
---

# Web Interface Guidelines

> Modernized version based on https://interfaces.rauno.me/

This document is a practical checklist for building good web interfaces. Use this checklist when reviewing or improving a web interface. Apply only the guidance relevant to the product, platform, and existing design system.

When reviewing an interface:

1. Inspect the implementation, not only screenshots.
2. Test important flows with keyboard, touch, reduced motion, zoom, slow networks, and realistic data.
3. Report concrete findings grouped as must fix, should fix, and nice to have.
4. For each finding, explain the problem, impact, and smallest practical fix.

This checklist supplements [WCAG 2.2](https://www.w3.org/TR/WCAG22/), [WAI-ARIA](https://www.w3.org/TR/wai-aria-1.2/), and the [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/). It is not a conformance audit and does not replace those standards.

## Core Principles

- Prefer native HTML elements and browser behavior before custom controls.
- Use ARIA only when native HTML cannot express the required semantics.
- Preserve accessible name, role, state, value, keyboard behavior, focus behavior, and form behavior when building custom controls.
- Keep behavior predictable, reversible when possible, and close to the user's action.
- Preserve basic usability when JavaScript, custom fonts, images, or animation fail.
- Follow the existing design system unless it conflicts with usability or accessibility.
- Measure performance before optimizing.

## Forms

- Give every control a visible label or accessible name; do not use placeholder text as the only label.
- Associate labels with controls so clicking or tapping the label focuses the control.
- Wrap submitted controls in a `<form>` so standard submission, including <kbd>Enter</kbd>, works.
- Use the appropriate input `type`, `autocomplete` token, validation attributes, and mobile keyboard hints.
- Disable autocomplete or spellcheck only when their output would be harmful or meaningless.
- Keep helper text and validation messages close to the relevant control.
- Explain how to correct an error; do not rely only on color or a generic failure message.
- Connect helper and error text with `aria-describedby` when appropriate.
- Set `aria-invalid="true"` when rendering a custom invalid state.
- Move focus to an error summary only when it helps users recover from a failed submission.
- Preserve user input after validation, network, or server errors.
- Communicate required and optional fields consistently.
- Use native `disabled` when a control should be unavailable and undiscoverable by keyboard.
- If an unavailable control must remain focusable, use `aria-disabled="true"`, prevent activation in code, and explain how to enable it.

## Interaction and Feedback

- Use `<button>` for actions and `<a>` for navigation.
- Make the complete visible control clickable; avoid small icon-only hit areas or dead space between list items.
- Make pointer targets at least 24×24 CSS pixels, or provide sufficient spacing or an equivalent alternative.
- Provide clear hover, active, focus, selected, loading, success, and error states where relevant.
- Do not depend on hover for essential information or actions.
- Show feedback near the action; reserve global notifications for cross-page or asynchronous events.
- Keep immediate input feedback independent from slow network requests, analytics, and logging.
- Use optimistic updates only when failure can be clearly communicated and safely reversed.
- Prefer undo for frequent, low-risk destructive actions; confirm irreversible, expensive, or security-sensitive actions.
- Give important drag, swipe, and long-press actions a keyboard or direct-control alternative.
- Use `user-select: none` only where text selection actively interferes with interaction.
- Give decorative overlays `pointer-events: none` when they could intercept input.
- Make empty states explain what is missing and suggest a useful next action.

## Accessibility and Keyboard

- Give every interactive element an accessible name.
- Give icon-only controls a name through visible text, `aria-labelledby`, or `aria-label`; hide decorative icons from assistive technologies.

```html
<button type="button" aria-label="Close">
  <svg aria-hidden="true" focusable="false">...</svg>
</button>
```

- Use meaningful `alt` text for informative images and empty `alt=""` for decorative images.
- Do not use color, motion, position, or sound as the only way to communicate state.
- Maintain sufficient contrast for text, controls, focus indicators, and important states.
- Keep focus indicators clearly visible; prefer `:focus-visible` and do not remove outlines without an equivalent replacement.
- Ensure sticky headers, floating toolbars, and overlays do not obscure focused elements.
- Keep content usable at 200% zoom and after text resizing.
- Remove hidden or off-screen controls from keyboard and accessibility navigation with `hidden`, `inert`, or conditional rendering.
- Follow ARIA APG keyboard patterns for custom widgets instead of inventing behavior.
- Preserve expected keyboard behavior: <kbd>Tab</kbd> moves between controls, <kbd>Enter</kbd> or <kbd>Space</kbd> activates buttons, and arrow keys navigate composite widgets.
- Document destructive shortcuts and make their effects reversible when possible.
- Test complex custom controls with keyboard and at least one screen reader.
- Test forced-colors and high-contrast modes; do not rely only on gradients, shadows, or opacity for state.

## Dialogs, Popovers, and Tooltips

- Use `<dialog>` with `showModal()` when the background must become inert.
- Use `popover` for non-modal menus, teaching bubbles, and lightweight contextual panels when it fits the target browser policy; otherwise provide equivalent non-modal behavior.
- For custom modals, implement focus containment, inert background content, scroll handling, Escape behavior, and focus restoration.
- Close non-critical dialogs, menus, popovers, and tooltips with <kbd>Escape</kbd>.
- Avoid stacked modals and focus traps without an obvious exit.
- Keep tooltips non-interactive; use a dialog, popover, or disclosure when content contains controls.
- Show tooltips on keyboard focus as well as hover, and keep them dismissible.

## Motion and Touch

- Respect `prefers-reduced-motion`; disable or simplify motion per component instead of applying a global reset.
- If code waits for `transitionend` or `animationend`, provide an immediate completion path when motion is disabled.
- Keep frequent animations short, subtle, non-blocking, and limited to properties such as transform and opacity.
- Avoid layout-heavy animation, large zooms, parallax, and persistent background motion unless they materially aid understanding.
- Pause looping animations and media when they are off-screen.
- Do not use animation as the only way to communicate a state change.

```css
html {
  scroll-behavior: smooth;
}

@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }

  .dialog,
  .popover,
  .toast {
    animation: none;
    transition: none;
  }
}
```

- Apply hover styles only on devices that support hover.
- Avoid autofocus on touch devices when it would unexpectedly open the keyboard.
- Keep text inputs at a size that avoids unwanted mobile zoom, commonly at least `16px`.
- Do not disable pinch zoom globally.
- Use the narrowest `touch-action` that preserves native scrolling and zooming.

## Layout, Typography, and Content

- Keep layouts usable across supported viewport sizes, zoom levels, text expansion, and input methods.
- Avoid fixed-height containers that clip content after text resizing or translation.
- Use readable font sizes, line heights, contrast, and tested font weights.
- Avoid font or weight changes on hover that cause layout shift.
- Load fonts intentionally and reduce layout shift with suitable fallbacks and font metrics.
- Use tabular numbers for timers, counters, prices, and numeric tables.
- Do not truncate important text without a way to inspect the full value.
- Use locale-aware date, time, number, currency, and plural formatting.
- Prefer explicit dates when relative or numeric dates could be ambiguous.
- Avoid sentence fragments that translators cannot reorder.
- Test supported interfaces with long text, CJK, emoji, code, mixed directionality, and right-to-left content.

## Performance

- Optimize interaction responsiveness as well as initial loading.
- Keep typing, clicking, dragging, scrolling, and route changes free from long main-thread tasks.
- Split expensive work and defer non-critical rendering or computation.
- Lazy-load non-critical images and iframes, but do not lazy-load the likely LCP image.
- Set explicit media dimensions and use responsive image sources to reduce layout shift and unnecessary transfer.
- Pause, lazy-load, or unmount off-screen video and other expensive media.
- Virtualize only genuinely long lists; avoid the complexity for small collections.
- Use `IntersectionObserver` and `ResizeObserver` instead of continuous polling when appropriate.
- Debounce or throttle expensive high-frequency handlers.
- Use `will-change` only for measured problems and remove it when no longer needed.
- Test on representative devices and networks with browser performance tools and real-user metrics when available.

## Resilience and Trust

- Preserve user work across validation errors, reloads, navigation, temporary network loss, and failed saves when possible.
- Show clear saving, saved, failed, conflict, and offline states for persistent edits.
- Make failures retryable and never silently discard user-authored content.
- Preserve layout during loading; use skeletons only when they resemble the final content and the wait is long enough to justify them.
- Write human-readable, action-oriented errors; do not expose raw backend errors.
- Protect destructive actions with undo, trash, version history, or confirmation according to their risk.
- Complete authentication and authorization checks before rendering protected content.
- Keep secrets and sensitive data out of URLs, page titles, analytics, referrers, screenshots, and error messages.
- Support copy, paste, and password managers in password, token, API key, and recovery-code fields.
- Label generated or automatically transformed content when provenance affects trust.
- Let users review, edit, accept, reject, or undo generated changes.
- Require explicit confirmation before generated actions affect other users, billing, permissions, publishing, or deletion.

## Visual Design

- Make hierarchy and primary actions clear without creating multiple competing primary buttons.
- Keep destructive actions visually distinct from primary actions.
- Prefer inline editing for nearby content and modal flows for focused blocking tasks.
- Use progressive disclosure for advanced or infrequent settings.
- Choose safe, useful, and reversible defaults.
- Keep the interface understandable without relying on visual polish alone.

## References

- [WCAG 2.2](https://www.w3.org/TR/WCAG22/)
- [WAI-ARIA 1.2](https://www.w3.org/TR/wai-aria-1.2/)
- [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
- [MDN: Forms](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms)
- [MDN: `<dialog>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dialog)
- [MDN: Popover API](https://developer.mozilla.org/en-US/docs/Web/API/Popover_API)
- [MDN: `prefers-reduced-motion`](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media/prefers-reduced-motion)
- [web.dev: Interaction to Next Paint](https://web.dev/articles/inp)
