# Sources: what to fetch, and how to read it

## 1. Linear (both environments)

Use the Linear connector (on claude.ai, load its tools with `tool_search`
first). The skill is **read-only**: never call `save_issue`, `save_comment`, or
any other write tool.

For the parent:
- `get_issue(id, includeRelations: true)` returns the description, status,
  attachments (Figma, docs), and relations.
- `list_comments(issueId)` returns the discussion.
- `list_issues(parentId, fields: [title, status, statusType, gitBranchName, completedAt, updatedAt, assignee])`
  returns the sub-tickets.

For each sub-ticket:
- `get_issue(id, includeRelations: true)` returns the description (which
  holds "What to build", acceptance criteria, contracts, and follow-ups),
  attachments (GitHub PRs and Figma frames), and blocked-by relations.
- `list_comments(issueId)` returns the comments.

### Blockers: two sources, and they can disagree

Parents often carry a "Blocked by" table, and Linear has formal `blockedBy`
relations. Read both. Store the union, and tag each blocker with where it came
from (`table`, `relation`, or both). If one lists a blocker the other doesn't,
add an open question. Blockers that aren't tickets (for example "backend
endpoints live") go in `externalBlockers` as text.

### Open-question material in ticket text

Scan descriptions and comments for:
- explicit markers: "open", "still open", "TBD", "to be decided", "set
  later", "confirm with", "need to check", "?", or "follow-up"
- checklists with unchecked items that describe decisions rather than work
- "Decisions" sections, where some items are decided and some are explicitly
  not

Quote the exact sentence. Don't paraphrase it into something more certain or
less certain than it is.

### Contradictions

Compare the parent against each sub-ticket, and sub-tickets against each other,
on scope (which ticket owns which module or primitive), mechanism (flag or no
flag, mock or real data), and contracts (field names and shapes). Record each
contradiction as a question with both quotes and both links.

## 2. Code

### Claude Code (repo + `gh`)

For each sub-ticket, find its PR from, in order: a GitHub attachment on the
Linear issue, then `gh pr list --search "<TICKET-ID>" --state all`, then
`gh pr list --head <gitBranchName>`. If none is found, the ticket has no code
yet. Record `code: "none-found"`, not "no changes".

```bash
gh pr view <n> --json number,title,state,isDraft,baseRefName,headRefName,headRefOid,mergedAt,url,files
gh pr diff <n>
```

Note `baseRefName`. If a sub-ticket PR targets something other than the
integration branch the parent describes, that's an open question.

Read the changed files in full at the PR head (`gh pr diff` plus
`git show <headRefOid>:<path>`), not just the hunks. Components and data flow
need the whole file.

### claude.ai

There's no GitHub access for private repos. For each ticket with a PR link,
record `code: "not-inspected"` along with the PR URL, and take components
from the ticket text only (evidence `ticket`, change `planned`). Tell the user
that code-level detail needs either a Claude Code run or pasted
`gh pr diff` / `gh pr view --json files` output. If they paste it, use it for
those tickets.

## 3. Reading the code

Only frontend files count (for example `.tsx`, `.ts`, `.jsx`, `.js`, and
`.css`/`.scss` modules under the frontend app). Ignore lockfiles and generated
files, but mention codegen output if the ticket is about it.

### Change type per file

From the PR file list, `added` means `created`, `removed` means `deleted`,
`modified` means `edited`, and `renamed` means `moved` (record the old path).
If a file is deleted in one place and a near-identical one is added
elsewhere, don't assume a move. Record both, and add a question if it
matters.

### What counts as a unit

| kind | how to recognize it |
|---|---|
| `route` | `app/**/page.tsx`, `layout.tsx`, `pages/**` |
| `component` | exported function or const returning JSX |
| `hook` | exported `use*` function |
| `fetcher` | module that performs or mocks network calls (`fetch`, `pinchFetch`, a mock resolver) |
| `fixture` | static mock data module |
| `types` | type- or contract-only module |
| `util` | anything else with logic |

One file can hold several units. List the ones that matter (exported ones,
plus internal ones the ticket calls out).

### Edges

- `renders`: parent JSX includes `<Child …>`. Take the component tree from
  these edges.
- `props`: the notable props passed on a `renders` edge. Label them with an
  example value when fixtures or tests give one (for example
  `goal={target: 120000, period: "2026"}`).
- `hook`: a component calls `useX()`.
- `fetch`: a hook calls a fetcher. Label it with the endpoint or fixture
  name, and whether it resolves **mock** or **real** data, exactly as the
  code shows.
- `context` / `callback` / `url`: as named.

Add only edges you actually read in code (evidence `code`). An edge a ticket
describes but that isn't in code yet gets evidence `ticket`, and renders
dashed.

## 4. Figma

Keep Figma links as references on the ticket. Don't claim the UI matches
Figma unless you actually compared it (for example with the Figma connector's
`get_screenshot` on the linked node against a screenshot of the running page).
Visual acceptance criteria otherwise stay "can't verify from code".
