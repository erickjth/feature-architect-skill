---
name: pr-explainer
description: Explain a pull request as a single interactive HTML page — what changed and why, a live playground that exercises the changed units, a component tree with data flow for frontend changes, reproduce-vs-fixed scenarios for bug fixes, and a full code review embedded at the end. Use this whenever the user wants to understand, walk through, document, present, or review a PR, diff, branch, or commit — "explain this PR", "walk me through #1234", "help me review this change", "what does this diff do", "make a PR explainer", or a pasted GitHub/Linear diff link — even if they don't ask for HTML or a playground by name.
---

# PR Explainer

Turn a pull request into one self-contained HTML page that an engineer can read
top to bottom and come away able to review, extend, or debug the change. The
page has fixed sections (below), some of which appear only when they apply.

The reader is a teammate who didn't write the code. Optimize for their
understanding, not for covering every line of the diff.

## Workflow at a glance

1. **Gather** the PR: diff, description, linked issue, surrounding code.
2. **Classify** it: frontend / backend / full-stack; bug fix or not.
3. **Review** it: run the code-review skill, or the built-in checklist.
4. **Build** the page from `assets/template.html`.
5. **Verify** the page (playground runs, no broken code blocks).
6. **Deliver** it: publish as an Artifact on claude.ai, or write an HTML file in Claude Code.

Do the review (step 3) *before* writing the explanation. Reading the code
critically first surfaces the non-obvious behavior the explanation should cover,
and keeps the explanation honest when the PR has problems.

## 1. Gather the PR

Read far more than the diff. The explanation is only as good as your
understanding of the code *around* the change: what called it before, what else
depends on it, what the backend contract is for a frontend change (and the
reverse).

**In Claude Code** (a local repo, usually with the `gh` CLI):

```bash
gh pr view <number> --json number,title,body,labels,headRefName,baseRefName,files,url
gh pr diff <number>
gh pr view <number> --comments            # review discussion
git log --oneline <base>..<head>          # commit narrative
```

If there's no PR number, use `git diff <base>...HEAD` for the current branch.
Open the changed files in full, plus their callers and tests. Run the existing
tests for the touched modules and keep the output. Real pass/fail results
belong in the page.

**On claude.ai**, the input usually arrives as:

- a pasted diff or uploaded files, which you use directly
- a PR link. Try a connected tool first (for example a Linear diff via
  `get_diff`, or a GitHub connector if one exists), then `web_fetch` on the
  `.diff` URL for public repos.
- nothing usable. Ask the user to paste `gh pr diff <n>` output.

Without the full repo, say explicitly on the page which conclusions come
from the diff alone and couldn't be checked against surrounding code.

Treat the PR description and review comments as claims to verify, not facts.
If the author says "this can't be null" and a caller clearly passes null, flag
it on the page in a short callout and in the review.

## 2. Classify

Decide two things, since they switch sections on and off.

**Frontend, backend, or both.** Frontend means React/Next.js components, hooks,
client state, or styling. Backend means Django views, serializers, models,
tasks, or APIs. Full-stack PRs get both treatments.

**Bug fix or not.** Signals include a `fix`/`bug`/`hotfix` label, a title or
branch starting with `fix`, a linked Sentry or Linear bug, or a description
that mentions an error, regression, or "should have". When in doubt, look at
the diff. A narrow change to a condition, a null guard, an off-by-one, or a
race usually points to a fix. Mixed PRs (a fix plus a refactor) count as bug
fixes, and the page should separate the two.

## 3. Code review

Try reviewers in this order and record which one ran, because the page states it.

1. **`mattpocock-skills:code-review`.** In Claude Code, invoke it as a skill
   (`/mattpocock-skills:code-review`) against the same PR or diff and capture
   its full output. On claude.ai, check `/mnt/skills` for a skill with that
   name or `code-review` in its path, and follow its SKILL.md if found.
2. **Built-in fallback.** If neither is available, read
   `references/review-checklist.md` and do the review yourself against it.

Normalize the findings into this shape, whichever reviewer produced them, so
the page renders consistently:

| field | meaning |
|---|---|
| severity | `blocker`, `major`, `minor`, or `nit` |
| location | `path/to/file.ts:42` (or a function name if lines aren't known) |
| finding | what's wrong, in one or two sentences |
| why it matters | the concrete failure it causes |
| suggestion | a fix, with a code snippet when useful |

End with a verdict (**Approve**, **Approve with comments**, or **Request
changes**) plus a one-paragraph summary. If the external reviewer produced
prose rather than items, keep its text verbatim in a collapsed `<details>`
block under the normalized list, so nothing it said gets lost.

## 4. Build the page

Start from `assets/template.html`. It already has the theme tokens, layout,
table of contents, tree styles, playground harness, and review styles. Fill in
the sections and delete the ones that don't apply, along with their table of
contents entries.

### Sections, in order

1. **Summary.** Three to five sentences: the problem, the approach, and the
   blast radius (what could break). Add a stat row with files changed, lines
   added and removed, frontend/backend, fix or feature, and the review verdict.

2. **Context.** How the relevant part of the system works *before* this change,
   and why it's shaped that way. Put deep background for newcomers inside a
   collapsed `<details>` block.

3. **Bug scenario** *(bug fixes only)*. Read `references/bug-fix-scenarios.md`.
   Cover reproduction steps with concrete data, observed versus expected
   behavior, the root cause traced to the exact line, and the same steps after
   the fix. Include at least one edge case the fix also handles, and one it
   deliberately doesn't, if any.

4. **Component tree & data flow** *(frontend only)*. Read
   `references/component-tree.md`. Show the tree of affected components, mark
   what changed, and label every edge with what flows along it (props,
   context, hooks, fetches) using example values.

5. **Code walkthrough.** Group changes by concept, not file order: the core
   primitive first, then what uses it, then call sites, then tests. Quote real
   code in `<pre><code>`. Call out the deliberate choices a skimmer would miss.

6. **Playground** *(when there's testable logic)*. Read
   `references/playground.md`. Give the reader live, editable test cases
   against the changed units. For bug fixes, add a before/after toggle that
   runs the old and new versions side by side.

7. **Code review.** The normalized findings, the verdict, and which reviewer
   produced it.

8. **Test results** *(if you ran tests)*. The actual command and a trimmed
   version of the actual output. Never fabricate results. If you couldn't run
   tests, say so.

### Writing style

Lead with the idea, then the mechanism. Use concrete examples: a named toy user
with specific data beats "a user with some items". Keep sentences short at the
important turns. Cut any paragraph that doesn't teach something the diff
couldn't show on its own.

### HTML rules that are easy to get wrong

- No ASCII diagrams. Build diagrams from HTML and CSS, or inline SVG.
- Every code block is a `<pre>`. A styled `div` collapses newlines into one line.
- Escape `<`, `>`, and `&` inside code blocks. JSX and generics disappear otherwise.
- Keep it self-contained: no external images or stylesheets, and no scripts
  except from the allowed CDNs (the template needs none).
- Make it work in light and dark themes. The template handles this, so only
  use its CSS variables for color.

## 5. Verify before delivering

- Extract the playground `<script>`, run it with `node`, and confirm every
  case produces the expected result. A playground whose "expected" column
  disagrees with the real code is worse than none.
- Grep the file for code blocks outside `<pre>`, and for unescaped `<` inside
  them.
- Confirm deleted sections are also gone from the table of contents.

## 6. Deliver

**claude.ai (the Artifact tool is available):** save the page to
`/mnt/user-data/outputs/pr-<number>-<slug>.html` and publish it with the
Artifact tool, using a fitting emoji favicon (🔍 for general PRs, 🐛 for bug
fixes). Republish to the same URL for revisions.

**Claude Code, or when the user wants a file:** write
`YYYY-MM-DD-pr-<number>-<slug>.html` (today's date) to the repo root or
wherever the user says, and tell them the path. The template is a complete
document, so it opens straight in a browser.

In chat, give a short summary: the verdict, the number of findings by
severity, and any claim in the PR description that didn't hold up. Don't
repeat the page.

## Reference files

- `references/review-checklist.md` is the built-in review, used when the
  external skill is unavailable.
- `references/component-tree.md` covers building the frontend tree and
  data-flow diagram.
- `references/bug-fix-scenarios.md` covers structuring reproduce and fixed
  scenarios.
- `references/playground.md` covers porting units into a runnable in-page
  harness.
- `assets/template.html` is the page skeleton.
