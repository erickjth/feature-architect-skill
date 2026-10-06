# Component tree & data flow (frontend PRs)

The goal is for the reader to see, at a glance, where the changed components
sit, what feeds them, and what they feed. An empty boxes-and-arrows diagram
teaches nothing. Every edge carries a label with real example data.

## 1. Build the tree from code, not guesses

Start at the nearest route or page (`app/**/page.tsx`, `layout.tsx`, or
`pages/*.tsx`) above the changed components, and walk down to the changed
leaves. Include:

- every changed component (mark it **changed**)
- every new component (mark it **new**)
- ancestors on the path from the page down, so the reader knows where it lives
- siblings or children that consume data the change affects
- providers (context, React Query, Redux store) that the changed components read

Leave out unrelated subtrees and collapse them to a single "…" node. With more
than about 20 nodes, split the tree into one overview plus one zoomed-in tree
per area the change touches.

Mark the server/client boundary, since in Next.js the `"use client"` line
decides what can hold state and what ships to the browser.

## 2. Classify each edge

Use one visual style per edge kind, and keep it consistent across the page.
The template has classes for each.

| kind | class | label example |
|---|---|---|
| props down | `edge-props` | `appointment={id: 812, status: "booked"}` |
| callback up | `edge-callback` | `onCancel(812)` |
| context read | `edge-context` | `useAuth() → {role: "provider"}` |
| hook / store | `edge-store` | `useAppointments() → [3 items]` |
| network fetch | `edge-fetch` | `GET /api/appointments/?date=2026-09-28` |
| URL / search params | `edge-url` | `?tab=upcoming` |

For each edge that the PR *changed* (new prop, different shape, removed
callback), show old and new values in the label, e.g.
`status: "booked"` → `status: "confirmed"`.

## 3. Render it

Use the nested-list tree in the template (`.tree`). Each node is an `<li>`
with a `.node` card holding:

- the component name, with a badge for **changed**, **new**, or **removed**
- the file path in small monospace text
- a `server` or `client` tag
- an edge label chip describing what it receives from its parent

Right after the tree, add a **data-flow table** that traces one user action end
to end with example data. For example, a user clicks "Cancel" on appointment
812: `AppointmentCard.onCancel(812)`, then `AppointmentList.handleCancel`, then
`POST /api/appointments/812/cancel/`, then a 200 with `{status: "cancelled"}`,
then the React Query cache update, then `AppointmentCard` re-renders with a
grey badge. The table makes the arrows concrete.

For bug fixes, render the flow table twice (before and after), with the
diverging row highlighted. That pair is usually the most useful figure on the
page.

## 4. Mermaid is optional

A published claude.ai artifact renders `<pre class="mermaid">` natively, so a
Mermaid graph is fine there as a *supplement*. A standalone HTML file can't
render it without loading a script, so always keep the HTML tree as the
primary diagram.
