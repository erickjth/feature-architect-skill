# Playground: testing the changed units in the page

The playground lets the reader poke the changed logic with their own inputs,
instead of trusting the explanation. It's only worth including when the PR
has **units with logic**, such as pure functions, reducers, hooks with
derivable output, validators, serializers' `validate_*` methods, formatters,
permission checks, and state machines. Skip it for pure markup, styling, config,
or migrations, and say in one line why there's no playground.

## 1. Pick the units

Choose one to four units at the heart of the change. For each, record:
- the name and file path
- its signature (inputs and output)
- what makes it testable in isolation, and which dependencies you stub

## 2. Get the code into the browser

The page runs plain JavaScript, with no build step and no network.

- **JS / TS:** copy the function in, strip TypeScript types by hand (remove
  annotations, `interface`/`type` blocks, `as` casts, and generics), and
  inline tiny helpers it calls. Keep the logic byte-for-byte identical
  otherwise.
- **React hooks and components:** extract the pure part (the reducer, the
  derived-state computation, the event handler logic) into a function and
  test that. Don't try to mount React unless the UI behavior *is* the change.
  If it is, React's UMD build from cdnjs works in a published artifact, but not
  in an offline file.
- **Python / Django:** the browser can't run it reliably here. Port the logic
  faithfully to JS, label it **"JS port of `module.func`"**, and keep the
  Python source visible next to it. Stub ORM calls with plain arrays of
  objects. If you ran the real pytest suite, show those real results in the
  Test results section, since they're the ground truth and the port is the
  toy.

For bug fixes, include **both** versions: `before` (from the base branch) and
`after` (from the PR head).

## 3. Define cases

Each case is `{ name, input, expected }`, where `input` is an array of
arguments. Seed the cases from:
1. the tests in the PR, translated directly
2. the bug scenario matrix, for fixes
3. boundary values from the review (empty, null, zero, max, timezone edges)

Derive `expected` by *running the real code* where you can (node for JS,
python for Python in the container or Claude Code), not by reasoning. Then
confirm the port produces the same values.

## 4. Use the template harness

`assets/template.html` has a `PLAYGROUND` config and renderer. Fill in:

```js
const PLAYGROUND = {
  units: [
    {
      id: "formatSlot",
      title: "formatSlot(isoString, tz)",
      source: "lib/schedule/formatSlot.ts",
      port: false,                    // true for JS ports of Python
      before: function (iso, tz) { /* old code */ },   // omit if not a bug fix
      after:  function (iso, tz) { /* new code */ },
      cases: [
        { name: "late evening Bogotá", input: ["2026-11-01T23:30:00-05:00", "America/Bogota"], expected: "Nov 1, 11:30 PM" },
      ],
    },
  ],
};
```

The renderer gives each unit:
- a table of cases showing input, expected, actual after, actual before
  (when present), and a pass/fail status
- an editable JSON input box with **Run** so the reader can try their own
  arguments
- a **Before / After / Both** toggle when `before` exists
- a pass count header, e.g. "6/6 passing"

Results are compared with deep equality on the JSON form. A thrown error
displays as `Error: …` and counts as a pass only when `expected` is
`{ "throws": true }`.

## 5. Verify

Copy the `PLAYGROUND` object and the harness's `deepEqual` into a scratch
`.js` file, run it with `node`, and confirm every case passes. For bug fixes,
also confirm the bug case *fails* under `before`. A playground that doesn't
demonstrate the bug is missing its point.
