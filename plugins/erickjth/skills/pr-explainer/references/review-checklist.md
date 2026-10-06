# Built-in code review checklist

Use this list when the `mattpocock-skills:code-review` skill isn't available.
It isn't a form to fill in. Walk the diff with each lens and report only real
findings. A review with three sharp findings beats one with fifteen nits.

For every finding, name the concrete failure: which input, which user action,
and what goes wrong. "This could be a problem" isn't a finding.

## Severity guide

- **blocker**: wrong behavior in a realistic path, data loss or corruption,
  a security hole, or breaking a public contract without migration.
- **major**: a likely bug in an edge case, a missing test for the core change,
  a significant performance regression, or a hard-to-reverse design choice.
- **minor**: clarity, maintainability, or a small inefficiency. Worth fixing,
  not worth blocking on.
- **nit**: naming, style, or taste. Group nits together, and keep five at most.

## 1. Correctness

- Does the change actually do what the PR claims? Trace one concrete input
  through it.
- Boundary cases: empty, null or undefined, zero, one, many, max length,
  unicode, timezones and DST, and rounding with money or doses.
- Error paths: what happens when the fetch fails, the queryset is empty, or
  the external API times out? Are errors swallowed?
- Concurrency: double submits, stale closures, race conditions between
  requests, and transactions or `select_for_update` where rows are contended.
- Is anything else relying on the old behavior? Search for other callers.

## 2. Frontend (React / Next.js)

- Hook rules: dependency arrays complete and stable, and no conditional hooks.
- Effects that should be event handlers or derived state.
- Keys on lists: stable and unique, not array indices when items reorder.
- Server and client boundaries: `"use client"` placement, secrets or
  server-only imports leaking into client bundles, and hydration mismatches
  from dates, randomness, or `window`.
- Data fetching: loading, error, and empty states. Cache or revalidation
  behavior, and request waterfalls.
- Re-render cost: new object or function props on every render passed to
  memoized children, and context values that change identity every render.
- Accessibility: labels, roles, focus management in modals, keyboard reach,
  and color contrast.
- SEO and metadata where pages change: `generateMetadata`, canonical URLs,
  and status codes for not-found.

## 3. Backend (Django)

- Queries: N+1s (look for loops touching relations without
  `select_related` / `prefetch_related`), missing indexes on new filters,
  and unbounded querysets.
- Migrations: are they reversible, safe on a large table (locks, defaults
  on big tables), and ordered correctly with code deploys?
- Serializers: validation on every writable field, `read_only` where it
  should be, and no accidental exposure of fields.
- Permissions: does every new view or action check auth *and* object-level
  ownership? Can user A reach user B's records by changing an ID?
- Transactions: multi-write operations wrapped in `transaction.atomic`, and
  side effects (emails, tasks) deferred with `on_commit`.
- Celery or background tasks: idempotent, safe to retry, and passed IDs
  rather than model instances.

## 4. Security & privacy

Pinch handles health data, so weigh these heavily.

- PHI or PII in logs, Sentry breadcrumbs, analytics events, URLs, or error
  messages.
- Injection: raw SQL, `mark_safe` or `dangerouslySetInnerHTML` with user
  input, and unvalidated redirects.
- Secrets or keys committed, or exposed to the client.

## 5. Tests

- Is the core behavior change covered by a test that would fail without it?
- For bug fixes, is there a regression test reproducing the original bug?
- Do the tests assert behavior, or only implementation details and snapshots?

## 6. Design & maintainability

- Is this the simplest shape that works? Watch for premature abstraction,
  and equally for copy-paste that should be shared.
- Naming that misleads, and dead code left behind.
- Does the PR mix unrelated changes that should be separate PRs?

## Verdict

- **Request changes** if there's any blocker.
- **Approve with comments** if there are majors the author can address in a
  follow-up, or only minors.
- **Approve** if there are only nits or nothing at all.
