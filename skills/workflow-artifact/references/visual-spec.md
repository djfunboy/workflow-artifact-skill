# Visual spec — the exact idiom

Read at Stage 3. **Everything here is hard-coded. Do not look any of it up.**
No design-system document to consult, no other project to check, no brand file
to resolve. The colours, fonts, box shapes, line styles and dash patterns are
all stated below as literal values. Use them as written.

---

## 1 · Colour

Paste `assets/tokens.css` in verbatim as the `:root` block.

### Where each colour goes — get this right before anything else

**Green is the page colour. Amber never leaves the inside of an SVG.**

| Where | Token | Examples |
|---|---|---|
| **Page chrome** | **`--durable` — green** | selected tab + its underline · section numbers · links · drawer border · `+` expander hover and `.on` state · focus rings · any accented heading |
| Inside a diagram | `--signal` — amber | the decision point · the hop under discussion — **and nothing else** |
| Inside a diagram | `--danger` — red | the broken path · `✗` · failure branches |
| Inside a diagram | `--durable` — green | the path that holds · YES branches · verified state |
| Inside a diagram | `--muted` | dotted callouts, kickers, secondary labels |
| Inside a diagram | `currentColor` | neutral structure — inherits the theme, works light and dark |

**Squint test before shipping: the colour you see most on the page is GREEN.**
If the page reads amber or brown, the assignment is wrong — not the values.

**Reserve colour for the thing that carries meaning.** A diagram where every
box is coloured has no emphasis left to spend — most boxes are `currentColor`.

---

## 2 · Type

```css
--sans: 'Inter', -apple-system, BlinkMacSystemFont, system-ui, sans-serif;
--mono: 'JetBrains Mono', ui-monospace, SFMono-Regular, Menlo, monospace;
```

Both have a **system fallback stack and no `@font-face`, no CDN link**. The
file must render offline; on a machine without these installed it falls back to
the system stack, which is correct and expected. Never add a Google Fonts
`<link>` — it breaks the offline rule for a cosmetic gain.

The typographic signature is **mono, uppercase, letter-spaced**. Use it for
kickers, tabs, labels and chips. Never for body prose.

### Section numbers — the large gradient numeral

Numbered sections carry a **large display numeral, not a small mono label.**
Do not substitute a badge, a circle, or a mono caption.

**The numeral HANGS in its own left column and everything else in the section
indents past it.** Heading text, paragraphs, figures, tables and callouts all
share one left edge; only the number sits outside it.

```css
/* the section body indents; the numeral is pulled back out into the margin */
[role="tabpanel"] { padding-left:84px; }

h2 {
  display:flex; align-items:baseline; gap:20px;
  margin-left:-84px;                /* hang the numeral */
  font-size:21px; font-weight:750; margin-top:48px;
}
h2 .num {
  flex-shrink:0;
  width:64px;                       /* fixed column so 1 and 20 align.
                                       MUST fit the widest numeral — see the
                                       two-digit trap below. */
  align-self:flex-start;
  font-family:var(--sans);          /* NOT mono */
  font-size:42px;
  font-weight:800;
  line-height:.86;
  letter-spacing:-0.03em;
  background:linear-gradient(135deg, #22C55E 0%, #4ADE80 100%);
  -webkit-background-clip:text;
  -webkit-text-fill-color:transparent;
  background-clip:text;
}

@media (max-width:700px){
  [role="tabpanel"] { padding-left:60px; }
  h2 { margin-left:-60px; gap:14px; }
  h2 .num { font-size:32px; width:46px; }
}
```

**Fixed-width number column** so section 1 and section 20 start their text at
the same x. Scale down on narrow viewports but never below 28px — the numeral
is structure, and shrinking it to caption size loses that.

**THE TWO-DIGIT TRAP — the column must be wider than the numeral, always.**
`background-clip:text` clips the gradient to the glyphs, and
`-webkit-text-fill-color:transparent` means the glyph has *no colour of its
own*. A gradient only paints inside the element's box. So any part of a numeral
that extends past `width` has nothing to clip and renders **invisible** — not
overflowing, not clipped at the edge, simply gone.

At `font-size:42px; font-weight:800`, two digits measure roughly 48px, so the
column must be wider than that. A numeral that overruns the column loses its
overrun silently — it looks like a font problem, not a box problem.

| Sections in the document | `h2 .num` width | `padding-left` / `margin-left` |
|---|---|---|
| 1–9 | 42px would do — **still use 64px** so the idiom is one value | 84px |
| 10–99 | **64px** | 84px |
| 100+ | 88px | 108px |

Use the two-digit column everywhere rather than sizing per document: a page
that grows from nine sections to ten otherwise breaks silently, and nobody is
watching for it.

---

## 3 · SVG diagram idiom

### Canvas

```
viewBox="0 0 1240 {H}"          H is free. No ceiling. Let it be tall.
role="img"
aria-label="{a full prose sentence stating what the diagram shows}"
```

Wrap every diagram: `<figure>` → `<svg>` → `<figcaption>`.
CSS: `max-width:100%; height:auto`.

### The kicker — every diagram opens with one

```xml
<text x="20" y="22" font-family="var(--mono)" font-size="10"
      letter-spacing="1.2" fill="var(--muted)">
  WHERE THE TOKEN IS VALIDATED, AND THE ONE PATH THAT SKIPS IT
</text>
```

It states the diagram's **claim**, not its topic. "AUTH FLOW" is a topic.
"WHERE THE TOKEN IS VALIDATED, AND THE ONE PATH THAT SKIPS IT" is a claim.

### Boxes

```xml
<!-- neutral: inherits the theme -->
<rect x="470" y="36" width="300" height="44" rx="6"
      fill="none" stroke="currentColor" stroke-width="1.8"/>

<!-- semantic: carries meaning -->
<rect x="380" y="104" width="480" height="52" rx="6"
      fill="var(--signal-bg)" stroke="var(--signal)" stroke-width="2"/>
```

`rx="6"` on every rect. Stroke 1.8 neutral, 2.0 semantic.

**Two lines of text per box** — a bold CAPS title and a regular detail line:

```xml
<text x="620" y="126" font-size="12" text-anchor="middle"
      fill="var(--signal)" font-weight="700">THE AGENT ASKS — NEVER WRITES UNASKED</text>
<text x="620" y="145" font-size="10.5" text-anchor="middle"
      fill="currentColor">"cover this with a test?"  ·  "add a check to the runbook?"</text>
```

A one-word box label is wrong. Boxes stay small; the diagram runs as wide and
as long as it needs.

### Font sizes inside SVG

| Size | Use |
|---|---|
| 12–13 | box titles |
| 10.5–11.5 | box detail lines, arrow labels *(the two most common sizes)* |
| 10 | the mono kicker |
| 9–9.5 | dense secondary labels |
| 8.5 | floor — do not go below |

### Arrows

Each diagram carries **its own** `<defs><marker>` with a local id:

```xml
<defs>
  <marker id="fb" viewBox="0 0 10 10" refX="9" refY="5"
          markerWidth="7" markerHeight="7" orient="auto-start-reverse">
    <path d="M0,0 L10,5 L0,10 z" fill="currentColor"/>
  </marker>
</defs>
```

Reference the id with `marker-end="url(#fb)"` — nearly every line is directed.

**Branch outcome is carried by colour AND dash together, labelled on the arrow:**

```xml
<!-- YES: solid, durable -->
<line x1="280" y1="172" x2="280" y2="188" stroke="var(--durable)"
      stroke-width="1.8" marker-end="url(#fb)"/>
<text x="300" y="168" font-size="11" fill="var(--durable)" font-weight="700">YES</text>

<!-- NO: dashed, danger -->
<line x1="960" y1="172" x2="960" y2="188" stroke="var(--danger)"
      stroke-width="1.8" stroke-dasharray="5 3" marker-end="url(#fb)"/>
```

`stroke-dasharray="5 3"` is the house dash. **There is no legend** — meaning
rides on the label, the colour and the dash.

---

## 4 · Tabs — how more fits on the page

```html
<div class="tabs" role="tablist" aria-label="Layers">
  <button class="tab" role="tab" id="tab-a" aria-controls="panel-a"
          aria-selected="true"  tabindex="0">HOW IT RUNS</button>
  <button class="tab" role="tab" id="tab-b" aria-controls="panel-b"
          aria-selected="false" tabindex="-1">HONEST STATE</button>
</div>
<div id="panel-a">…</div>
<div id="panel-b" hidden>…</div>
```

```css
.tabs { display:flex; gap:2px; margin:30px 0 0; border-bottom:1px solid var(--rule); }
.tab  { appearance:none; background:none; border:0; cursor:pointer;
        font-family:var(--mono); font-size:12.5px; letter-spacing:.07em;
        text-transform:uppercase; color:var(--muted); padding:12px 18px;
        border-bottom:2px solid transparent; }
.tab:hover                   { color:var(--ink); }
.tab[aria-selected="true"]   { color:var(--durable); border-bottom-color:var(--durable); }
.tab:focus-visible           { outline:2px solid var(--durable); outline-offset:-2px;
                               border-radius:3px; }
@media (max-width:700px) { .tab { padding:11px 12px; font-size:11.5px; } }
```

Behaviour: click selects · `←`/`→` move and wrap, with `preventDefault` ·
roving `tabindex` (selected 0, others −1) · switching sets `aria-selected` and
toggles the panel's `hidden`.

**Tabs are the density device.** Three chapters on one page instead of three
files. Use them whenever the subject has more than one natural layer.

---

## 5 · The `+` expander — depth without crowding

This is what lets a box stay small while the detail stays on the page — the
mechanism that makes depth and brevity compatible. Expander content is
**uncapped**: the ~120-word caption target does not apply inside a drawer, and
this is where triggers, dependencies and error handling belong.

**Required, not optional.** Put a `+` on every node that has more to say than
fits in its box. A missing expander is the most common way real detail goes
missing from a document.

```xml
<g class="exp" data-k="claude">
  <rect x="424" y="66" width="12" height="12" rx="2"/>
  <text x="430" y="75" text-anchor="middle">+</text>
</g>
```

```css
.exp            { cursor:pointer; }
.exp rect       { fill:var(--panel); stroke:var(--muted); stroke-width:1;
                  transition:fill .12s, stroke .12s; }
.exp text       { font-family:var(--mono); font-size:9px; font-weight:700;
                  fill:var(--muted); pointer-events:none; }
.exp:hover rect,
.exp.on rect    { fill:var(--durable); stroke:var(--durable); }
.exp:hover text,
.exp.on text    { fill:#fff; }
```

**Green, not amber** — the expander is page chrome, not a diagram semantic.
`pointer-events:none` on the `<text>` matters — without it the glyph swallows
clicks meant for the rect.

---

## 6 · The drawer — one shared detail surface

**The drawer opens at the bottom of the section the expander lives in — never
at the bottom of the page.** One drawer element is reused, but on click it is
*moved* into the current section, immediately before the next `<h2>`. A reader
who clicks a `+` in section 1 must not be thrown to the end of the document.

This is the shipped, working version — the naive one breaks on SVG elements
(where `.closest` may be absent) and re-enters itself once the drawer is
already in the section. Copy it as written:

```js
// move the drawer to the end of the clicked expander's section
function placeDrawer(fromEl){
  var node = fromEl.closest ? fromEl.closest('figure') : null;
  if(!node){ node = fromEl; while(node && node.tagName!=='FIGURE') node = node.parentNode; }
  if(!node || !node.parentNode) return;
  while(node.nextElementSibling && node.nextElementSibling.tagName !== 'H2'
        && node.nextElementSibling !== drawer){
    node = node.nextElementSibling;
  }
  node.parentNode.insertBefore(drawer, node.nextSibling);
}
```

Call it **before** `render()` on every expander click, then
`drawer.scrollIntoView({block:'nearest', behavior:'smooth'})`.

Border colour is `--durable` (green) — the drawer is chrome. One drawer per
page; every expander renders into it.

```html
<div class="drawer" id="drawer" hidden>
  <div class="dh"><span class="dt" id="d-title"></span>
                  <span class="dk" id="d-key"></span></div>
  <div class="db" id="d-body"></div>
</div>
```

Content lives in a keyed store, **not** in the DOM:

```js
var D = {
  gateway: {
    t: "api-gateway",
    k: "services/gateway · entry point for every request",
    b: [
      ["What it is",     "The single ingress. …"],
      ["What's in it",   "Route table · auth middleware · rate limiter."],
      ["Honest state",   "VERDICT_MID Rate limits configured but not enforced on the admin routes: …"]
    ]
  }
};
```

Rendering rules:

- each `b` pair → a mono 10px CAPS heading (letter-spacing .11em) + body
- a body may be `{cols:[…], rows:[[…]]}` instead of a string → renders a table
- `VERDICT_OK` / `VERDICT_MID` / `VERDICT_BAD` anywhere in a body string are
  substituted for inline chips reading **working** / **partly** / **not working**
- escape `&` and `<` in all body text before inserting

```css
.verdict { display:inline-block; font-family:var(--mono); font-size:10px;
           letter-spacing:.08em; text-transform:uppercase; font-weight:700;
           padding:2px 8px; border-radius:4px; }
.v-ok  { background:var(--durable-bg); color:var(--durable); }
.v-mid { background:var(--signal-bg);  color:var(--signal); }
.v-bad { background:var(--danger-bg);  color:var(--danger); }
```

Interaction: click an expander → mark it `.on`, render, unhide the drawer,
`scrollIntoView({block:'nearest', behavior:'smooth'})`. Clicking the same
expander again closes it. Opening a different one clears the previous `.on`.

---

## 7 · Page furniture

### Widths — prose must not stop short of the diagrams

```css
.wrap { max-width:1600px; margin:0 auto; padding:40px 32px 96px; }
p, li, figcaption, .callout, .drawer .db > div { max-width:100ch; }
```

**`100ch`, not 80ch.** A 1240-wide diagram in a 1600 wrap renders ~1250px.
Prose capped at 80–85ch stops around 940px, leaving a 300px dead gutter beside
every paragraph while the diagrams run full width. 100ch is still inside
readable measure at this type size.

| Element | Spec |
|---|---|
| Body background | `var(--ground)`, explicitly set — never transparent |
| Card / panel | `var(--panel)`, 1px `var(--rule)`, `border-radius:8px` |
| Wide tables | wrap in `.tablewrap { overflow-x:auto }` — the page body never scrolls sideways |
| Section heads | `<h2>` sans; `<h3>` for subsections |
| Labels, chips, kickers | mono, uppercase, letter-spaced |
| Callouts | dotted `var(--rule)` border, 3px solid accent left edge, beside the flow |
| Theme | full light palette on bare `:root`; dark under both `@media (prefers-color-scheme: dark)` guarded as `:root:not([data-theme="light"])` **and** `:root[data-theme="dark"]` |
| Motion | everything behind `prefers-reduced-motion` |
