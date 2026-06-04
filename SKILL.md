---
name: frontend-workflow
description: Multi-site series development workflow engine. New-site onboarding, cross-site consistency audits, shared-component versioning, feature breakdown, code review, bug triage, performance, and deployment — all series-aware. MANUAL TRIGGER ONLY — invoke explicitly via /skill frontend-workflow. Do NOT auto-trigger.
trigger: manual
---

# frontend-workflow — Multi-Site Series Execution Engine

This is NOT a general-purpose checklist. It is the operating manual for building and maintaining
a SERIES of websites that share components, design tokens, and infrastructure. Every workflow
below assumes you are running multiple sites, not one.

---

## 1. New Site Onboarding

When starting a new site in the series:

### Step 1 — Bootstrap
```
□ Clone the series starter template (NOT create-next-app from scratch)
□ Run: git clone <series-starter> <new-site-name> && cd <new-site-name>
□ Run: npm install
□ Update site name in package.json, .env.example, metadata
```

### Step 2 — Design Token Sync
```
□ Verify tokens.css imports the shared token file from the series monorepo / shared package
□ If this site has a unique primary color: override --color-primary in site-local tokens
□ All OTHER tokens must remain inherited — do NOT re-define spacing, radius, shadows locally
□ Run: npm run tokens:validate (or visual check: does the starter look right?)
```

### Step 3 — Site Registry
Add the new site to the series site registry (a markdown table or JSON file in the shared repo):

```yaml
- name: "new-site"
  url: "https://new-site.example.com"
  repo: "https://github.com/<org>/new-site"
  primary_color: "#xxxxxx"
  description: "What this site does"
  status: "in-development"  # in-development → staging → production
  created: "YYYY-MM-DD"
```

### Step 4 — Shared Component Verification
```
□ Run the shared component test suite against this site: npm test -- --scope=@series/ui
□ Verify all mandatory components render: Modal, Form, ErrorBlock, Skeleton, EmptyState
□ If any Mandatory component breaks: fix the SHARED component, not this site's override
```

### Step 5 — First Deploy
```
□ Build: npm run build
□ Lint: npm run lint
□ Type-check: npx tsc --noEmit
□ Deploy to staging
□ Run cross-site smoke tests (check shared components render identically vs other sites)
□ Promote to production when ready
```

---

## 2. Feature Breakdown (Series-Aware)

For each feature, first ask: **does this belong in the shared library or this specific site?**

```
Decision tree:
  Is this feature identical across 2+ sites?
    YES → Build it in @series/ui (shared), then consume in each site
    NO  → Build it in the specific site

  Does this feature have the SAME behavior but DIFFERENT data per site?
    YES → Build the component in @series/ui, pass data as props/config
    NO  → Build separately per site
```

### Task Card Template (Series-Aware)
```yaml
Task:
  id: "feat-042"
  title: "User dashboard widget"
  location: "@series/ui"  # or "site-name"
  sites_affected: ["site-a", "site-b", "site-c"]
  priority: P1
  status: todo
  depends_on: ["feat-041: Auth context"]
  consistency_check:
    - "Modal behavior matches across all affected sites"
    - "Error states use shared ErrorBlock"
    - "Loading states use shared Skeleton"
    - "Tokens reference CSS variables, not hardcoded values"
```

---

## 3. Cross-Site Consistency Audit

Run this before any site goes to production, or when a shared component changes.

### Audit Checklist
```
□ Modal: Esc closes, overlay click closes, focus trap works, body scroll locked
    → Test on: (list all sites)
    → Command: npx playwright test --grep "modal-consistency"

□ Form: inputs are controlled, validation errors appear identically, submit loading state works
    → Test on: (list all sites)
    → Command: npx playwright test --grep "form-consistency"

□ ErrorBlock: looks identical, retry button works, alert role present
    → Visual diff vs reference screenshots
    → Command: npx playwright test --grep "error-visual"

□ Skeleton: same animation, same shape, no layout shift on data arrival
    → Visual diff vs reference screenshots

□ EmptyState: same layout (icon + title + desc + action), same spacing
    → Visual diff vs reference screenshots

□ Design tokens: no hardcoded colors/px found via grep
    → Command: grep -rn '#[0-9a-fA-F]\{3,6\}' src/ --include='*.tsx' | grep -v 'tokens'
    → Command: grep -rn '[0-9]\+px' src/ --include='*.tsx' | grep -v 'tokens'

□ Transitions: 150ms/200ms/300ms timing consistent
    → Command: grep -rn 'transition' src/ --include='*.css'

□ Bundle: no site accidentally bundles another site's code
    → Command: npm run build -- --analyze
```

---

## 4. Shared Component Versioning

When changing a shared component (`@series/ui`):

```
1. Branch: git checkout -b feat/component-change
2. Change the component in @series/ui
3. Run the shared-component test suite
4. Run cross-site E2E tests against ALL sites that use this component
5. Bump version: npm version patch|minor|major (semver — see below)
6. Update CHANGELOG.md with migration notes if breaking
7. PR review → merge
8. Each site runs: npm update @series/ui && npm test
```

### Version Bump Rules
| Change | Bump |
|--------|------|
| Bug fix, same API | `patch` (1.0.0 → 1.0.1) |
| New prop, backwards-compatible | `minor` (1.0.0 → 1.1.0) |
| Prop removed, behavior changed | `major` (1.0.0 → 2.0.0) |

---

## 5. Code Review (Series-Aware)

Standard review items PLUS these series-specific checks:

### Series-Specific Review Questions
```
□ Is this component in the right place? (@series/ui vs site-local)
□ If in @series/ui: does it work identically across ALL sites?
□ If site-local: could it be promoted to @series/ui later?
□ Are design tokens used (not hardcoded values)?
□ Are Mandatory components used where applicable (Modal, Form, ErrorBlock, etc)?
□ Does this change break any other site's visual regression tests?
□ Is the Site Registry updated if this is a new site?
```

---

## 6. Bug Triage (Series-Aware)

### Severity in Multi-Site Context
| Severity | Definition |
|----------|-----------|
| S1 | Bug affects ALL sites (shared component crash) |
| S2 | Bug affects 2+ sites |
| S3 | Bug affects a single site |
| S4 | Visual glitch on a single site |

### Bug Report Template
```yaml
Bug:
  id: "bug-042"
  severity: S2
  component: "@series/ui/Modal"
  sites_affected: ["site-a", "site-b"]
  sites_unaffected: ["site-c"]
  environment: "Chrome 125 / macOS / 1920x1080"
  reproduction_steps:
    - "1. Open modal on site-a"
    - "2. Press Tab repeatedly"
  expected: "Focus stays trapped inside modal"
  actual: "Focus escapes to browser chrome on 5th Tab press"
  root_cause_hypothesis: "Focus trap selector missing a new button variant"
```

---

## 7. Performance Optimization (Series-Aware)

### Shared Optimization Impact
When you optimize a shared component, the benefit multiplies across all sites.
Prioritize shared-component optimizations over site-specific ones.

### Audit Commands (Concrete)
```
# Lighthouse (per site)
npx lighthouse https://site-a.example.com --output html --output-path reports/site-a.html
npx lighthouse https://site-b.example.com --output html --output-path reports/site-b.html

# Bundle analysis (per site)
npm -C apps/site-a run build -- --analyze
npm -C apps/site-b run build -- --analyze

# Shared component bundle impact
npx vite-bundle-visualizer --scope=@series/ui

# Cross-site visual regression
npx playwright test --config=visual-regression.config.ts

# Unused CSS across all sites
npx purgecss --css '**/*.css' --content '**/*.tsx' --output dist/
```

---

## 8. Deployment (Series-Aware)

### Pre-Deploy (any site)
```
□ Build passes: npm run build
□ Lint passes: npm run lint
□ Type-check passes: npx tsc --noEmit
□ Tests pass: npm test
□ Cross-site E2E passes (if shared components changed): npm run test:e2e -- --all-sites
□ Visual regression diff reviewed and approved
□ Site Registry status updated (in-development → staging → production)
```

### Post-Deploy (any site)
```
□ Smoke test critical paths on THIS site
□ Smoke test ONE critical path on OTHER sites (shared infra might have changed)
□ Check error monitoring (Sentry / LogRocket) across all sites
□ Verify CDN cache purged
□ Update Site Registry: status = "production", deployed_at = "YYYY-MM-DD"
```

### Rollback Decision Matrix
```
Did the deploy change a SHARED component?
  YES → Does it break N sites?
    1 site broken    → Fix forward (hotfix to @series/ui)
    2+ sites broken  → ROLLBACK all sites that auto-updated, fix forward
    0 sites broken   → Monitor, no action

  NO (site-local change only) →
    Bug found? → Rollback THIS site only, fix forward
```

---

## 9. Site Registry (Living Document)

Maintain a registry of all sites in the series. Keep this in the shared monorepo root
as `SITES.md` or `sites.json`.

```yaml
series:
  name: "<series-name>"
  shared_ui_version: "1.2.3"
  sites:
    - name: "site-a"
      url: "https://a.example.com"
      repo: "https://github.com/<org>/site-a"
      primary_color: "#3B82F6"
      status: "production"
      deployed_at: "2026-06-01"
      notes: "Main marketing site"

    - name: "site-b"
      url: "https://b.example.com"
      repo: "https://github.com/<org>/site-b"
      primary_color: "#10B981"
      status: "staging"
      deployed_at: "2026-06-04"
      notes: "Docs & support portal"

    - name: "site-c"
      url: ""
      repo: "https://github.com/<org>/site-c"
      primary_color: "#F59E0B"
      status: "in-development"
      deployed_at: ""
      notes: "Admin dashboard — not yet deployed"
```

---

## Quick Patterns

### Feature Flag (shared across series)
```tsx
// In @series/ui
const useFeatureFlag = (flagName: string): boolean => {
  const searchParams = typeof window !== 'undefined'
    ? new URLSearchParams(window.location.search)
    : null;
  // 1. URL param override (for testing)
  if (searchParams?.has(`ff_${flagName}`)) {
    return searchParams.get(`ff_${flagName}`) === '1';
  }
  // 2. Environment variable
  if (typeof process !== 'undefined' && process.env[`NEXT_PUBLIC_FF_${flagName}`]) {
    return process.env[`NEXT_PUBLIC_FF_${flagName}`] === 'true';
  }
  // 3. Default off
  return false;
};
```

### Error Boundary (every route)
```tsx
import { Component, type ReactNode, type ErrorInfo } from 'react';

interface ErrorBoundaryProps {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, info: ErrorInfo) => void;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  state: ErrorBoundaryState = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: ErrorInfo) {
    this.props.onError?.(error, info);
  }

  render() {
    if (this.state.hasError) {
      return (
        this.props.fallback ?? (
          <div className="flex min-h-[60vh] items-center justify-center p-8">
            <div className="max-w-md text-center">
              <h2 className="mb-2 text-xl font-semibold">Something went wrong</h2>
              <p className="mb-4 text-sm text-[var(--color-text-muted)]">
                {this.state.error?.message ?? 'An unexpected error occurred'}
              </p>
              <button
                onClick={() => this.setState({ hasError: false, error: null })}
                className="rounded-[var(--radius-md)] bg-[var(--color-primary)] px-4 py-2 text-sm font-medium text-white"
              >
                Try again
              </button>
            </div>
          </div>
        )
      );
    }
    return this.props.children;
  }
}
```
