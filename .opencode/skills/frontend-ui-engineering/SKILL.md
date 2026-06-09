---
name: frontend-ui-engineering
description: Build production-quality UI with semantic HTML, full accessibility, responsive layout, and no runtime errors. Every component is keyboard-navigable, screen-reader compatible, and works at all viewports before it ships.
license: MIT
compatibility: opencode
metadata:
  phase: build
  agent: build
---

# Frontend UI Engineering

## What I Do

I enforce the baseline quality bar for every UI component: semantic HTML, accessibility, responsiveness, error handling, and no console noise. "It looks right" is not the bar. Production-ready is.

## When to Use Me

Use me for every slice that creates or modifies a UI component, page, form, or interactive element.

## The UI Engineering Checklist

Run this checklist for every UI slice before marking it complete.

### 1. Semantic HTML

- [ ] Uses the correct HTML element for its purpose (button, not div; nav, not div; article, not div)
- [ ] Headings are in logical order (h1 → h2 → h3, not chosen for appearance)
- [ ] Form inputs have associated `<label>` elements (not placeholder as label)
- [ ] Lists use `<ul>/<ol>/<li>`, not div-with-padding
- [ ] Tables have `<thead>`, `<th scope>`, and `<caption>` if used for data
- [ ] Images have meaningful `alt` text (not "image" or filename)
- [ ] Icons that convey meaning have `aria-label` or accompanying visible text

### 2. Accessibility (A11y)

**Keyboard navigation:**
- [ ] All interactive elements are reachable by Tab
- [ ] Logical tab order (matches visual order, no Tab traps)
- [ ] Escape closes modals, dropdowns, and popovers
- [ ] Enter/Space activates buttons (not just click)
- [ ] Arrow keys navigate within composite widgets (menus, tabs, sliders)

**Screen reader:**
- [ ] Page landmarks present: `<header>`, `<main>`, `<nav>`, `<footer>`
- [ ] Dynamic content updates announced: use `aria-live` for status messages
- [ ] Modal/dialog traps focus inside and restores on close
- [ ] Error messages are associated with inputs via `aria-describedby`
- [ ] Loading states announced: `aria-busy`, loading spinner has `role="status"`

**Color and contrast:**
- [ ] Text contrast ratio ≥ 4.5:1 (normal text), ≥ 3:1 (large text)
- [ ] State (error, success, disabled) communicated by more than color alone

**Motion:**
- [ ] Animations respect `prefers-reduced-motion`

### 3. Responsive Design

- [ ] Works at 320px (small mobile), 768px (tablet), 1280px (desktop)
- [ ] No horizontal scroll at any breakpoint
- [ ] Touch targets ≥ 44×44px on mobile
- [ ] No fixed pixel widths on containers that should be fluid
- [ ] Images don't overflow their containers

### 4. Error and Loading States

Every data-fetching component must implement:
- [ ] **Loading state**: skeleton, spinner, or progressive disclosure — not a blank render
- [ ] **Error state**: user-readable message, not raw error object; with retry if applicable
- [ ] **Empty state**: explicit message when list/data is empty — not blank
- [ ] **Optimistic updates**: if used, must be rolled back cleanly on failure

### 5. Performance Basics

- [ ] No unnecessary re-renders (in React: stable references for callbacks and derived values)
- [ ] Images use correct format and appropriate size for context
- [ ] No blocking operations in the render path
- [ ] Event listeners cleaned up in component teardown

### 6. No Console Noise

Before marking the slice complete:
- [ ] No console.error in normal operation
- [ ] No unhandled promise rejections
- [ ] No React key warnings or prop-type warnings
- [ ] No "Can't update state on an unmounted component" warnings

## Coding Patterns

### Component structure (React example)
```tsx
// 1. Props are typed and documented
type ButtonProps = {
  label: string;          // visible text AND accessible name
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  isLoading?: boolean;
};

// 2. Accessibility built in, not added later
export function Button({ label, onClick, variant = 'primary', disabled, isLoading }: ButtonProps) {
  return (
    <button
      type="button"
      onClick={onClick}
      disabled={disabled || isLoading}
      aria-disabled={disabled || isLoading}
      aria-busy={isLoading}
      className={`btn btn-${variant}`}
    >
      {isLoading ? <span aria-hidden="true">...</span> : null}
      <span>{label}</span>
    </button>
  );
}
```

### Form pattern
```tsx
// Always: label → input → error message — in that DOM order
<div>
  <label htmlFor="email">Email address</label>
  <input
    id="email"
    type="email"
    aria-describedby={error ? "email-error" : undefined}
    aria-invalid={!!error}
  />
  {error && <span id="email-error" role="alert">{error}</span>}
</div>
```

## What You Do NOT Do

- Use `onClick` on a `<div>` when `<button>` is correct
- Use placeholder text as the only label for an input
- Hard-code `z-index: 9999` as a solution to stacking issues
- Ship with console errors and plan to "fix them later"
- Build mobile as an afterthought — design mobile-first or at minimum simultaneously

## Verification

Before reporting slice complete, open the UI and:
1. Tab through all interactive elements — confirm reachability and order
2. Test with screen reader (or axe DevTools) — check for violations
3. Resize to 320px — check for overflow or usability issues
4. Test the error state — trigger it, confirm the message is clear
