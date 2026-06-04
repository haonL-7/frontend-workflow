---
name: frontend-workflow
description: Frontend development workflow engine. Breaks down UI features into implementable tasks, manages design-to-code handoffs, provides structured code review checklists, bug triage workflows, performance optimization guides, and deployment checklists. Trigger when planning frontend features, reviewing code, debugging UI bugs, optimizing performance, or preparing deployments.
---

# frontend-workflow — Frontend Development Workflow Engine

A structured workflow engine for planning, executing, and tracking frontend development tasks. Use this skill to break down complex UI features into manageable steps, track component implementation progress, and ensure quality handoffs.

## When to Use

- Breaking down a frontend feature into implementable tasks
- Tracking multi-component UI development progress
- Coordinating design-to-development handoffs
- Planning responsive/mobile-first implementations
- Managing frontend code review and QA workflows
- Setting up frontend CI/CD and deployment pipelines

## Workflow: Feature Breakdown

### Step 1 — Requirements Analysis

Before writing any code, clarify:
```
- What user problem does this solve?
- What are the acceptance criteria?
- What states does this UI have? (loading, empty, error, edge cases)
- What browsers/devices must be supported?
- Are there accessibility requirements (WCAG level)?
```

### Step 2 — Component Tree Design

Map the component hierarchy:
```
Page / Route
├── Layout
│   ├── Header
│   │   ├── Logo
│   │   ├── Navigation
│   │   └── UserMenu
│   ├── Main
│   │   ├── Sidebar (optional)
│   │   └── Content
│   │       ├── FeatureComponent
│   │       └── SubFeatureComponent
│   └── Footer
└── Modal (portaled)
```

### Step 3 — Task Breakdown Template

For each component, create a task card:

```yaml
Task:
  id: "feat-001"
  component: "UserMenu"
  priority: P1  # P1=blocking, P2=important, P3=nice-to-have
  status: todo  # todo → in-progress → review → done
  dependencies: ["feat-000: AuthContext"]
  estimated_hours: 2
  acceptance:
    - "Dropdown opens on click"
    - "Keyboard navigation (↑↓ Enter Esc)"
    - "Closes on outside click"
    - "ARIA labels for screen readers"
    - "Responsive: collapses to icon on mobile"
```

### Step 4 — Implementation Order

Follow the dependency graph:
1. **Foundation** — theme, design tokens, global styles, layout
2. **Atomic Components** — Button, Input, Badge, Icon (no dependencies)
3. **Composite Components** — Form, Modal, Card (depends on atomic)
4. **Feature Components** — specific business features (depends on composite)
5. **Pages / Routes** — assemble components, add data fetching
6. **Polish** — animations, transitions, loading states, error boundaries

## Workflow: Design-to-Code Handoff

### Design Spec Extraction
- Extract: colors, typography, spacing, breakpoints
- Map design tokens to CSS custom properties or Tailwind config
- Identify reusable patterns vs one-off styles

### Component Audit
- List all distinct UI elements in the design
- Group by reusability
- Flag potential a11y issues early (contrast, focus states, labels)

### Implementation Checklist
```
□ Design tokens → CSS variables / Tailwind config
□ Global styles (reset, base typography, scrollbar)
□ Layout component (responsive grid/flex)
□ Atomic components (Button, Input, Select, etc.)
□ Accessibility audit (axe-core / Lighthouse)
□ Responsive testing (mobile → tablet → desktop)
□ Cross-browser testing
□ Performance check (Lighthouse > 90)
```

## Workflow: Frontend Code Review Checklist

### Code Quality
- [ ] Components are single-responsibility
- [ ] No props drilling beyond 2 levels
- [ ] Custom hooks extracted where logic is reused
- [ ] TypeScript types are explicit (no `any`)
- [ ] No inline styles (use CSS modules / Tailwind / styled)
- [ ] No hardcoded strings (use i18n or constants)

### Performance
- [ ] No unnecessary re-renders (React.memo, useMemo, useCallback)
- [ ] Images are lazy-loaded and optimized
- [ ] Bundle size is checked (no heavy imports)
- [ ] No memory leaks (cleanup in useEffect)
- [ ] API calls are debounced where appropriate

### Accessibility
- [ ] Semantic HTML used (`<button>` not `<div onClick>`)
- [ ] All images have alt text
- [ ] Form inputs have labels
- [ ] Color contrast meets WCAG AA (4.5:1)
- [ ] Focus management works (modals trap focus)
- [ ] Screen reader tested

### Testing
- [ ] Critical paths have unit tests
- [ ] User interactions tested (click, type, navigate)
- [ ] Error states tested
- [ ] Loading states tested

## Workflow: Frontend Bug Triage

### Bug Report Template
```yaml
Bug:
  id: "bug-042"
  severity: S1  # S1=crash, S2=broken-feature, S3=visual-glitch, S4=enhancement
  component: "UserMenu"
  environment: "Chrome 125 / macOS / 1920x1080"
  reproduction_steps:
    - "1. Log in as admin"
    - "2. Click user menu icon"
    - "3. Select 'Settings'"
  expected: "Navigates to /settings"
  actual: "Dropdown closes, no navigation"
  console_errors: "TypeError: Cannot read property 'name' of undefined"
```

### Debug Workflow
1. **Reproduce** — confirm the bug with exact steps
2. **Isolate** — find the smallest reproduction case
3. **Diagnose** — check console, network, React DevTools
4. **Fix** — minimal change that addresses root cause
5. **Test** — verify fix + check for regressions
6. **Document** — add a test case to prevent recurrence

## Workflow: Frontend Performance Optimization

### Audit Steps
1. Run Lighthouse audit (Performance tab)
2. Check bundle size (`npm run build -- --analyze` or `npx vite-bundle-visualizer`)
3. Profile with React DevTools Profiler
4. Check network waterfall for slow requests

### Common Fixes
| Problem | Solution |
|---------|----------|
| Large bundle | Code splitting: `React.lazy()` + `Suspense` |
| Slow re-renders | `React.memo`, `useMemo`, `useCallback` |
| Heavy images | Next.js `<Image>`, lazy loading, WebP format |
| Slow API calls | Debounce search, cache responses, optimistic UI |
| CLS (layout shift) | Set explicit width/height on images |
| Slow FCP | Critical CSS inline, defer non-critical JS |
| Large deps | Tree-shake imports, use lighter alternatives |

## Workflow: Deployment Checklist

### Pre-Deploy
- [ ] All tests passing (`npm test`)
- [ ] Build succeeds (`npm run build`)
- [ ] Lint passes (`npm run lint`)
- [ ] Type check passes (`npx tsc --noEmit`)
- [ ] Environment variables configured
- [ ] .env.example updated

### Post-Deploy
- [ ] Smoke test critical paths
- [ ] Check error monitoring (Sentry / LogRocket)
- [ ] Check analytics
- [ ] Monitor performance metrics
- [ ] Verify CDN cache is purged (if applicable)

## Quick Patterns

### Feature Flag Pattern
```tsx
const useFeatureFlag = (flagName: string): boolean => {
  // Replace with your feature flag service
  return process.env[`NEXT_PUBLIC_FF_${flagName}`] === 'true';
};

// Usage
function MyComponent() {
  const showNewUI = useFeatureFlag('NEW_DASHBOARD');
  if (showNewUI) return <NewDashboard />;
  return <OldDashboard />;
}
```

### Error Boundary Pattern
```tsx
class ErrorBoundary extends React.Component<
  { children: React.ReactNode; fallback?: React.ReactNode },
  { hasError: boolean; error?: Error }
> {
  constructor(props: { children: React.ReactNode; fallback?: React.ReactNode }) {
    super(props);
    this.state = { hasError: false };
  }
  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }
  componentDidCatch(error: Error, info: React.ErrorInfo) {
    console.error('ErrorBoundary caught:', error, info);
  }
  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? <div>Something went wrong.</div>;
    }
    return this.props.children;
  }
}
```

### Loading Skeleton Pattern
```tsx
function Skeleton({ className = '' }: { className?: string }) {
  return (
    <div className={`animate-pulse rounded bg-gray-200 ${className}`} />
  );
}

function CardSkeleton() {
  return (
    <div className="space-y-3 rounded-lg border p-4">
      <Skeleton className="h-4 w-3/4" />
      <Skeleton className="h-4 w-1/2" />
      <Skeleton className="h-20 w-full" />
    </div>
  );
}
```
