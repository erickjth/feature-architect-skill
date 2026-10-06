# Frontend review checklist (fallback)

Use this list when `mattpocock-skills:code-review` isn't available. Walk the
sub-ticket's diff through each lens and report only real findings, each tied to
a concrete failure (which input or user action, and what goes wrong).

## Severity
- **blocker**: wrong behavior on a realistic path, a security or privacy leak,
  or a broken build or type-check.
- **major**: a likely bug in an edge case, a missing test for core logic, a
  notable performance or a11y regression, or a decision that's costly to undo.
- **minor**: clarity and maintainability issues, small inefficiencies.
- **nit**: naming and style. Keep five at most, grouped together.

## 1. Does it do what the ticket says?
- Trace one concrete fixture through the change end to end.
- Check each acceptance criterion against the code (`met`, `not met`, or
  `can't verify from code`).
- Look for scope creep (work belonging to another sub-ticket) and scope gaps.

## 2. Data layer
- Hooks go through the agreed fetcher. No component calls fetch directly
  if the feature defines a single swap point.
- Provisional or hand-typed contract types match the "Proposed API contract"
  in the ticket, field for field. Mismatches are findings.
- Loading, error, and empty states are handled for every hook consumer.
- Mock and real parity: fixtures shaped like the real response, including
  nulls and empty arrays.
- Cache keys are stable and don't collide across modules.

## 3. React / Next.js
- Hook rules: complete, stable dependency arrays; no conditional hooks.
- Effects that should be derived state or event handlers.
- List keys: stable and unique, not indices when items reorder.
- `"use client"` placement, and no server-only imports or secrets in client
  bundles. Watch for hydration mismatches from dates, `Date.now()`,
  randomness, and `window`.
- Re-render cost: new object or function props passed to memoized children,
  and context values that change identity every render.
- Dates and timezones: the provider's timezone versus the browser's, and
  month/year boundaries for period stats.

## 4. Design system
- The token-by-token Figma comparison is a separate, required step (see
  `design-token-audit.md`). Here, turn its mismatches into findings.
- Conventions the ticket or repo states (for example typography through a
  shared component, colors through semantic tokens, shared card primitives).
  Flag raw hex values, ad-hoc font sizes, and duplicated primitives.
- Responsive behavior matches the stated breakpoints. Container-width versus
  viewport-width choices should be deliberate.
- Accessibility: semantic elements, labels, focus order, keyboard reach,
  contrast, `aria-live` for async updates, and reduced motion.

## 5. Privacy
- No client PHI or PII in logs, analytics events, Sentry breadcrumbs, URLs, or
  error messages.
- No `dangerouslySetInnerHTML` with API data.

## 6. Tests
- Logic units (hooks, formatters, and the fetcher) have tests that would fail
  without the change.
- Skeleton and error states are tested where the ticket requires it.
- Tests assert behavior, not implementation details.

## 7. Cleanup
- Deleted components have no remaining imports, stories, or tests.
- No dead flags, local-storage keys, or unused fixtures left behind.
- There's a TODO for every provisional piece another ticket will replace,
  with that ticket's number referenced.

## Verdict
- **Request changes** if there's any blocker.
- **Approve with comments** if there are majors fit for a follow-up, or only minors.
- **Approve** if there are only nits or nothing at all.
