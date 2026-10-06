# Bug fix scenarios

A reader should be able to reproduce the bug from this section alone, then see
why the fix works and where it stops working. Write it as a story with named,
concrete data, not a generic description.

## Structure

### 1. Symptom
One or two sentences on what the user or system saw. Quote the real error,
Sentry title, or support ticket if there is one.

### 2. Reproduce (before the fix)

Put the preconditions in a small table or code block holding the exact state
needed. For example, provider "Dr. Lina Ortiz" in timezone `America/Bogota`,
with an appointment at `2026-11-01T23:30:00-05:00`.

Number the steps. Each step is one user action or API call, written so a
teammate could follow it in staging:

1. Log in as provider Lina.
2. Open `/schedule?date=2026-11-01`.
3. …

Give **expected versus actual** side by side, in a two-column table or cards
with the `bad` and `good` accents from the template.

### 3. Root cause
Point to the exact line or lines in the *old* code, quoted in a `<pre>`. Trace
the concrete data through them until the wrong value appears, and show the
intermediate values. This is where the reader has the "oh" moment.

If the root cause sits somewhere other than where the fix landed (a band-aid
fix), say so plainly, and add a matching review finding.

### 4. Same steps, after the fix
Repeat the reproduction steps with the new behavior at each step where it
differs, and quote the changed lines.

### 5. Scenario matrix
Make a table covering the original case plus two to five related cases:

| scenario | input | before | after | covered by test? |
|---|---|---|---|---|
| original bug | 23:30 UTC-5 | shows Nov 2 ❌ | shows Nov 1 ✅ | yes, `test_late_evening_tz` |
| DST boundary | … | … | … | no |
| UTC user | … | ✅ | ✅ | yes |

Include at least one case that was already fine (proving the fix didn't
regress it), and any case the fix does **not** handle, labelled honestly as
out of scope or a remaining risk.

### 6. Wire it into the playground
Each row of the matrix should become a playground case with the before/after
toggle (see `playground.md`), so the reader can run the matrix instead of
trusting it.

## Where the details come from

In rough order of reliability: a regression test added in the PR, the linked
Sentry event or Linear issue, the PR description, then your reading of the
diff. If you built the reproduction from the diff alone, say so, and describe
it as "expected to reproduce" rather than "reproduces".
