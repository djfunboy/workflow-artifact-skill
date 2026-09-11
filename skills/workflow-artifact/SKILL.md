---
name: workflow-artifact
license: MIT
description: "USE THIS whenever the user asks for a workflow, process flow, architecture doc, system map, config map, pipeline, or explainer as an HTML page, file, document, artifact, page, or something they can open — including phrasings like 'workflow html', 'give me a workflow on X', 'html of how X works', 'map out X', 'document how X works', 'architecture html', 'show me the process for X', or 'build me a doc explaining X'. Builds a complete tabbed multi-section HTML presentation — hand-authored SVG diagrams, tables, prose, click-to-expand explainers — written to disk and opened in the browser. The word 'file', 'html' or 'page' anywhere in the request is a trigger. If it is ambiguous whether the user wants a file or a diagram in the chat reply, Stage 0 asks — ambiguity is a reason to invoke and ask, not a reason to skip. Only skip when they have explicitly said they want it in the chat reply."
---

# workflow-artifact — complete HTML workflow presentations

## The one thing to get right

**The deliverable is the DIAGRAMS AND TABLES. Prose is the caption.** A wall of
text is a failure of this skill, no matter how accurate it is.

The value the document creates is **revelation**: the reader looks at it and
says "wait — I thought that step came *after* this one" or "I didn't realize
there were eight files involved." Everything below serves that moment. A
document that only confirms what the reader already believed was not worth
building.

### The format rules

1. Answer in **workflows, relationship maps, decision trees, and tables**.
2. **Never a wall of text.** Prose that could have been a table is a defect.
3. **Solid arrows for flow, dashed `5 3` for reference or a broken path.**
4. **`✗` where a flow is broken.**
5. **Callout boxes beside the flow, not in it.**
6. **Tables carry rationale and recommendation, not just facts.** A table of
   bare facts is half a table — add the "so what" column.
7. **Box every node. Boxes stay small.** The diagram runs as long as it needs
   but never wider than the page — vertical growth is free, horizontal
   scrolling is a defect. Hold the `1240` viewBox width and grow downward.
8. **Parallel things side by side, branches as side-by-side boxes, every arrow
   labelled.**

Aim for roughly **120 words of caption prose per diagram** — guidance, never a
gate. Go over when the subject needs it; what is forbidden is a wall of text,
not a word count. Detail that will not fit a box goes in a `+` expander —
uncapped, and where the depth lives.

### Depth is resolution, not volume

**Measured in steps, triggers and dependencies — never in words.** A document
that traces every flow to leaf, names every trigger and dependency, and puts
the mechanism behind a `+` on every full node is deep — at whatever length that
takes. One that shows five boxes is shallow, however many words surround it.

---

## Stage 0 — the two gates

### Gate A: file or chat?

The word "workflow" over-triggers. Before anything else, ask:

> Do you want this as a separate HTML file, or a workflow here in the chat?

```
   "in the chat"  ──►  STOP. Exit this skill. Draw it in the reply with
                       text boxes and arrows. Nothing else.

   "file"         ──►  continue to Gate B
```

Never skip this and never guess from the wording of the request.

### Gate B: codebase, or a described process?

> Am I reading a repo, or are you describing a process to me?

```
   ┌──────────────────────────┐        ┌──────────────────────────┐
   │ REPO / CODEBASE          │        │ PROCESS                  │
   │ "map how auth works"     │        │ "explain how the config  │
   │                          │        │  loads", "draw the       │
   │                          │        │  release flow"           │
   └────────────┬─────────────┘        └────────────┬─────────────┘
                │                                   │
                ▼                                   ▼
        Stage 1 = SCAN                      Stage 1 = INTERVIEW
                └──────────────┬────────────────────┘
                               ▼
                 identical from Stage 2 onward
```

Ask once. Do not ask again later in the same run.

---

## Stage 1 — GATHER

### 1-SCAN (codebase path)

**Enumerate, do not sample.** Map the directory, read **every** entry point and
orchestration file, and trace **every** flow end to end — not one
representative one. Extract components, boundaries, data flows, external
services, and the **real names** — class, module, file, method, service. Never
a generic placeholder: `Postgres · users`, not "Database".

Decompose each flow until every step passes the **leaf test**:

```
   A step is a LEAF when it names ONE mechanism you could point at in
   a file:  one function · one hook registration · one config key ·
            one cron line · one API call · one file write.

   Still contains an "and" or a "then"?  ->  not a leaf. Split it again.
```

### 1-INTERVIEW (process path)

**Read before asking.** If the user named a doc, repo, runbook, or folder, read
it first and extract everything it already answers. Then ask **only the gaps**,
one question at a time, each with a recommended answer.

Gaps worth asking about: what triggers the process · what the terminal states
are (success *and* failure) · which steps are automated vs. manual · what
enforces each step · where it is known to break.

### The INVENTORY — what Stage 1 produces

Seven rows, enumerated for **every** flow, built as you go. A row you never
wrote down is a row that silently vanishes from the document.

| Row | Capture | The question it answers |
|---|---|---|
| **STEPS** | every discrete step, decomposed to leaf | what actually happens |
| **TRIGGERS** | cron · hook · webhook · user action · file watch · event · manual | what makes it start |
| **DEPENDENCIES** | env var · secret · service · binary · file · permission · network | what it needs to work |
| **BRANCHES** | every conditional, **both** sides, each labelled | where it can go two ways |
| **FAILURE PATHS** | what happens when each step fails, and then what | what breaks |
| **HANDOFFS** | where control crosses process · machine · service · human | where it leaves one system |
| **STATE** | what is written, read, persisted, and where | what survives the run |

A row that genuinely does not apply is said so, not padded with filler.

### The floor — cover every flow, cite everything, guess nothing

Output floors can be met by padding. These cannot — they demand things you
have to go and find:

- **Every flow traced, not one.** Six entry points means six flows on the page.
- **Both sides of every branch.** A conditional drawn as one arrow is a
  missing branch.
- **Every flow's failure path investigated**, not assumed.
- **A `+` expander on every node with more to say than fits its box.**
- **Cite it.** Never assert from a name, a convention, or an inference — open
  the file. "Probably calls" → the line that calls it. "Likely triggered by" →
  the cron entry or hook registration, quoted. Every node and every arrow
  traces to a file and line, a command output, or a quoted sentence.
- **Unknowns are listed explicitly.** A named unknown beats a confident guess.
- **Contradictions between sources are surfaced**, not silently resolved.
  Where docs and live state disagree, say so — usually the most valuable
  finding on the page.

**End Stage 1 by printing what you found**, so the user can correct a wrong
premise before any HTML exists:

```
GATHER  ✓  {N} sources · {M} steps (cited) · {K}/{K} flows · {T} triggers
           {D} deps · {B} branches · {F} failure paths · {U} unknowns
           {C} contradictions
```

---

## Stage 2 — PLAN

Read the **Topology** half of `references/diagram-geometry.md`.

**Size the document to the subject.** A three-flow system gets a small page; a
multi-subsystem one gets tabs. Padding a small subject out to look substantial
is as bad as leaving steps out. **There are no size gates** — no word floor, no
section count, no diagram-to-prose ratio.

**Resolution never scales down with size.** A small document still traces every
flow to leaf, still names every trigger and dependency, still draws both sides
of every branch.

**Do not force symmetry across tabs.** One tab usually carries the weight — put
the depth where the subject actually is, and let the others be short.

Produce a **visible outline and stop for the user:**

```
PLAN
  Tabs:      {tab 1} · {tab 2}
  Spine:     {entry} → {…} → {exit}
  SECTIONS   1. {title} [diagram|table|prose]   2. {…}
```

### Section vocabulary — vary the shape

A page of identical prose-under-diagram sections reads as one long grey column.
Rotate through these. **No shape three times consecutively.**

| Shape | Use it for |
|---|---|
| **Diagram + prose** | the mechanism itself — the spine of the document |
| **Table** | anything with 3+ parallel cases: routing rules, comparisons, file inventories. Rationale and recommendation, not just facts |
| **Decision walk-through** | prose that walks each branch of a diagram in order, naming what happens and what enforces it |
| **Honest-state / verdict** | what works, what does not, and the evidence |
| **Inventory list** | files, hooks, endpoints — with a one-line "what it is" each |
| **Verbatim quote block** | an error, a spec line, a rule — quoted exactly |
| **Rejected alternatives** | what was considered and why it lost. High value, almost always omitted |
| **Key identifiers table** | IDs, paths, versions — the things someone needs to act |

### Honest state — required, at least one per tab

A presentation that only shows the intended design is worth one read. One that
says where the design fails is worth returning to. **Draw the broken path with
`✗` and say so in prose.**

```
   VERDICT_OK   ──►  chip "working"
   VERDICT_MID  ──►  chip "partly"
   VERDICT_BAD  ──►  chip "not working"
```

### Failure modes — what a FAILED document looks like

```
   ┌────────────────────────────────┐   ┌────────────────────────────────┐
   │ ✗ BIG-PICTURE ONLY             │   │ ✗ CAPTIONED PICTURES           │
   │   5 boxes for a 34-step flow.  │   │   Each diagram gets two lines  │
   │   No triggers, deps, branches  │   │   restating what the boxes     │
   │   or expanders.                │   │   already say. Adds no fact.   │
   └────────────────────────────────┘   └────────────────────────────────┘

   ┌────────────────────────────────┐   ┌────────────────────────────────┐
   │ ✗ HAPPY PATH ONLY              │   │ ✗ GENERIC NODES                │
   │   No failure state, no ✗, no   │   │   "Service", "Database",       │
   │   honest verdict. Decoration.  │   │   "API". True of any system,   │
   │                                │   │   therefore useless.           │
   └────────────────────────────────┘   └────────────────────────────────┘

   ┌────────────────────────────────┐   ┌────────────────────────────────┐
   │ ✗ ONE FLAT SCROLL              │   │ ✗ SYMMETRY FOR ITS OWN SAKE    │
   │   A multi-layer subject on one │   │   Three tabs padded to equal   │
   │   flat page, so everything     │   │   length. Put the depth where  │
   │   competes. A small subject on │   │   the subject actually is.     │
   │   one page is correct.         │   │                                │
   └────────────────────────────────┘   └────────────────────────────────┘
```

---

## Stage 3 — BUILD

Read `references/visual-spec.md`, the **Geometry** half of
`references/diagram-geometry.md`, and `assets/tokens.css`.

- **One self-contained HTML file.** No CDN, no external stylesheet, no runtime
  library. It must render correctly by double-clicking it with the network off.
- Hand-authored inline SVG only. **No Mermaid, no D3, no library.**
- **The look is hard-coded in this skill. Do not look any of it up.** Colours,
  fonts, box shapes, line styles and dash patterns are literal values in
  `assets/tokens.css` and `references/visual-spec.md`. Paste `assets/tokens.css`
  in verbatim as the `:root` block and use the token names.
- **Green is the page colour; amber never leaves the inside of an SVG.** Tabs,
  section numbers, links, the drawer border, the `+` expander — all `--durable`.
  Squint at the finished page: the colour you see most must be green.
- **The expander drawer opens at the end of its own section**, not at the bottom
  of the page.
- Light and dark both work. Responsive. `prefers-reduced-motion` respected.

Write the sections in outline order. **Do not write all the diagrams first and
backfill prose** — that is how the document ends up as a diagram gallery.

```
BUILD   ✓  {lines} lines, {sections} sections, {D} diagrams
```

---

## Stage 4 — DELIVER

**Propose the path — never ask blindly, never write to one the user has not
seen.** Name a concrete destination for each reading and let them accept in one
word:

> permanent → `<where the subject's own docs already live>/<real-name>.html`
> one-off   → a scratch or temp directory, out of the repo

Derive the permanent suggestion from **where you were just reading**. That is
the choice being made: a one-off in a repo becomes stale documentation nobody
deletes; a permanent doc in temp gets swept.

**Then open it in the browser. This is mandatory, not optional:**

```bash
open "<absolute path>"        # macOS · use xdg-open on Linux, start on Windows
```

These documents are read in a browser, not in a terminal. A document the user
has to go find and open themselves is a document they will not read. Open it
before you report anything — and open it again after the Stage 5 fix pass, so
what is on screen is the repaired version, never a stale one.

```
DELIVER ✓  {path} — opened in browser
```

---

## Stage 5 — VERIFY & FIX (one pass, then done)

**Render it and look at the screenshots.** Reading the markup is not checking;
a document that was never rendered has not been checked.

### Document boilerplate — three lines, before anything else

Every document opens with these, above the `<title>`:

```html
<!doctype html>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
```

Without the doctype a browser may fall into quirks mode; without the charset an
em-dash and a `·` depend on the browser guessing UTF-8; without the viewport tag
a phone lays the page out at ~980px and the reader pans sideways. Stage 5 asks
for a 390px check — this is the line that makes it pass.

### Render — in order of preference

1. **Browser DevTools MCP**, if available: navigate to the `file://` URL and
   capture, at minimum: full page light · full page dark · **each tab's first
   viewport** (a blank or unstyled panel is invisible from the default tab) ·
   **one expander open** (catches a dead `data-k` and drawer overflow) · a
   ~390px viewport. **Read the console** — a silent JS throw kills every
   expander while the page still looks right.
2. **Headless Chrome from the shell:**

```bash
CHROME="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"  # or google-chrome / chromium
"$CHROME" --headless --disable-gpu --hide-scrollbars \
  --window-size=1440,2400 --screenshot=/tmp/wf-light.png "file://<encoded path>"
# dark: add --force-dark-mode, AND verify data-theme="dark" separately —
# they exercise different CSS guards and a document can pass one and fail the other
# tabs and expanders: headless cannot click — render each tab by temporarily
# toggling the hidden attributes, or fall back to the wiring checks below
```

3. **Neither available** → say exactly: **"UNVERIFIED — the document was not
   rendered."** Do not soften it, do not say "should render correctly", and do
   not call the work done.

**Never judge horizontal overflow from a narrow screenshot.** Headless Chrome
floors its layout viewport at roughly 485px, so `--window-size=390` lays the
page out at 485 and crops the image to 390. A perfectly healthy page then looks
clipped, and the clipping is in the capture, not the document. Measure it
instead — inject a probe on `load` and compare:

```js
document.documentElement.scrollWidth   // vs
document.documentElement.clientWidth   // equal ⇒ no sideways scroll
```

Read it back with `--virtual-time-budget=2500 --dump-dom`.

### Look for — the things markup review cannot catch

| Defect | Why it slips through |
|---|---|
| text past a box edge, or sitting on a line | the coordinates looked plausible |
| two labels touching, arrowhead through text | ditto |
| a line stopping short, or pointing at nothing | the path data parsed |
| something invisible in dark mode | the token was only defined for light |
| an empty tab panel or a drawer with nothing in it | the markup was there |
| the page scrolling sideways | fits at your width, not the reader's |
| console errors | a silent throw, page still looks right |

Also check the wiring: every `.exp` `data-k` has a store entry and vice versa;
every tab's `aria-controls` resolves to a panel with content; exactly one panel
visible on load; tabs navigable by arrow keys; every `<svg>` has `role="img"`
and an `aria-label` stating the diagram's claim.

### Fix — one pass, not a loop

Fix what is clearly broken. **Surgical edits, one per finding — never a
redesign.** Re-render once to confirm the fixes landed, re-open the file in the
browser, and stop. Do not re-run the whole review; do not chase a clean
scorecard. The user's review is the real review — a second round spent
polishing a version they have not seen is effort spent on the wrong draft.

Anything still open goes to the user in one line each, with the document.

```
QA      ✓  rendered light + dark + mobile · {M} fixed · 0 console errors
           left for you: {one line each, or "nothing"}
```

---

## Stage 6 — PRESENT

```
WORKFLOW ARTIFACT: {subject}   ·   OPEN: {path}
  GATHER {N} sources · {M} steps · {U} unknowns · {C} contradictions
  BUILD  {sections} sections · {D} diagrams · {T} tables
  QA     rendered light + dark + mobile · {M} fixed · {K} left for you
```

Say plainly if the render could not be verified. Never imply it was.

---

## Standing rules

| Rule | Why |
|---|---|
| **No legend.** Ever. | Meaning rides on the arrow label, the colour and the dash; the mono kicker frames the diagram. |
| **Every arrow is labelled.** | An unlabelled arrow says "related somehow". `writes`, `invalidates`, `polls 30s` is information. |
| **Solid = flow. Dashed = reference or a broken path.** | Load-bearing, not decorative. |
| **Callouts sit beside the flow, never inside it.** | Dotted border, off the spine. |
| **No text overlaps a line or a box.** | Check before handing over. |
| **Real names only.** | `Developer ID Application (TEAM123456)`, not "the certificate". |
| **Say what is not working.** | A presentation that only shows the happy path is decoration. |

## Reference files

| File | Read at |
|---|---|
| `references/diagram-geometry.md` | Stage 2 (Topology half) and Stage 3 (Geometry half) — **always** |
| `references/visual-spec.md` | Stage 3 — tokens, type, boxes, tabs, expander, drawer |
| `assets/tokens.css` | Stage 3 — paste verbatim |

**If one of these files is missing**, say so in one line, carry on with what the
rest of this skill states, and report it at Stage 6 as **UNVERIFIED** for
whatever that file governs. Never proceed as though it was read, and never
reconstruct its contents from memory — the values in `tokens.css`,
`visual-spec.md` and `diagram-geometry.md` are literal and cannot be inferred.
