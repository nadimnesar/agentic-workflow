---
name: browser-testing-with-devtools
description: Use Chrome DevTools MCP for runtime verification of UI — check console errors, network requests, accessibility violations, layout at multiple viewports, and performance. Automated tests alone are not enough for UI; runtime verification catches what unit tests miss.
license: MIT
compatibility: opencode
metadata:
  phase: test
  agent: test
---

# Browser Testing with DevTools

## What I Do

I verify that UI components actually work in the browser at runtime — not just that unit tests pass. I catch issues that only appear in a real browser: console errors, network failures, layout breaks, accessibility violations, and runtime exceptions that tests don't simulate.

## When to Use Me

Use me for every UI slice after unit/integration tests pass. I am a complement to TDD, not a replacement.

## The Browser Verification Protocol

### Step 1: Open with DevTools connected

If the Chrome DevTools MCP server is available, connect to the running app:
```
- Open the app at its local dev URL
- Open DevTools (F12 / Cmd+Option+I)
- Clear the console before testing (to avoid seeing pre-existing noise)
```

### Step 2: Console audit

Before interacting with anything:
- [ ] No errors in the console at initial load
- [ ] No warnings about missing required props, invalid prop types, or key issues (React)
- [ ] No deprecation warnings from libraries you're using

After each interaction:
- [ ] No new errors appear in the console
- [ ] Network requests succeed (no failed fetches shown as errors)

**Rule**: Any console.error in normal operation is a bug. Silence it by fixing the cause, not by `console.error = () => {}`.

### Step 3: Network panel

For components that fetch data:
- [ ] Expected network requests are made (check URL, method, headers)
- [ ] No unexpected extra requests (N+1 is visible here as multiple identical requests)
- [ ] Responses have expected status codes
- [ ] No requests to wrong environments (staging URL in production build)
- [ ] Auth tokens are in headers — not in URLs (URLs get logged)

Test the error state:
- Use the Network panel to throttle or block a request
- Confirm the UI shows a useful error state (not a blank screen or raw JSON)

### Step 4: Accessibility audit (axe)

Run the axe accessibility checker (DevTools > Lighthouse, or axe DevTools extension):
- [ ] No critical violations
- [ ] No serious violations
- [ ] Review moderate violations — not all are worth fixing, but document them

Common violations to catch:
- Buttons without accessible names
- Images without alt text
- Form inputs without labels
- Insufficient color contrast
- Missing landmark regions
- Focus not visible on keyboard navigation

### Step 5: Viewport testing

Test at three widths minimum:
- **320px** (small mobile — this breaks most layouts that weren't designed mobile-first)
- **768px** (tablet — often neglected)
- **1280px** (standard desktop)

Check:
- [ ] No horizontal scroll at any viewport
- [ ] Content remains readable (no overflow, no text truncation that hides meaning)
- [ ] Interactive elements remain usable (touch targets large enough on mobile)
- [ ] Navigation is usable at all viewports

Use DevTools Device Toolbar (Cmd+Shift+M) to quickly toggle viewports.

### Step 6: Keyboard navigation test

Tab through the component with mouse disconnected:
- [ ] All interactive elements reachable
- [ ] Focus indicator visible at every step
- [ ] Logical tab order (matches reading/visual order)
- [ ] Modals and dropdowns trap focus correctly
- [ ] Escape closes overlays
- [ ] Enter/Space activates buttons

### Step 7: Performance sanity check

For components that render lists, load heavy data, or have complex interactions:
- Open DevTools > Performance
- Record a representative interaction
- Check: no jank (frame drops below 60fps during animation/scroll)
- Check: no long tasks > 50ms blocking the main thread on interaction

For React apps:
- Install React DevTools
- Use Profiler to confirm no unnecessary re-renders on stable props
- Check: components that should be memoized are memoized

### Step 8: Document findings

```markdown
## Browser Verification: [Component/Slice Name]

Console: ✓ clean / ✗ issues: [list]
Network: ✓ clean / ✗ issues: [list]
Accessibility (axe): ✓ no violations / ✗ violations: [list]
Responsive:
  - 320px: ✓ / ✗ [issue]
  - 768px: ✓ / ✗ [issue]
  - 1280px: ✓ / ✗ [issue]
Keyboard nav: ✓ / ✗ [issue]
Performance: ✓ no issues / ✗ [issue]

Action items: [list any issues found, with severity]
```

## When DevTools MCP Is Not Available

If the browser MCP is not connected, perform these checks manually:
1. Run `npx axe [URL]` from CLI if available
2. Run Lighthouse in the browser
3. Manually test keyboard navigation
4. Check the browser console during manual testing
5. Use responsive design mode in any modern browser

Always record what was and wasn't verified.
