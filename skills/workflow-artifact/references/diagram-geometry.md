# Diagram geometry — where nodes go, how lines connect, how text fits

Two halves. **Topology** (where nodes go) is read at Stage 2, before the plan.
**Geometry** (how lines route and text fits) is read at Stage 3, alongside
`visual-spec.md`.

Every rule here is mechanical. None requires taste, and each one can fail a
specific check — which is the point. "Make it look clean" is not an instruction
an agent can obey. "Estimated text width 148px exceeds rect width 150px minus
30px padding" is.

Topology heuristics adapted from
[konraddzbik/architecture-diagram-skill](https://github.com/konraddzbik/architecture-diagram-skill);
geometry rules adapted from `svg-exemplar.md` and `design-qa.md` in
[rafaelolsr/archflow](https://github.com/rafaelolsr/archflow).

---

# TOPOLOGY — read at Stage 2

## Plan the topology before writing any SVG

Sketch which nodes exist, where they sit relative to each other, and which
flows connect which. **Avoiding wire crossings is the single biggest
readability win available.** It is nearly impossible to retrofit — a
crossing-free layout comes from placement, not from routing.

## The five-zone canvas

Left-to-right zoning against the 1240 canvas. Most systems fit:

| Zone | x range | Typical occupants |
|---|---|---|
| **Entry** | 0–10% (0–124) | user, browser, CLI, webhook source, external caller |
| **Edge** | 20–30% (248–372) | gateway, hook, auth proxy, load balancer, router |
| **Core** | 40–55% (496–682) | orchestrators, business logic, the agent, app servers |
| **Backends** | 60–75% (744–930) | DBs, queues, caches, third-party APIs |
| **Heavy** | 80–90% (992–1116) | LLMs, batch jobs, long-running compute |

Vertical placement is freer. Two heuristics:

- **Mainline centred; side concerns above and below.** The happy path runs
  horizontally through the middle. Optional things — caches, observability,
  dead-letter queues, audits — sit above or below it.
- **Reads above, writes below.** Mirrors how people scan system diagrams.

**One-shot and rare nodes go in a corner.** Cron jobs, seed scripts,
migrations — peripheral, so park them top-left or top-right and keep the
mainline clean. Unless the rare node *is* the subject, in which case it becomes
the protagonist and gets centre stage.

## Direction carries meaning

The reader's eye follows the request left to right. **A wire running
right-to-left should mean a response.** If a *request* runs right-to-left,
something is misplaced — move it, don't route around it.

## When two nodes share a role

Distinguish them by **label** (`Postgres · users` vs `Redis · session cache`,
never "Database"), by **vertical position** (source of truth lower, ephemeral
higher), and by **shape**, used consistently.

## Decomposition — when to split a diagram

| Signal | Action |
|---|---|
| More than ~8 boxes in one row | split into two stacked rows, or two diagrams |
| More than ~20 boxes total | split by layer, one diagram per layer |
| Two unrelated flows on one canvas | two diagrams — a shared canvas implies a shared flow |
| A node needs a paragraph to explain | give it a `+` expander, don't grow the box |
| Height passing ~970 | split, or move detail into expanders |

**Prefer more diagrams over one crowded diagram.** Each answers one question.

## One figure, one claim

Every diagram makes exactly one point, stated in its mono kicker and repeated
in its `aria-label`. If the kicker needs an "and", it is two diagrams.

```
   ✓  "WHERE THE TOKEN IS VALIDATED, AND THE ONE PATH THAT SKIPS IT"
      — one claim: there is a bypass. The "and" names the exception.

   ✗  "AUTH FLOW AND THE BILLING PIPELINE"
      — two subjects. Two diagrams.
```

## Draw the mechanism, not its name

A box labelled `cache` says less than the prose already did. What the prose
cannot say: the path a request takes through it, the two stores it sits
between, and the arrow that disappears when it is removed. Show what the
argument turns on. **Comparing two options?** Draw the difference — two
topologies side by side with the one edge that changes highlighted in
`--signal`. Two labelled boxes with nothing connecting them is a restated
option list, not a comparison.

## Match complexity to the stakes

A one-hop question is a three-box diagram. A migration that reroutes writes
through a queue needs the queue, the writer, the reader, and the ordering
arrow. Draw as much as the decision actually turns on — no forced minimalism,
no inventory of the whole system either.

---

# GEOMETRY — read at Stage 3

## 1 · Orthogonal only — no curves, no diagonals

```
   ALLOWED                          FORBIDDEN
   M  move                          Q  quadratic curve  — never, not even
   H  horizontal                       for "cosmetic rounding"
   V  vertical                      C  cubic bezier     — never
   L  where EITHER x OR y changes   A  arc              — never
                                    L  where BOTH x and y change — never
                                       e.g. d="M200,100 L500,300"
```

Every arrow is a clean staircase of horizontal and vertical segments. A single
diagonal is what makes a generated diagram read as sloppy. Every path or line
ends in `marker-end="url(#…)"`.

## 2 · Plan the grid in a comment BEFORE drawing

Write the layout grid as an SVG comment at the top of each diagram. This forces
placement to be decided rather than accumulated, and makes the drawing
auditable against its own stated grid.

```xml
<!--
  LAYOUT GRID
  Spine row:    y=120..300   (arrow y=210)
  Gap:          y=300..370
  Branch row:   y=370..500
  Columns:      Entry 20..160 | gap | Edge 270..460 | gap | Core 580..780
  Verify:       every branch arrow drops straight down, no crossings
-->
```

**Rows first** — spine row, a 40–60px gap, branch row; all spine arrows on one
shared `ARROW Y`. **Columns second** — every element left to right with its x
range; **each inter-node gap is 80–120px**, enough for the arrow *and* its
label. **Branches third** — a branch group sits directly below its parent, or
to the right of the rightmost spine group; never between two spine groups, and
never overlapping another branch's vertical corridor. **Route fourth** — spine
arrows straight horizontal; branch-down straight vertical; a branch not
directly below its parent gets a single L-turn. Branch-returns route *outside*
all branch groups. **Verify fifth** — does any arrow pass through a branch it
does not serve? Rearrange until every branch has a clear corridor.

## 3 · Collision avoidance — the critical rule

**An arrow must never pass through a box it does not connect to.** Before
writing any path, trace the route and ask: *does this cross a rect I did not
intend to connect?*

```
   ✗ BAD — the horizontal run at y=310 crosses everything between x=240 and 620
       d="M620,260 L620,310 L240,310 L240,410"

   ✓ GOOD — branch moved directly below its parent, straight drop
       d="M240,260 L240,340"
```

Three fixes, in order of preference:

| Fix | When |
|---|---|
| **Rearrange** — move the branch directly below its parent | almost always cleanest; try first |
| **Route below** — down past the blocker's bottom edge, across, back up | when placement is fixed |
| **Route above** — same, using the top edge | when below is congested |

**Clearance: 15px minimum** from any box edge an arrow passes near but does
not connect to. Less and the reader cannot tell whether it touches.

## 4 · Arrow labels — beside the line, never on it

Every arrow is labelled (standing rule). Where the label goes is geometry:

| Arrow direction | Label placement |
|---|---|
| **Horizontal** | centred in the gap between the two boxes, y **10–15px above** the arrow's y |
| **Vertical** | **left** of the line, `text-anchor="end"`, x offset **15px** left of the arrow's x |

Two constraints, both checkable:

1. The label's full width must sit **inside the inter-box gap**. For
   `text-anchor="middle"`, estimated half-width is `chars × font-size × 0.30`.
2. The label's y must not coincide with the arrow's y over the same x range.

A gap too narrow for its label is the single most common defect in generated
diagrams. **Widen the gap; do not shrink the label to fit.**

## 5 · Text containment — arithmetic, not judgment

**Size the box to the text, never the text to the box.**

```
   estimated width  =  characters × font-size × 0.55   (mono)
                    =  characters × font-size × 0.50   (sans)

   rect width  ≥  estimated width + 30px padding
```

Worked: `"VectorStoreIndex"` at 11.5px sans → 16 × 11.5 × 0.50 = **92px** → the
rect needs **≥122px**. A 100px rect overflows and is a defect.

If the text does not fit, in order: **shorten the label** · **widen the rect**
(and shift everything downstream) · **split to two lines** with `<tspan>`.
Past ~92% of the box width, give the node a `+` expander instead of a longer
label. **Never truncate hoping it fits. Do the multiplication.**

SVG `<text>` does not wrap — multi-line means `<tspan>` with explicit `x` and
`dy` (`dy="12"` for 8–9px · `"14"` for 9–10px · `"16"` for 10–12px).

**Vertical fit:** top padding (12px) + title line (14px) + gap (6px) + detail
lines × line-height + bottom padding (12px). The standard two-line box —
bold 12 title + 10.5 detail — lands at **44–52px** tall.

**Nested rects — four inequalities, every time:**

```
   inner.x                ≥  outer.x + padding
   inner.x + inner.width  ≤  outer.x + outer.width  - padding
   inner.y                ≥  outer.y + header-space
   inner.y + inner.height ≤  outer.y + outer.height - padding
```

## 6 · Sizing conventions

| Element | Size |
|---|---|
| viewBox | **1240** wide (house standard); height free |
| Standard box | 200–300 wide, 44–52 tall |
| Entry / terminal box | 140–300 wide, 44–60 tall |
| Horizontal gap between boxes | **80–120px** — arrow plus label |
| Vertical gap between rows | 40–60px |
| `+` expander | 12×12, `rx=2` |
| SVG font floor | 8.5px — never below |

## 7 · Grid discipline

Shared baselines and even gaps are most of what makes a hand-authored diagram
read as deliberate. Align box tops and lefts to a consistent step (20px works
against 1240) · equal gaps between siblings · text baselines at the same offset
from the box top throughout · arrow-label baselines consistent relative to
their line.

## 8 · Construction checklist — run before handing over

1. **Text fit** — every `<text>` inside a rect: `chars × size × 0.50` (sans) or
   `× 0.55` (mono) under `rect width − 30`? Do the multiplication.
2. **Arrow clearance** — trace every path against every rect. Crossing anything
   it does not connect to? 15px clear of what it passes?
3. **Box overlap** — do any two rects share coordinate space unless nested by
   design?
4. **Label containment** — every arrow label inside its gap, off the line?
5. **Orthogonality** — any `L` changing both x and y? Any `Q`, `C`, `A`?
6. **Vertical budget** — does the column stack fit the viewBox height?
7. **Grid comment** — does the drawing match the grid it declared?
8. **No orphans** — is every `<g>` connected by at least one arrow?
