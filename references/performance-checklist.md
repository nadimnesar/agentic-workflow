# Performance Checklist Reference

Quick reference for performance optimization. Use alongside `performance-optimization` skill.

## Core Web Vitals Targets

| Metric | Good | Needs Improvement | Poor |
|---|---|---|---|
| LCP | <= 2.5s | <= 4.0s | > 4.0s |
| INP | <= 200ms | <= 500ms | > 500ms |
| CLS | <= 0.1 | <= 0.25 | > 0.25 |

## Frontend Checklist

- [ ] Images optimized (responsive sizes, WebP/AVIF, lazy loading below fold)
- [ ] Bundle size within budget (< 200KB gzipped initial JS)
- [ ] Code splitting at route level
- [ ] No render-blocking resources in critical path
- [ ] Fonts self-hosted or preloaded with `font-display: swap`
- [ ] Long tasks (< 50ms) on main thread minimized
- [ ] No layout shifts (set explicit dimensions on images/embeds)
- [ ] Service worker for caching and offline support

## Backend Checklist

- [ ] No N+1 query patterns
- [ ] List endpoints paginated with limits
- [ ] Database queries have appropriate indexes
- [ ] Caching for frequently-read, rarely-changed data
- [ ] Response compression enabled (gzip/brotli)
- [ ] Connection pooling configured
- [ ] API response times < 200ms (p95)

## Performance Budget

```
JavaScript (initial):  < 200KB gzipped
CSS:                  < 50KB gzipped
Images (above fold):  < 200KB each
Fonts:                < 100KB total
API response (p95):   < 200ms
Lighthouse score:     >= 90
```

## Measurement Tools

```bash
# Bundle analysis
npx vite-bundle-analyzer    # Vite
npx source-map-explorer     # webpack

# Lighthouse
npx lhci autorun            # CI

# Web Vitals (RUM)
import { onLCP, onINP, onCLS } from 'web-vitals';

# API profiling
console.time('query');
const result = await db.query(...);
console.timeEnd('query');
```
