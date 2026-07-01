---
name: web-interface-guidelines
description: Use practical web interface guidelines to review and improve UI, accessibility, forms, keyboard behavior, focus states, loading states, error states, responsive layout, performance, and interaction details.
---

# Web Interface Guidelines

> Modernized version based on https://interfaces.rauno.me/

This document is a practical checklist for building good web interfaces. Use these guidelines to review, design, debug, or improve web interfaces.

When reviewing a UI, give concrete feedback grouped by severity:

- Must fix
- Should fix
- Nice to have

For each issue, explain the problem, why it matters, and how to fix it.

It does not duplicate [WCAG 2.2](https://www.w3.org/TR/WCAG22/), [WAI-ARIA 1.2](https://www.w3.org/TR/wai-aria-1.2/), or the [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/). Prefer semantic HTML first. When building custom widgets, follow WAI-ARIA semantics and ARIA APG keyboard patterns.

## Principles

- Prefer native HTML elements before custom components.
- Do not use ARIA to replace native semantics when native HTML can express the same behavior.
- Custom controls should preserve accessible name, role, state, value, keyboard behavior, focus behavior, and form behavior where relevant.
- Interface behavior should be predictable, reversible when possible, and close to the user's action.
- Interfaces should remain usable when animation, custom fonts, images, or JavaScript fail.
- Test with keyboard navigation, screen readers, touch input, reduced motion settings, high contrast settings, slow devices, and real data.

## Forms and Controls

- Every form control should have a visible label or an accessible name.
- Prefer visible labels over placeholder-only labels.
- Placeholder text should provide examples or hints, not replace labels.
- Clicking or tapping an input label should focus the associated input field.
- Inputs should be wrapped with a `<form>` when submission is expected, so users can submit by pressing <kbd>Enter</kbd>.
- Inputs should use an appropriate `type`, such as `email`, `password`, `url`, `tel`, `number`, or `search`.
- Use appropriate `autocomplete` tokens instead of disabling autocomplete by default.
  - Examples: `email`, `username`, `current-password`, `new-password`, `one-time-code`, `name`, `street-address`.
- Disable `autocomplete` only when autofill is harmful, misleading, or meaningless.
- Disable `spellcheck` only for non-natural-language inputs, such as code, usernames, slugs, emails, URLs, tokens, IDs, and command syntax.
- Use HTML form validation when appropriate, such as `required`, `minlength`, `maxlength`, `min`, `max`, `step`, and `pattern`.
- Do not rely only on native validation messages when product-specific guidance is needed. Provide clear inline error messages.
- Error messages should be close to the relevant field and explain how to fix the problem.
- Use `aria-describedby` to connect inputs with helper text or error messages when appropriate.
- Use `aria-invalid="true"` for invalid fields when rendering custom validation state.
- Required and optional fields should be communicated consistently.
- Avoid clearing user input after a failed submission.
- Preserve user input across validation errors.
- Use fieldsets and legends for grouped controls such as radio groups and checkbox groups.
- Prefer radio buttons when users must choose one option from a small set.
- Prefer checkboxes when users can choose multiple options or toggle a boolean state.
- Avoid custom select, combobox, date picker, or autocomplete components unless native controls are insufficient.
- Custom comboboxes, listboxes, menus, tabs, dialogs, and toolbars should follow ARIA APG patterns.

## Interaction

- Links should be used for navigation. Buttons should be used for actions.
- Avoid attaching click handlers to non-interactive elements unless you also provide correct role, keyboard behavior, focus behavior, and accessible name.
- Clicking a visual input wrapper should focus the input when it is clearly part of the same control.
- Input prefix and suffix decorations, such as icons, should usually be visually placed inside the input area with padding, not inserted as text content.
- Decorative input adornments should not interrupt text selection, pointer behavior, or screen reader output.
- Toggles should usually take effect immediately.
- Destructive, expensive, or security-sensitive toggles may require confirmation, undo, or an intermediate state.
- Prevent duplicate submissions with client-side state, request de-duping, idempotency keys, or server-side protection.
- During submission, disable repeated activation when appropriate, show a loading state, and preserve a way to recover from errors.
- Do not leave users trapped in a permanently disabled state after a failed submission.
- Use optimistic updates when they make the interface feel faster, but provide rollback and feedback on failure.
- Avoid optimistic updates for irreversible, security-sensitive, financially significant, or permission-changing actions unless recovery is very clear.
- Use `user-select: none` only where accidental text selection harms interaction, such as buttons, drag handles, sliders, and toolbar icons.
- Avoid applying `user-select: none` to selectable content, table rows, cards with text, article content, or messages.
- Decorative elements, such as glows, gradients, overlays, and particles, should use `pointer-events: none` when they could otherwise hijack pointer events.
- Interactive elements in a vertical or horizontal list should have no dead areas between items. Increase padding or the clickable area instead.
- Show feedback close to the related action when possible; reserve global notifications for cross-page, asynchronous, or system-level events.
- Empty states should explain what is missing and suggest a useful next action.

## Accessibility

- Use semantic HTML first.
- Reach for ARIA only when native HTML cannot express the required semantics.
- Every interactive element should have an accessible name.
- Icon-only interactive elements need an accessible name, preferably from visible text or `aria-labelledby`; use `aria-label` when no visible label exists.

```html
<button aria-label="Close">
  <svg aria-hidden="true" focusable="false">...</svg>
</button>
```

- Decorative icons should be hidden from assistive technologies.

```html
<svg aria-hidden="true" focusable="false">...</svg>
```

- Images that communicate content should use `<img>` with appropriate `alt`.
- Decorative images should use empty `alt=""`.
- Meaningful illustrations built with HTML or SVG should have an accessible name, or be hidden when decorative.
- Do not rely on color alone to communicate state.
- Text and important UI states should meet WCAG contrast expectations.
- Focus indicators should be clearly visible.
- Prefer `:focus-visible` for keyboard focus styling.
- Do not remove focus styles unless replacing them with an equally visible alternative.
- `outline` is usually preferred for focus rings because it does not affect layout.

```css
:focus-visible {
  outline: 2px solid CanvasText;
  outline-offset: 2px;
}
```

- Focused elements should not be hidden behind sticky headers, floating toolbars, cookie banners, or overlays.
- Content should remain usable at 200% zoom.
- Avoid fixed containers that cut off content when text is resized.
- Avoid hover-only content for important information.
- Pointer targets should generally be at least 24×24 CSS pixels, or provide enough spacing or an equivalent alternative.
- Important drag-and-drop actions should have a non-drag alternative.
- Authentication should not rely only on memorization, transcription, puzzles, or other cognitive tasks unless alternatives are available.
- Multi-step flows should avoid asking users to re-enter information they already provided.
- Help mechanisms, when available across multiple pages, should appear in a consistent location.
- Disabled native controls are not focusable and may be skipped by keyboard users.
- Do not rely on tooltips attached to native disabled controls to explain why an action is unavailable.
- Prefer inline helper text, validation messages, or enabled controls that explain unmet requirements.
- If a disabled-looking control must remain discoverable, consider `aria-disabled="true"` and prevent activation in JavaScript.

```html
<button aria-disabled="true" type="button">Publish</button>
```

```js
button.addEventListener("click", (event) => {
  if (button.getAttribute("aria-disabled") === "true") {
    event.preventDefault();
    return;
  }

  publish();
});
```

- Tooltips triggered by hover or focus should not contain interactive content.
- If content contains buttons, links, inputs, or other interactive elements, use a dialog, popover, or disclosure pattern instead of a tooltip.
- Tooltips should also appear on keyboard focus, not only on mouse hover.
- Tooltips should be dismissible, commonly with <kbd>Escape</kbd>.
- Do not invent keyboard behavior for custom widgets. Follow ARIA APG patterns.
- Common keyboard expectations:
  - <kbd>Tab</kbd> moves between focusable controls.
  - <kbd>Enter</kbd> or <kbd>Space</kbd> activates buttons.
  - <kbd>Escape</kbd> closes dialogs, menus, popovers, and tooltips when appropriate.
  - Arrow keys navigate within composite widgets such as menus, tabs, listboxes, grids, and command palettes.
- Destructive keyboard shortcuts should be documented and reversible when possible.
- Platform-specific shortcuts should respect user expectations.
  - Use <kbd>Cmd</kbd> on macOS.
  - Use <kbd>Ctrl</kbd> on Windows and Linux.
- Test with keyboard only.
- Test with at least one screen reader when shipping complex custom components.

## Layered UI

- Prefer native platform primitives when they fit the use case.
- Use `<dialog>` with `showModal()` for modal dialogs that require user attention and should make background content inert.
- Use `popover` for non-modal overlays such as lightweight menus, teaching bubbles, contextual panels, and simple floating UI.
- Do not use `popover` as a replacement for modal dialog behavior when the background must be inert.
- Custom modal implementations must handle focus containment, inert background content, Escape behavior, scroll locking, and focus restoration.
- Hidden or off-screen interactive regions should use `hidden`, `inert`, or conditional rendering so they are not reachable by keyboard or screen readers.
- Escape should close non-critical dialogs, menus, popovers, and overlays.
- Do not trap focus in a component without an obvious way to leave.
- Avoid stacking modals.

## Motion

- Respect `prefers-reduced-motion`.
- Keep frequent interaction animations short, subtle, and non-blocking.
- Longer transitions may be appropriate for larger page-level changes, but they should not block interaction.
- Avoid parallax, large zooming motion, and persistent background movement unless the experience strongly benefits from it.
- Prefer CSS transitions for simple state changes.
- Prefer transform and opacity for frequent animations.
- Avoid animating layout-heavy properties such as `width`, `height`, `top`, `left`, `margin`, and `box-shadow` in hot paths.
- Looping animations should pause when not visible on screen.
- Smooth scrolling should respect reduced motion.

```css
html {
  scroll-behavior: smooth;
  scroll-padding-top: var(--header-height);
}

@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }

  *,
  *::before,
  *::after {
    animation-duration: 0.001ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.001ms !important;
  }
}
```

- Do not use animation as the only way to communicate state changes.
- Motion should support understanding, not hide latency or distract from the task.

## Touch

- Hover states should be scoped to devices that support hover.

```css
@media (hover: hover) and (pointer: fine) {
  .button:hover {
    background: var(--button-hover-bg);
  }
}
```

- Font size for text inputs should generally not be smaller than `16px` to avoid unwanted iOS zoom on focus.
- Inputs should not autofocus on touch devices when autofocus would open the keyboard and cover important content.
- Touch targets should be large enough to activate reliably.
- Avoid placing destructive actions too close to common actions.
- Gestures should have non-gesture alternatives when the action is important.
- Swipe, drag, and long-press interactions should not be the only way to perform an important action.
- Avoid disabling pinch zoom globally.
- Disable the default iOS tap highlight only when replacing it with an appropriate visible pressed or focus state.
- For custom gestures, set the narrowest appropriate `touch-action`.
  - Use `pan-y` when the component handles horizontal gestures but vertical page scrolling should remain native.
  - Use `pan-x` when the component handles vertical gestures but horizontal page scrolling should remain native.
  - Use `pinch-zoom` when browser zoom should remain available.
  - Use `manipulation` for common direct manipulation where panning and pinch zoom should remain available.
  - Use `none` only when the component truly owns all touch gestures.

## Typography

- Use readable font sizes, sufficient contrast, comfortable line height, and tested font weights.
- Avoid global typography hacks unless they are tested across supported browsers and displays.
- Avoid typography choices that cause layout shift, such as changing font weight on hover.
- Fonts should be subset based on the content, alphabet, and relevant language(s).
- Use `font-display` intentionally.
- Avoid layout shift when web fonts load. Use fallback font metrics overrides when necessary.
- Use tabular numbers for values that should not visually shift, such as timers, prices, counters, and tables.

```css
.timer,
.price,
.numeric-cell {
  font-variant-numeric: tabular-nums;
}
```

- Avoid truncating important text without a way to read the full content.
- Avoid justified text for long passages unless hyphenation and spacing are carefully handled.
- Test typography in supported languages, especially when mixing Latin, CJK, emoji, and code.

## Performance

- Measure before optimizing.
- Optimize for interaction responsiveness, not only initial page load.
- Keep typing, clicking, dragging, scrolling, and route transitions responsive.
- Avoid long tasks during user interactions.
- Split expensive work, defer non-critical updates, and keep immediate visual feedback lightweight.
- Avoid blocking input feedback on network requests, analytics, logging, or large re-renders.
- Lazy-load non-critical images and iframes.
- Do not lazy-load the likely LCP image.
- Use explicit `width` and `height` on images and videos to reduce layout shift.
- Use responsive images with `srcset` and `sizes` when serving different viewport sizes or densities.
- Prefer modern image formats when appropriate, while keeping fallbacks if needed.
- Auto-playing too many videos can hurt performance and battery life, especially on mobile devices.
- Pause, lazy-load, or unmount off-screen videos when possible.
- Use virtualization for very long lists.
- Debounce or throttle expensive event handlers.
- Prefer `IntersectionObserver` over scroll listeners for visibility tracking.
- Prefer `ResizeObserver` over polling layout size.
- Use `will-change` only for measured performance issues.
- Avoid expensive framework re-renders in high-frequency interactions such as scroll, pointer move, wheel, drag, and resize.
- Treat hardware and network adaptation as progressive enhancement.
- Do not assume the Network Information API is available.
- Use browser performance tools, framework profilers, Web Vitals, and real-device testing.

## Resilience

- Preserve user work across errors, reloads, navigation, and temporary network loss when possible.
- Warn users before losing unsaved changes.
- Autosave should show clear save, saving, saved, failed, and offline states.
- Failed saves should be retryable and should not silently discard local changes.
- When conflicts happen, explain what changed and provide a safe way to resolve or recover.
- Destructive actions should be recoverable through undo, trash, version history, or confirmation depending on risk.
- Loading states should preserve layout when possible.
- Skeletons should resemble the final layout and should not be used for very short waits.
- Error messages should be human-readable and action-oriented.
- Avoid exposing raw backend errors to users.

## Security and Privacy UX

- Authentication redirects should happen on the server before the client loads when possible, to avoid janky URL changes and hydration flicker.
- Client-side redirects are acceptable for non-critical flows, but should not expose protected content before redirecting.
- Do not expose protected content before authentication or authorization checks complete.
- Sensitive actions should clearly explain what will happen before the user confirms.
- Password, token, API key, and recovery code fields should support copy, paste, and password managers.
- Do not block password managers unless there is a strong security reason.
- Avoid leaking sensitive information in URLs, page titles, analytics events, referrers, screenshots, or error messages.
- Use clear session expiration and re-authentication flows.
- Security-sensitive settings should show recent changes, active sessions, or recovery options when relevant.

## Internationalization

- Respect system preferences such as color scheme, contrast, reduced motion, and locale.
- Use locale-aware formatting for dates, times, numbers, currency, and pluralization.
- Avoid ambiguous dates. Prefer explicit dates when the context may be unclear.
- Use relative time only when exact time is not important, or provide both.
- Design layouts to tolerate text expansion and different word lengths.
- Avoid hardcoding sentence fragments that cannot be reordered in translation.
- Support pluralization, grammatical gender, and locale-specific formatting when relevant.
- Test with CJK, long Germanic text, emoji, code, mixed-direction text, and right-to-left languages when supported.
- Do not assume names, addresses, phone numbers, currencies, or date formats follow one locale’s structure.
- Avoid truncating translated text without a way to inspect the full value.

## Visual Design

- Layout should remain usable without custom fonts, images, or animations.
- Design should support both pointer and keyboard workflows.
- Primary actions should be visually clear, but not overused.
- Avoid multiple competing primary buttons in the same region.
- Keep destructive actions visually distinct from primary actions.
- Provide undo for destructive or easily reversible actions.
- Ask for confirmation for destructive, irreversible, expensive, or security-sensitive actions.
- Avoid confirmation dialogs for frequent low-risk actions when undo is possible.
- Prefer inline editing when users are modifying nearby content.
- Prefer modals for focused, blocking tasks.
- Use progressive disclosure for advanced or rarely used settings.
- Defaults should be safe, useful, and reversible.
- Avoid relying on visual polish alone.

## Contrast Preferences

- Test forced colors and high contrast modes.
- Do not rely on gradients, shadows, opacity, or subtle background colors as the only way to communicate state.
- Use system colors where appropriate for focus rings, selected states, and borders.
- Avoid hardcoded focus colors that disappear in forced colors mode.

```css
@media (forced-colors: active) {
  :focus-visible {
    outline: 2px solid Highlight;
  }

  button {
    border: 1px solid ButtonText;
  }
}
```

## Generated and Automated Content

- Clearly indicate when content is generated, suggested, or automatically transformed.
- Users should be able to review, edit, accept, reject, or undo generated changes.
- Do not silently overwrite user-authored content with generated content.
- Show confidence, source, or limitation information when it affects user trust or decision-making.
- Avoid presenting generated content as authoritative when it may be incomplete, stale, or uncertain.
- Generated actions that affect other users, billing, permissions, publishing, or deletion should require explicit confirmation.

## References

- [WCAG 2.2](https://www.w3.org/TR/WCAG22/)
- [WAI-ARIA 1.2](https://www.w3.org/TR/wai-aria-1.2/)
- [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
- [ARIA APG Tooltip Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/tooltip/)
- [MDN: `<dialog>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dialog)
- [MDN: `inert`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/inert)
- [MDN: `popover`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/popover)
- [MDN: `autocomplete`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/autocomplete)
- [MDN: `spellcheck`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/spellcheck)
- [MDN: `aria-disabled`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-disabled)
- [MDN: `prefers-reduced-motion`](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media/prefers-reduced-motion)
- [MDN: `forced-colors`](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@media/forced-colors)
- [MDN: `touch-action`](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/touch-action)
- [MDN: `will-change`](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/will-change)
- [MDN: `user-select`](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/user-select)
- [MDN: `outline`](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/outline)
- [MDN: Autoplay guide](https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Autoplay)
- [web.dev: Interaction to Next Paint](https://web.dev/articles/inp)
- [web.dev: INP as a Core Web Vital](https://web.dev/blog/inp-cwv-launch)
