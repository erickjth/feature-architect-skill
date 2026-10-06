# Design-token audit: Figma vs implementation

This runs on every sub-ticket review. It covers four categories:
**typography, color, border, and background**. The aim is a row for every
styled element the ticket built or changed, saying whether the code uses the
same token Figma specifies.

The don't-assume rules apply at full strength here. Figma values come only
from the Figma connector, and code values only from files you read. Never
estimate a value from a screenshot, and never assume two tokens are the same
because their names look alike.

## 1. Collect the Figma side

Use the Figma frames attached to the sub-ticket (and to the parent, if the
sub-ticket has none). Load the Figma connector tools with `tool_search`. From
each frame URL, take the `node-id` (write `20551-48349` as `20551:48349`) and
the file key.

- `get_variable_defs(nodeId)` returns the variables and styles the node uses.
  This is the source of truth for token *names* and values.
- `get_design_context(nodeId)` returns per-layer styling, so you know *which
  element* uses which token. Take the specific card or module frame, not the
  whole page, to keep it focused.
- `get_screenshot(nodeId)` is optional, for matching layers to components by
  eye. Don't read values from it.

For each relevant layer, record:

| category | what to record from Figma |
|---|---|
| typography | the text style or variable name, plus font family, size, line height, weight, and letter-spacing |
| color | the text or icon fill variable name, plus the resolved value |
| background | the fill variable name on the container, plus the resolved value |
| border | the stroke variable name, plus width, style, and radius (with radius variables if used) |

If a layer uses a **raw value with no variable or style** (for example
`#ffe9d9`), record it as untokenized. That's a design-side gap, not a code
bug.

If the connector fails, the frame isn't accessible, or no frame is linked, set
`tokenAudit.status: "not-audited"` with the reason, and stop. Don't audit from
memory of the design.

## 2. Collect the code side

Read the changed components in full at the PR head, plus the files that
define tokens. Find those files; don't guess their paths. Look for things like
a Tailwind config's `theme.extend`, CSS custom properties, a tokens module,
or the typography component's variant map. Record which files you used in
`tokenAudit.tokenSources`.

For each styled element, record what the code uses:
- **typography**: the typography component and variant or props, a text
  utility class, or raw `font-size`/`font-weight`/`line-height`
- **color / background / border**: a semantic token (class, CSS variable, or
  theme key), a raw token (a palette step), or a raw literal (hex, rgb, px)

Then resolve every code token to its final value through the token files, so
it can be compared.

## 3. Match and classify

Pair Figma layers with code elements by what they are: "DashboardCard ›
eyebrow label", "GoalCard › progress bar track". If you can't pair a Figma
layer with any code element, or the reverse, that's its own status (below).
Don't force a pairing.

Figma variable names and code token names often follow different
conventions. Match them in this order, and record which method you used
(`matchedBy`):
1. `name`: a documented mapping in the repo, or an identical semantic name
2. `value`: identical resolved values but different names (flag it, since it
   may be a coincidental match)

| status | meaning |
|---|---|
| `match` | same token, same resolved value |
| `token-mismatch` | a different token is used, even if the values happen to match. For example, Figma uses `text/secondary` and the code uses `gray-600` |
| `value-mismatch` | the resolved values differ |
| `hardcoded` | the code uses a raw literal where Figma uses a token |
| `untokenized-figma` | Figma uses a raw value, so there's no token to compare |
| `missing-in-code` | the Figma layer has no implemented counterpart |
| `not-in-figma` | the code styles an element that the frame doesn't show (hover and focus states often land here) |
| `cant-verify` | the value couldn't be resolved on one side. State why |

Every `untokenized-figma` row, and every `value`-based match you aren't sure
about, also becomes an **open question** on the tracker (kind `scope` or
`code-vs-ticket`) with both values quoted. The same goes for any conflict with
decisions written in the ticket. For example, PIN-1668 already notes
"`semantic-warning-subtle` (#fff3e0) vs Figma's untokenized #ffe9d9". Link to
that existing question rather than making a duplicate.

Responsive variants count separately. If the ticket links both desktop and
mobile frames, audit both, and note the breakpoint on each row.

## 4. Record it

Store the result on the review as `tokenAudit` (see `state-schema.md`). Keep
one row per element, per category, per property that differs. Don't write
separate rows for each property that matches. A `match` row can cover a whole
text style.

Findings: turn `hardcoded` and `token-mismatch` rows into review findings,
grouped by component so there's one finding per component. `value-mismatch`
rows become findings with the two values side by side. Don't turn
`untokenized-figma` rows into findings. They're questions for design.
