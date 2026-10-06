# Tracker state schema

The page renders entirely from this JSON. Keep field names exact, because the
renderer reads them directly. Omit optional fields rather than setting them
to empty placeholder strings.

```jsonc
{
  "version": 1,
  "meta": {
    "parentId": "PIN-1667",
    "parentTitle": "FE Provider Dashboard Development",
    "parentUrl": "https://linear.app/…/PIN-1667/…",
    "parentStatus": "In Progress",
    "integrationBranch": "pin-1667-fe-provider-dashboard-development", // only if stated or seen
    "figma": [{ "title": "Provider Dashboard Updates – Sep 2026", "url": "…" }],
    "lastSynced": "2026-09-28T15:00:00Z",
    "environment": "claude-code | claude-ai",
    "filePath": "…"                        // Claude Code only, the path the user chose
  },

  "changelog": {                           // latest sync only
    "since": "2026-09-25T22:00:00Z",       // previous lastSynced, or omit on first run
    "items": ["PIN-1674 moved In Progress → In Review", "3 new components from PIN-1670"]
  },

  "tickets": [
    {
      "id": "PIN-1668",
      "number": "01",                      // from the title prefix, if present
      "title": "Dashboard V2 shell and mock data layer",
      "url": "…",
      "status": "In Review",
      "statusType": "backlog | unstarted | started | completed | canceled",
      "blockedBy": [{ "id": "PIN-1667", "source": ["table", "relation"] }],
      "externalBlockers": ["backend endpoints live"],
      "prs": [{
        "number": 1251, "url": "…", "title": "…",
        "state": "open | merged | closed | draft",
        "base": "…", "head": "…", "headSha": "abc1234",
        "files": { "created": 12, "edited": 5, "deleted": 7, "moved": 1 }
      }],
      "code": "inspected | not-inspected | none-found",
      "codeNote": "PR is private; no diff available on claude.ai",   // optional
      "figma": [{ "title": "Desktop / Dashboard / Active", "url": "…" }],
      "acceptance": [
        { "text": "Mock fetcher and card primitive have unit tests…", "checkedInLinear": true,
          "verified": "met | not-met | cant-verify | not-reviewed", "note": "…" }
      ],
      "review": null                       // or the review object below
    }
  ],

  "components": [
    {
      "id": "DashboardCard",               // unique; use the export name, suffix with path segment if duplicated
      "name": "DashboardCard",
      "kind": "route | component | hook | fetcher | fixture | types | util",
      "path": "src/…/DashboardCard.tsx",
      "oldPath": "…",                      // moved only
      "parent": "ProviderDashboardPage",   // primary render parent id, components only; omit for roots and non-components
      "changes": [                         // one per ticket that touched it
        { "ticket": "PIN-1668", "type": "created | edited | deleted | moved | planned", "evidence": "code | ticket | figma", "note": "…" }
      ],
      "clientServer": "client | server",   // only if seen ("use client" or the lack of it in the app dir)
      "note": "…"
    }
  ],

  "edges": [
    {
      "from": "useGetAnnualGoal", "to": "mockFetcher",
      "kind": "renders | props | hook | fetch | context | callback | url",
      "label": "GET goal → fixture annualGoal.active (mock)",
      "ticket": "PIN-1671",
      "evidence": "code | ticket"
    }
  ],

  "questions": [
    {
      "id": "q1",
      "text": "Is the dashboard behind a feature flag or not?",
      "kind": "open-in-ticket | contradiction | unverifiable | code-vs-ticket | scope",
      "sources": [
        { "ticket": "PIN-1667", "where": "description", "quote": "A finished sub-issue is demoable on the flagged dashboard route", "url": "…" },
        { "ticket": "PIN-1668", "where": "description", "quote": "no feature flag, no V1/V2 variants", "url": "…" }
      ],
      "raised": "2026-09-14",
      "status": "open | answered",
      "answer": { "text": "…", "source": { "ticket": "…", "where": "comment", "url": "…", "date": "…" } },
      "tickets": ["PIN-1667", "PIN-1668"]   // which tickets it affects, used for filtering
    }
  ]
}
```

## Review object (on a ticket)

```jsonc
{
  "reviewedAt": "2026-09-28T15:10:00Z",
  "reviewer": "mattpocock-skills:code-review | built-in checklist",
  "pr": 1251,
  "sha": "abc1234",                        // PR head SHA that was reviewed
  "stale": false,                          // set true on sync if the head SHA has changed since
  "verdict": "approve | comments | changes",
  "summary": "One paragraph.",
  "findings": [
    { "severity": "blocker | major | minor | nit",
      "title": "…", "location": "src/…/file.tsx:42",
      "finding": "…", "why": "…", "suggestion": "…", "snippet": "optional code" }
  ],
  "raw": "verbatim external reviewer output, if any",
  "tokenAudit": {
    "status": "audited | partial | not-audited",
    "reason": "why not audited or partial, e.g. Figma node not accessible",
    "figmaNodes": [{ "title": "Desktop / Dashboard / Active", "url": "…", "nodeId": "20551:48349", "breakpoint": "desktop" }],
    "tokenSources": ["tailwind.config.ts", "src/styles/tokens.css"],
    "items": [
      { "category": "typography | color | border | background",
        "element": "DashboardCard › eyebrow label",
        "location": "src/…/DashboardCard.tsx:31",
        "property": "text style | fill | stroke | radius | …",
        "breakpoint": "desktop",
        "figma": { "token": "label/sm", "value": "Inter 12/16 600" },
        "code":  { "token": "Typography variant=\"labelSm\"", "value": "Inter 12/16 600" },
        "status": "match | token-mismatch | value-mismatch | hardcoded | untokenized-figma | missing-in-code | not-in-figma | cant-verify",
        "matchedBy": "name | value",
        "note": "…",
        "question": "q3" }          // id of the linked open question, if any
    ]
  }
}
```

## Merge rules on sync

- Match tickets by `id`, components by `id`, questions by `id`, and edges by
  `from + to + kind`.
- A component that no longer appears in any ticket's code *and* was only ever
  `planned` stays, since the plan still exists. A component whose only code
  change was reverted (for example the file is gone from the PR) loses that
  change entry. Mention it in the changelog.
- Never delete a question. Only move it to `answered`, with an answer source.
- Never alter a stored review, except for setting `stale`.
