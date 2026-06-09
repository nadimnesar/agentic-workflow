---
name: performance-optimization
description: Measure first, then optimize only what measurably matters. No intuition-based optimizations. Every optimization is preceded by a measurement that confirms the problem, and followed by a measurement that confirms the fix.
license: MIT
compatibility: opencode
metadata:
  phase: review
  agent: review
---

# Performance Optimization

## What I Do

I apply the discipline of measure-first optimization. I prevent the most common performance anti-pattern: spending time optimizing code that isn't slow, while missing the thing that actually is.

## The Prime Directive

> You cannot optimize what you have not measured. A performance change without a before/after measurement is a guess, not an improvement.

## The Optimization Protocol

### Step 1: Identify candidates (don't optimize yet)

Read the changed code with one question: _"Is this code on a hot path?"_

Hot path definition: code that runs:

- On every request to a high-traffic endpoint
- On every user interaction (keypress, scroll, resize)
- In a loop over a large dataset
- In a tight background process

Non-hot path (don't optimize):

- Code that runs once at startup
- Code that runs on rare user actions (settings page, export)
- Code that handles edge cases

### Step 2: Measure the baseline

For any candidate, measure before touching it:

**Backend:**

```bash
# Benchmark a specific function
# Use your test framework's benchmark mode, or:
hyperfine 'node -e "require(\"./src/fn\").run()"'
```

**Frontend:**

```
DevTools > Performance > Record > perform the action > Stop
- Note: total frame time, main thread blocking time
- Note: which functions appear in the flame chart
```

**Database:**

```sql
EXPLAIN ANALYZE SELECT ...
-- Note: actual rows scanned, actual time, index used/not used
```

Record the baseline number before any change.

### Step 3: Identify the actual bottleneck

A flame chart / profiler tells you where time is actually spent. Trust it over intuition.

Common places time actually goes (in order of frequency):

1. **Database queries**: N+1, missing indexes, full table scans, unneeded columns
2. **Network round-trips**: Too many sequential requests that could be batched or parallel
3. **Unnecessary re-computation**: Same expensive calculation done on every call
4. **Large payload size**: Sending more data than the client needs
5. **Blocking the main thread** (browser): Long tasks preventing interaction
6. **Memory pressure**: Large allocations causing GC pauses

Rare sources of actual bottlenecks:

- Algorithmic complexity (O(n²) vs O(n log n)) — only matters at significant scale
- String concatenation in tight loops — real but rare at typical data sizes
- Individual function call overhead

### Step 4: Apply the targeted fix

Match the fix to the diagnosis:

| Bottleneck                | Fix                                                       |
| ------------------------- | --------------------------------------------------------- |
| N+1 query                 | Batch with `WHERE id IN (...)` or eager load with JOIN    |
| Missing index             | Add index on the filtered/sorted column                   |
| Sequential async          | `Promise.all()` for independent operations                |
| Repeated computation      | Cache/memoize the result; compute once                    |
| Large payload             | Paginate, project only needed fields, add field selection |
| Re-render on stable props | `useMemo`, `useCallback`, `React.memo` (with measurement) |
| Long task in browser      | `setTimeout(fn, 0)` to yield, or move to Web Worker       |
| Large list render         | Virtualize (windowing) when list > ~200 items             |

### Step 5: Measure the improvement

After applying the fix, measure again with the same method:

```
Before: [X ms / Y ops/sec / Z rows scanned]
After:  [X' ms / Y' ops/sec / Z' rows scanned]
Change: [% improvement]
```

If the improvement is < 10% or not perceptible to users: the fix may not be worth its complexity cost.

### Step 6: Document the optimization

Add a code comment when the optimization isn't obvious:

```typescript
// Batching user lookups to avoid N+1 queries.
// Without this, loading 100 items results in 101 DB calls.
const users = await db.users.findMany({
  where: { id: { in: userIds } },
});
```

```typescript
// useMemo here because renderExpensiveChart() takes ~80ms on a 1000-item dataset.
// Measured with React DevTools Profiler on 2024-01 hardware.
const chart = useMemo(() => renderExpensiveChart(data), [data]);
```

## What You Do NOT Do

- Optimize code that isn't on a hot path
- Introduce complexity for < 10% gains
- Add caching without documenting the cache invalidation strategy
- Change an O(n) algorithm to O(1) "just in case" without a benchmark
- Micro-optimize (avoid function calls, manual loop unrolling) — the engine does this better than you

## Performance Budget Awareness

When reviewing, flag if the changed code could exceed these rough budgets:

| Context                   | Budget                                                  |
| ------------------------- | ------------------------------------------------------- |
| API response time (p95)   | < 200ms for simple reads, < 1s for complex operations   |
| Page load (TTI)           | < 3s on a mid-range device on 4G                        |
| User interaction response | < 100ms to feel instant                                 |
| Background job / batch    | Document expected time; add progress reporting if > 10s |

## Output: Performance Report

```markdown
## Performance Review

### Candidates identified

- [list.tsx] Item rendering loop — hot path (runs on every render)
- [api/search.ts] Text search handler — hot path (every search keystroke)

### Measurements taken

- list.tsx: 45ms render for 500 items (baseline)
- api/search.ts: EXPLAIN shows full table scan on `content` column

### Optimizations applied

- list.tsx: Added virtualization; 500-item render now 8ms (83% improvement)
- api/search.ts: Added GIN index on `content`; scan reduced from 50k rows to 200

### Not optimized (and why)

- [settings.ts] Loading is slow (300ms) but runs once per session — not worth the complexity
```
