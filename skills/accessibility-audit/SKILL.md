---
name: accessibility-audit
description: Audits and suggests fixes for accessibility (WCAG, keyboard, screen readers). Use when checking a11y, before release, or when the user asks for accessibility review.
---

# Accessibility Audit

Review UI and markup for accessibility and suggest concrete fixes aligned with WCAG so all users can perceive, operate, and understand the interface.

## When to Activate

- User asks for "accessibility", "a11y", "WCAG", or "screen reader support"
- Before shipping a new page or component
- Reviewing forms, modals, or interactive UI
- After a design or UI change that affects interaction or content

## Work Process

1. **Define scope** — Page, component, or flow to review (e.g. login form, report table, modal dialog).
2. **Check each area** — Semantic HTML, keyboard, labels and names, color and contrast, dynamic content, motion. Use the checklists below.
3. **Document findings** — Location (component/file or element type), WCAG criterion or principle, issue, impact, and recommended fix. Severity: critical / major / minor.
4. **Suggest fixes** — Code snippet or attribute change where possible. Prefer native HTML and correct ARIA over custom widgets when they suffice.
5. **Recommend follow-up** — Suggest automated tools (e.g. axe, Lighthouse) and manual testing (keyboard only, screen reader) for broader coverage. Do not claim full WCAG compliance from a single review.

## Focus Areas and Checklist

### 1. Semantic HTML and structure

- [ ] One `<h1>` per page; heading levels in order (no skipping 2 → 4).
- [ ] Landmarks: `<main>`, `<nav>`, `<aside>`, `<footer>` where appropriate.
- [ ] Lists use `<ul>`/`<ol>`/`<li>`; tables use `<th>`, `scope`, and caption if applicable.
- [ ] Buttons are `<button>` or `<input type="submit">`; links are `<a href="...">`. Do not use `<div onclick>` for actions without role, keyboard, and focus.
- [ ] Form controls have correct type and grouping (`<fieldset>`, `<legend>` for groups).

### 2. Keyboard

- [ ] All interactive elements are focusable (no `tabindex="-1"` on buttons/links unless managed for modal).
- [ ] Focus order is logical (matches visual order or is explicitly managed).
- [ ] Visible focus indicator (outline or custom style); not removed with `outline: none` without replacement.
- [ ] No keyboard trap: user can tab out of modals and menus; Escape closes where expected.
- [ ] Custom widgets (tabs, accordions, menus) operable with keyboard (Enter/Space to activate, Arrow keys if applicable).

### 3. Labels and names

- [ ] Every form input has a visible `<label>` or `aria-label`; labels are associated (e.g. `for`/`id` or wrapping).
- [ ] Buttons and links have clear, unique names (text content or `aria-label`). No "Click here" or "Read more" without context.
- [ ] Images: meaningful images have `alt` describing content; decorative images have `alt=""` or `role="presentation"`.
- [ ] Iframe and embedded content have `title` or `aria-label`.

### 4. Color and contrast

- [ ] Text meets contrast ratio (e.g. 4.5:1 for normal text, 3:1 for large; 3:1 for UI components). Check against background.
- [ ] Information is not conveyed by color alone (e.g. required fields, errors, status). Use icon, text, or pattern as well.
- [ ] Focus indicator is visible and not reliant only on color change.

### 5. Dynamic content and focus

- [ ] Content that appears or updates (e.g. toast, live results) is announced: `aria-live="polite"` or `aria-live="assertive"` as appropriate.
- [ ] Modals and dialogs: focus moves into the modal when opened; focus is trapped inside; focus returns to trigger when closed; first focusable element or explicit `autoFocus` per pattern.
- [ ] State is communicated: expanded/collapsed (`aria-expanded`), selected (`aria-selected`), current (`aria-current`), disabled (`disabled` or `aria-disabled`).

### 6. Motion

- [ ] If the project supports it: respect `prefers-reduced-motion` (disable or reduce animation). Optional but recommended for vestibular sensitivity.

## Output Format

For each issue:

```markdown
**[Location: file or component/element]**
- **Issue:** [What is wrong.]
- **WCAG / principle:** [Criterion or principle, e.g. 1.3.1 Info and Relationships, 2.1.1 Keyboard.]
- **Impact:** [Who is affected and how.]
- **Recommendation:** [Concrete fix: code or attribute.]
- **Severity:** Critical | Major | Minor
```

Summary: "Reviewed: [scope]. Found X critical, Y major, Z minor. Recommend automated scan (axe/Lighthouse) and keyboard/screen reader testing."

## Severity

- **Critical** — Blocks core task (e.g. cannot submit form, cannot navigate with keyboard, no labels on required fields). Fix before release.
- **Major** — Significant barrier (e.g. poor contrast, missing headings, confusing order). Fix soon.
- **Minor** — Improvement (e.g. redundant label, minor contrast). Backlog or fix when touching the component.

## Good vs bad examples

**Button:**
```html
<!-- BAD -->
<div onclick="submit()">Submit</div>

<!-- GOOD -->
<button type="submit">Submit</button>
```

**Image:**
```html
<!-- Decorative -->
<img src="decoration.svg" alt="" role="presentation">

<!-- Meaningful -->
<img src="chart.png" alt="Bar chart showing revenue up 20% in Q4">
```

**Form:**
```html
<!-- BAD -->
<input type="email" placeholder="Email">

<!-- GOOD -->
<label for="email">Email</label>
<input id="email" type="email" placeholder="you@example.com">
```

## Rules

- **Do not claim full compliance** — Frame as "issues found in reviewed scope." Recommend automated and manual testing for full coverage.
- **Prefer native HTML** — Use `<button>`, `<label>`, `<main>`, etc. Use ARIA when semantics cannot be expressed with HTML (e.g. `aria-expanded` on a custom accordion).
- **Component libraries** — If the project uses one (e.g. Radix, MUI), note library a11y patterns (focus trap, roles) and ensure they are used correctly rather than reimplementing.

## Checklist (before finishing)

- [ ] Scope clearly defined
- [ ] Each finding has location, issue, WCAG reference, recommendation, severity
- [ ] At least semantic structure, keyboard, and labels/names covered
- [ ] Follow-up (automated + manual testing) suggested
- [ ] No "fully WCAG 2.1 AAA compliant" without full audit

## Anti-patterns

| Anti-pattern | Better approach |
|--------------|-----------------|
| Only checking color contrast | Cover keyboard, labels, structure, and dynamic content |
| Suggesting "add aria-label" everywhere | Prefer visible labels and semantic HTML; use ARIA when necessary |
| Ignoring focus order in modals | Document focus trap and return focus behavior |
| Claiming compliance after code review only | Recommend axe/Lighthouse and real assistive tech testing |
