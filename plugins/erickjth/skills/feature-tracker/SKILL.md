---
name: "feature-tracker"
description: Keep a living, visual progress tracker for a frontend feature that is split into a Linear parent ticket with sub-tickets — ticket dependency map with statuses, every component created, edited or deleted per sub-ticket, how data flows between components, hooks and fetchers, open questions pulled from tickets, comments and code, and code-review findings (including a Figma-vs-code audit of typography, color, border and background tokens) for sub-tickets the user marks as done. Use this whenever the user shares or names a Linear parent issue (e.g. "PIN-1667", a linear.app/…/issue/ link) and wants to track, update, sync, visualize or check progress on it, asks "where are we on the dashboard work", "update the tracker", "review PIN-1668, it's done", or wants to see which components a feature touched — even if they don't say "tracker".
---

# Feature Tracker

Maintain **one living HTML artifact per Linear parent ticket**. It shows the
state of a multi-ticket frontend feature: which sub-tickets are where, what
code each one actually changed, how the pieces connect, what's still
undecided, and what the code reviews found.

The artifact is a **renderer plus data**. `assets/tracker.html` draws
everything from a JSON state block embedded in the page. Each run reads the
current state, merges in fresh data, writes the state back, and republishes.
Never hand-edit the rendered markup, only the state.

## The rule that matters most: don't assume

This tracker is only useful if the user can trust every line on it. So:

- **Record evidence for every fact.** Each component, data edge, and status
  carries an `evidence` field: `code` (seen in a diff or file),
  `ticket` (stated in a Linear description or comment), or `figma`.
  Something a ticket *plans* but the code hasn't shown yet renders as
  "planned", never as done.
- **Never fill a gap with a guess.** If you can't tell whether a component
  was created or moved, whether a hook fetches real data or a fixture, or which
  ticket a file belongs to, add an **open question** instead of picking.
- **Surface contradictions; don't resolve them.** Two tickets disagreeing, a
  ticket disagreeing with the code, or a ticket disagreeing with Figma each
  becomes an open question that quotes both sides with links. The user
  decides.
- **Only mark a question answered when you find explicit text answering it**
  (a later comment, a description edit, or code that settles it), and link
  that source.
- **Say what you couldn't inspect.** If a PR's diff wasn't readable, the page
  says so for that ticket, rather than implying it has no changes.

## Operations

Work out which one the user wants. "Update/sync the tracker" runs **sync**.
"Review PIN-xxxx", or "PIN-xxxx is done", runs **review** (followed by a sync of
that ticket). A first request for a parent that has no tracker yet runs
**create**.

### Create (first run for a parent)

1. Look for an existing tracker first (see "Finding the artifact" below). If
   one exists, run sync instead.
2. Gather everything (see `references/sources.md`).
3. Build the state from scratch, following `references/state-schema.md`.
4. Copy `assets/tracker.html`, embed the state, and publish (see
   "Deliver").

### Sync

1. Load the existing state from the artifact.
2. Re-gather Linear data for the parent and every sub-ticket: status,
   description, comments, attachments, and relations.
3. Re-gather code for each sub-ticket that has a PR or branch.
4. Merge into the existing state:
   - Update statuses, PR metadata, and component changes.
   - Add new open questions. Keep existing ones, and update their status only
     on explicit evidence.
   - **Keep review results as they are.** If a reviewed PR's head SHA has
     changed since the review, set that review's `stale: true`. Don't
     re-review unless the user asks.
   - Write a `changelog` entry listing what changed since the last sync
     (status moves, new or removed components, new questions, new commits on
     reviewed PRs). Keep only the latest entry, since the artifact is the
     current picture, not a history.
5. Publish in place.

### Review (only when the user says so)

Never start a review on your own, even if a ticket reaches Done or its PR
merges. The user decides when a sub-ticket is ready.

1. Get the sub-ticket's full diff (see `references/sources.md`).
2. Run `mattpocock-skills:code-review` if it's available. In Claude Code,
   invoke `/mattpocock-skills:code-review` on that PR. On claude.ai, check
   `/mnt/skills` for it. Otherwise, review against
   `references/review-checklist.md`.
3. Also check the diff against the ticket's own acceptance criteria. Mark each
   criterion `met` or `not met` (with evidence), or `can't verify from code`
   (for visual or behavioral criteria, which need a human or a screenshot).
4. **Design-token audit.** Every review of frontend code includes one. Compare
   every typography, color, border, and background token in the ticket's
   Figma frames against the implementation, following
   `references/design-token-audit.md`. Figma values come from the Figma
   connector, never from memory or from a screenshot estimate. Anything you
   can't fetch or resolve is recorded as `cant-verify`, with the reason.
5. Normalize the findings into the review shape in the state schema,
   including the reviewer used, the PR head SHA you reviewed, and the
   `tokenAudit` block. Token mismatches with real visual impact also become
   review findings (usually `minor`, and `major` when they break the design
   system, for example a raw hex value where a semantic token exists).
6. Save it on the ticket, then sync that ticket and publish.

In chat, report the verdict and the counts by severity. Don't paste the whole
review.

## Finding the artifact

The tracker is read-only toward Linear: never comment, edit, or change status there.

- **claude.ai:** call the Artifact tool with `action: "list"` and look for the
  title `Tracker · <PARENT-ID> · <parent title>`. If found, `action: "read"`
  copies it into the container. Parse the `<script type="application/json"
  id="tracker-state">` block. If several match, ask which one.
- **Claude Code:** the tracker is a file. On the first run, **ask the user
  where to keep it**. Don't pick a repo path yourself. Record the path in
  `state.meta.filePath`. On later runs, look for it there, or ask if you can't
  find it.

## Building the state

Read `references/state-schema.md` for the full shape. The main pieces:

- **tickets**: one per sub-ticket, with number, title, status, blockers
  (from the parent's table *and* from Linear relations, noting when the two
  disagree), PRs, Figma links, acceptance criteria, and code availability.
- **components**: every frontend unit the feature touches (component, hook,
  fetcher, fixture, type module, util, or route), with its change type per
  ticket (`created`, `edited`, `deleted`, `moved`, or `planned`), its render
  parent, and evidence.
- **edges**: how data flows, from a component or hook to another, with a kind
  (`renders`, `props`, `hook`, `fetch`, `context`, `callback`, or `url`), a
  label with example data where known, the ticket that introduced it, and
  evidence.
- **questions**: open questions, with the verbatim quote, source link, raised
  date, status, and any answer.
- **reviews**: stored on each ticket.

How to derive components and edges from code is in `references/sources.md`
under "Reading the code".

## Deliver

1. Copy `assets/tracker.html` to the outputs folder (claude.ai) or the
   recorded path (Claude Code).
2. Replace the contents of the `tracker-state` script block with the state
   JSON. **Escape `</` as `<\/`** inside the JSON so a string can't close the
   script tag.
3. Validate before publishing. Extract the JSON and parse it, then run the
   page's render script under `node` + jsdom (or at minimum check that the
   JSON matches the schema) and confirm there are no errors in the console
   output.
4. On **claude.ai**, publish with the Artifact tool. Pass the existing
   artifact's `url` on sync so it updates in place. Use the title from
   "Finding the artifact" and the 🧭 favicon. In **Claude Code**, write the
   file and give the path.

In chat, say what changed since the last sync in a few sentences, plus the
number of open questions and anything you couldn't inspect. The page holds the
detail.

## Reference files

- `references/sources.md` covers what to fetch from Linear, GitHub, and Figma
  in each environment, and how to read components and data flow out of
  diffs.
- `references/state-schema.md` defines the JSON state shape, with an example.
- `references/review-checklist.md` is the frontend review checklist used as
  a fallback.
- `references/design-token-audit.md` covers the Figma-vs-code audit of
  typography, color, border, and background tokens, run on every review.
- `assets/tracker.html` is the renderer template.
