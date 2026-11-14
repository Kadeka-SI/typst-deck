---
name: typst-presentation
description: Use when asked to create a presentation, slide deck, or polished paginated document with Typst. Covers Typst 0.15+, delimiter-safe nested content, syntax-error diagnosis, reliable slide layout, visual design, assets, CLI workflow, source attribution, and quality assurance.
---

# Typst Presentations

> **First-load instruction:** The workflow is simple: create a project folder, write the presentation to `main.typ`, and keep any images or other assets inside that folder. The user can then run `typst watch main.typ` in a terminal from that folder; Typst creates `main.pdf` and recompiles it whenever a source file changes. The user can stop watching with Ctrl+C. Explain this briefly in the user's language, then proceed with the task.

## 1. What Typst is and what this skill targets

Typst is a markup language, compiler, and web app for authoring documents. A `.typ` file is plain text; the compiler can produce PDF, PNG, SVG, and experimental HTML output. A basic Typst presentation is a paginated document in which each landscape PDF page is one slide. The core language does not provide a separate mandatory slide mode.

**Version target:** This skill targets Typst **0.15.0 or newer**. Its CLI examples and language guidance are aligned with the Typst 0.15.1 documentation. Check the installed version with `typst --version` and check the locally supported commands with `typst help` and `typst help compile` / `typst help watch`. Do not assume that a command-line option exists on older releases.

The standard template below is package-free and uses no external image assets. It should therefore be a good starting point for a clean Typst 0.15+ project. Still compile every generated deck in the actual target environment; never claim that code has been compiler-tested unless it was run successfully.

Typst is well suited to reproducible, vector-based slides, tables, diagrams, and PDF handouts. A simple Typst PDF deck is static: do not promise slide animations, click-through interactions, or speaker notes unless a suitable package or separate presentation system is deliberately used and tested.

## 2. Project workflow

1. Create a project folder, for example:
   ```bash
   mkdir my-deck
   cd my-deck
   mkdir -p assets preview
   ```
2. Write the presentation to `main.typ`. Put every image, logo, or other file the document needs inside the project folder, usually in `assets/`.
3. Compile once:
   ```bash
   typst compile main.typ
   ```
   This creates `main.pdf` in the current folder.
4. For live recompilation:
   ```bash
   typst watch main.typ
   ```
   Save changes to `main.typ` or imported assets; Typst recompiles the output. Stop the watcher with Ctrl+C.
5. Open the PDF and inspect it. For visual QA, export every page as an image (see §12), not just the first page.

For a one-shot compile with an explicit output name:

```bash
typst compile main.typ deck.pdf
```

To export every page as a separate PNG:

```bash
mkdir -p preview
typst compile --format png --ppi 160 main.typ "preview/slide-{0p}.png"
```

`{p}` is replaced by the page number, `{0p}` by a zero-padded page number, and `{t}` by the total page count. Multi-page PNG and SVG exports require a page-number placeholder in the output path.

### Common CLI options

Use `typst help compile` for the exact options supported by the installed compiler. Common options include:

| Option | Purpose |
|---|---|
| `-f, --format` | Select an output format such as `pdf`, `png`, `svg`, or `html`. It is usually inferred from the output extension. |
| `--pages 2,3-6` | Export only selected pages or ranges. |
| `--root DIR` | Set the project root used to resolve absolute paths. |
| `--font-path DIR` | Add a directory containing fonts. |
| `--input key=value` | Make a string input available through `sys.inputs`. |
| `-j, --jobs N` | Set the number of parallel compilation jobs. |
| `--ppi 160` | Set PNG export resolution; the default is 144 PPI. |
| `--open [VIEWER]` | Open the output after compilation. |
| `--pretty` | Produce human-readable output for supported textual/vector formats. Added in Typst 0.15; it does not affect PNG output. |

Compile and watch share many options, but **do not assume their option sets are identical**. Check `typst help watch` for watch-specific options.

Other useful commands:

```bash
typst --version
typst fonts
typst eval '1 + 1'
typst help
typst help compile
typst help watch
```

`typst eval` and the current `--pretty` option are available in Typst 0.15+. `typst fonts` lists fonts discovered by the installed compiler. Run it before choosing a font.

### HTML export is experimental

HTML export is still an experimental feature and is not suitable as the default production deliverable. On the CLI, it requires the feature flag:

```bash
typst compile --features html --format html main.typ main.html
```

Do not choose HTML unless the user specifically needs it and the limitations are acceptable. PDF should remain the default for ordinary slide decks.

### Typst web app presentation mode

The Typst web app can detect 16:9 and 4:3 page ratios and show a Present control. The built-in presentation mode is a Typst Pro feature. Do not promise this control or feature to users who are working only with the free local CLI.

## 3. Typst language basics

### Markup

```typ
Normal text forms paragraphs. A blank line starts a new paragraph.
*bold*  _emphasis_  ~highlight~
A "smart quote" becomes typographic quotation marks.
-- is an en dash; --- is an em dash.
A literal currency amount is written as \$100.

- Unordered list item
+ Numbered list item
/ Definition term: its definition

= Heading level 1
== Heading level 2

#raw("inline code")

#link("https://typst.app/docs")[Typst documentation]
```

A literal dollar sign must be escaped as `\$` in markup. Do not write `$100` as ordinary text: `$` begins math mode. For example, use `\$100` to display `$100`.

Do not write `\n` in markup to force a line break. `\n` is an escape sequence for strings, not a markup line-break command. In markup, use `#linebreak()` or a backslash followed by whitespace; use a blank line when a new paragraph is intended.

### Syntax modes

Typst has **three syntax modes**:

1. **Markup mode** is the default and is used for ordinary text and document structure.
2. **Code mode** is entered with `#` and is used for expressions, variables, functions, and control flow.
3. **Math mode** is entered and exited with `$` and is used for mathematical notation.

Examples:

```typ
#let x = 5
The value is #(x + 1).
#text(size: 20pt)[This is content passed to a function.]

Inline math: $x^2 + y^2 = z^2$.
Display math:
$ x^2 + y^2 = z^2 $
```

When an expression is introduced with `#`, Typst remains in code mode until the expression ends. Use square brackets `[...]` to create content and return to markup within a code expression.

### Content, functions, and arguments

Content is written in square brackets. In a normal function call, use parentheses and separate arguments with commas. Named arguments must use their parameter names.

```typ
#let simple-card(title, body) = [
  #text(weight: "bold")[#title]
  #v(6pt)
  #body
]

#simple-card([A title], [A short body.])
```

For templates, prefer explicit, unambiguous function signatures and calls. A frequent source of errors is mixing markup-mode content syntax with code-mode function-call syntax. For example, `#f[x], y: [z]` ends the call before the comma; use `#f([x], y: [z])` instead.

When a function has optional named parameters, put them in the function call's parentheses and pass content in the documented position. Do not guess whether adjacent content blocks are separate arguments: inside a code-mode call, separate content arguments with commas.

### Safe call pattern for the `slide()` helper

The starter template defines the helper as `#let slide(title, body) = ...`. It therefore expects **two positional arguments**: a title content block and a body content block. Use this exact multiline shape for every slide:

```typ
#slide(
  [A concise slide title],
  [
    // All slide content goes here.
    #text(size: 20pt)[Example body text]
  ],
)
```

The delimiters have distinct jobs:

- `#slide(` opens the function call and must be matched by the final `)`.
- `[A concise slide title]` is the first argument and must close before the comma.
- The second `[` opens the body content and must be matched by `]` before the final `)`.
- Nested calls such as `#grid(...)`, `#card(...)`, and `#text(...)[...]` must each close independently.

Do not leave a slide body open when starting `#pagebreak()`, another `#slide(...)`, or the next section. A completed slide call ends with `],` followed by `)` (or `])` when written on one line). Avoid writing a long slide call on one line: the multiline form makes missing delimiters easier to spot.

**When the compiler reports `unclosed delimiter` at the `#slide(` line or at the body-opening `[`**, do not assume the opening line is wrong. The missing closing `]` or `)` is often later in the body. Inspect from that call to the next slide, counting nested `[`/`]` and `(`/`)` pairs. Check especially the last `#grid`, `#card`, `#figure`, `#align`, and `#text` call in the body. Do not insert a random closing bracket at the reported line; find the unmatched opening delimiter and close it at the intended structural boundary.

### Safe nesting for cards inside grid cells

Do not compress deeply nested content calls into a single line. In particular, a grid cell that contains a card with its own content block has **three separate closing delimiters**: first close the card's content block with `]`, then close the `#card(...)` call with `)`, then close the outer grid-cell content block with `]`. Use this expanded pattern:

```typ
#grid(
  columns: (1fr, 1fr),
  [
    #card(
      [
        #text(size: 18pt)[Card heading]
        #v(6pt)
        Card body text.
      ], // closes the content passed to #card
    )   // closes the #card(...) call
  ],    // closes this grid cell
  [Right-hand cell],
)
```

The following close order is wrong and can make the compiler report `unclosed delimiter` at a much earlier `#slide(` line:

```typ
[#card([
  Card body text.
)] // WRONG: the function call is closed before its content block
]
```

The correct compact close order for `[#card([ ... ])]` is `])]` (content block, function call, outer cell). Prefer the expanded multiline form above so the nesting is visually obvious. Apply the same rule to nested `#rect`, `#figure`, `#align`, `#text`, and custom helper calls.

When repairing an error, inspect the **innermost** open call first. A compiler location at `#slide(` may only identify the outer expression that never became balanced; it does not prove that the `#slide` definition or opening line is wrong.

### Control flow

```typ
#let topics = ("Context", "Problem", "Solution")

#for topic in topics [
  - #topic
]

#if 2 > 1 [This branch is shown.] else [This branch is not shown.]

#let double(value) = value * 2
#double(21)
```

Loops that emit markup need a content block after the loop expression. In code-mode calls such as `grid(...)` and `table(...)`, separate content cells with commas; do not place adjacent `[...]` blocks where Typst could parse the second block as indexing.

### Common types and values

Useful types include strings (`str`), content (`content`, created with `[...]`), lengths (`16pt`, `1cm`), fractions (`1fr`), ratios (`100%`), colors (`rgb(...)`, `luma(...)`, and other color constructors), arrays, dictionaries, and functions. A percentage such as `100%` is a ratio relative to the relevant layout context, not a fixed physical length.

### Styling with `set` and `show`

```typ
#set text(size: 14pt, fill: rgb("#202124"))
#set par(justify: false, leading: 0.85em, spacing: 1em)
#set heading(numbering: "1.1")
#show heading: it => text(size: 22pt, weight: "bold", fill: rgb("#0f4c81"))[#it.body]
```

A `set` rule applies within its content scope. Put shared theme and page settings near the beginning of the document. When a style is defined inside a helper function or nested content block, verify the scope rather than assuming it changes the whole document.

## 4. Page setup: one page is one slide

Use a landscape page size. The built-in presentation formats are preferred:

```typ
#set page(paper: "presentation-16-9")
// Or:
#set page(paper: "presentation-4-3")
```

Other useful choices:

- Explicit 16:9: `width: 320mm, height: 180mm`.
- Explicit 3:2: `width: 240mm, height: 160mm`.
- A4 landscape: `paper: "a4", flipped: true` (better for handouts than projected slides).

Put the page `set` rule at the beginning of the document, before slide content. Changing page properties later can introduce a page break.

Example with a footer:

```typ
#let muted = rgb("#5f6b7a")
#let project_title = [Project title]

#set page(
  paper: "presentation-16-9",
  margin: (left: 18mm, right: 18mm, top: 14mm, bottom: 16mm),
  fill: rgb("#faf8f3"),
  footer: context [
    #set text(size: 9pt, fill: muted)
    #grid(
      columns: (1fr, auto),
      width: 100%,
      [#project_title],
      [#counter(page).display("1 / 1", both: true)],
    )
  ],
)
```

Define every variable used by a page header, footer, background, or foreground **before** the `#set page(...)` rule. This avoids scope and forward-reference problems.

### Page breaks and overflow

- Use `#pagebreak()` between slides when writing a normal paginated deck.
- `pagebreak` must not be placed inside containers such as `block`, `grid`, or `rect`; put it between top-level slide elements.
- Typst automatically paginates content. A slide with too much content can create extra pages or overflow its intended layout. A manual page break does not automatically guarantee that the content before it fits on exactly one page.
- A `block(breakable: false)` can prevent a block from splitting across pages, but it is not a substitute for checking the rendered result. If the block is taller than the available page area, it still requires a layout correction.
- Always count the rendered pages and compare them with the intended slide count. Investigate every unexpected extra page.

Use `counter(page)` for page numbering. Dynamic counter output in headers or footers should be placed in a `context [...]` block, as in the example above.

## 5. Layout and design toolbox

These built-in elements cover most slide needs without third-party packages:

```typ
// Filled card
#rect(
  width: 100%,
  fill: rgb("#e8f1fa"),
  stroke: 1pt + rgb("#0f4c81"),
  radius: 8pt,
  inset: 12pt,
)[Card content]

// Number badge
#circle(radius: 16pt, fill: rgb("#0f4c81"))[
  #align(center + horizon)[#text(size: 12pt, fill: white, weight: "bold")[1]]
]

// Vertical spacing
#v(12pt)
#v(1em, weak: true) // collapsible spacing

// Two columns
#grid(
  columns: (1fr, 1fr),
  gutter: 24pt,
  [Left column],
  [Right column],
)

// Three columns and two rows
#grid(
  columns: (1fr, 1fr, 1fr),
  rows: (auto, auto),
  column-gutter: 12pt,
  row-gutter: 12pt,
  [A], [B], [C],
  [D], [E], [F],
)

// Stacking/layering
#stack(dir: ttb, [Back layer], [Front layer])

// Table
#table(
  columns: 3,
  stroke: none,
  inset: (x: 12pt, y: 6pt),
  table.header([Topic], [Before], [After]),
  [Speed], [Slow], [Fast],
)

// Large statistic
#text(size: 40pt, weight: "bold", fill: rgb("#0f4c81"))[42%]
```

`grid` is for visual layout; `table` is for related data with tabular semantics. Use a table for actual comparisons or datasets, not simply because the content can be arranged in columns.

## 6. Typography and fonts

### Presentation typography

As a practical starting point for 16:9 slides:

- Body text: usually 18–24pt.
- Slide titles: usually 28–36pt.
- Cover titles: usually 40–52pt.
- Footnotes and source labels: aim for at least 11–12pt where possible.

These are starting ranges, not rigid rules. Short titles, room size, screen resolution, audience distance, language length, and slide density all matter. Prefer removing content over shrinking body text until it becomes difficult to read.

### Font selection

Always inspect available fonts before selecting one:

```bash
typst fonts
```

Use a family that actually appears in the output. The CLI has a small set of embedded fonts, including `Libertinus Serif`, `New Computer Modern`, and `DejaVu Sans Mono`; system fonts and custom font folders may provide more choices. Do not assume that fonts such as Noto Sans, Arial, Helvetica, or Segoe UI are installed on every machine.

For a portable baseline, use an embedded font such as:

```typ
#set text(font: "Libertinus Serif", size: 18pt)
```

If a sans-serif look is important, select a sans-serif family confirmed by `typst fonts`, or add the licensed `.ttf` / `.otf` files to a project font folder and compile with the appropriate `--font-path`. Keep the font files in the project only when licensing permits redistribution.

A list of font families can be used as a fallback priority list, but only include families known to exist in the target environment. Missing font families can produce warnings or lead to substituted glyphs. Do not ignore font warnings: check the PDF output for unexpected substitutions, missing glyphs, or changed line breaks.

## 7. Images and other assets

Keep all assets inside the project folder, preferably under `assets/`. Typst image paths are resolved relative to the project root (normally the folder containing `main.typ`). Check that each referenced file exists before compiling.

```typ
// Basic image
#image("assets/photo.jpg", width: 80%, alt: "A researcher examining a sample")

// Fixed image area, preserving the full image
#image(
  "assets/photo.jpg",
  width: 100%,
  height: 60%,
  fit: "contain",
  alt: "A researcher examining a sample",
)

// Fill the area and crop where necessary
#image("assets/hero.jpg", width: 100%, height: 65%, fit: "cover", alt: "A city skyline at dusk")

// Figure with caption and image description
#figure(
  image("assets/diagram.svg", width: 85%, alt: "System architecture showing three connected services"),
  caption: [System architecture overview],
)

// Embed a specific page of a PDF as an image
#image("assets/reference.pdf", page: 2, width: 90%, alt: "Second page of the reference diagram")
```

Supported image formats include PNG, JPG, GIF, SVG, PDF, and WebP. GIFs should be treated as static artwork in a normal PDF workflow; do not rely on animation. For PDF images, use `page: n` to select a page other than the first.

### Asset rules for the agent

- Prefer user-provided assets when they are relevant and usable.
- If an image must be sourced externally, verify that it is suitable for the intended use and that its licensing/usage conditions are acceptable. Do not present an unverified image as a genuine photograph of a named person, event, product, or place.
- Copy required local assets into the project and use relative paths. Typst does not automatically fetch ordinary remote image URLs in an `image("...")` call.
- Never leave placeholder paths such as `assets/example.png` in the final deck unless that file is actually included.
- If a suitable asset is unavailable, use a native Typst diagram or clearly tell the user which file they need to provide. Do not silently create a broken image reference.
- Write meaningful `alt` descriptions for informative images. Decorative images may use an empty description when that is semantically appropriate. When an image is inside a `figure`, generally put its description on the `image` element; use a figure-level description only when the figure is one combined semantic unit.
- Use SVG for crisp logos and diagrams, PNG for screenshots containing text, and JPG/WebP for photographs. Avoid stretching images unless distortion is intentional.

For full-bleed images, plan the page margin and image placement deliberately. Do not use negative offsets as a routine shortcut; make sure the final PDF has no accidental clipping or content outside the page.

## 8. Charts, tables, diagrams, and data

Use the simplest tool that produces an honest, legible visual:

- Use `table` for precise comparisons, specifications, or datasets.
- Use `grid`, `rect`, `line`, `circle`, and `stack` for simple diagrams and schematic illustrations.
- Use a suitable chart package only when a real chart is needed and the package's current documentation and version have been checked.

For every chart:

- Include a clear title, units, axis labels, and a time period where relevant.
- Label important values directly when that improves readability.
- Keep scales and baselines honest. In particular, do not truncate a bar chart axis in a way that exaggerates differences without clearly disclosing it.
- Use a consistent color meaning across slides.
- Cite the underlying source for externally sourced data and state the data date or period.
- Do not invent, interpolate, or silently alter data. Distinguish estimates, projections, and measured values.

Do not use a pie chart for many similar categories or values that are hard to distinguish. Prefer a bar chart or a compact table in those cases.

## 9. Package-free starter template

This starter template deliberately contains no external images and no package imports. It uses a built-in font and native Typst elements. Copy it to `main.typ`, then change the title, palette, and slide content. It targets Typst 0.15+; compile it in the target environment before using it as the base for a deliverable.

```typ
// ── Package-free presentation starter (Typst 0.15+) ───────────────

// Theme tokens: define these before page settings because the footer uses them.
#let accent = rgb("#0f4c81")
#let background = rgb("#faf8f3")
#let soft = rgb("#e8f1fa")
#let ink = rgb("#202124")
#let muted = rgb("#5f6b7a")
#let project_title = [Project Title]

#set page(
  paper: "presentation-16-9",
  margin: (left: 18mm, right: 18mm, top: 14mm, bottom: 16mm),
  fill: background,
  footer: context [
    #set text(size: 9pt, fill: muted)
    #grid(
      columns: (1fr, auto),
      width: 100%,
      [#project_title],
      [#counter(page).display("1 / 1", both: true)],
    )
  ],
)

#set text(font: "Libertinus Serif", size: 18pt, fill: ink)
#set par(leading: 0.85em, justify: false)

// A card with a required content argument and optional styling.
#let card(body, fill: soft, radius: 8pt) = rect(
  width: 100%,
  fill: fill,
  radius: radius,
  inset: 12pt,
  body,
)

// Cover page.
#let cover(title, subtitle, author, date) = align(horizon)[
  #align(center)[
    #text(size: 44pt, weight: "bold", fill: accent)[#title]
    #v(10pt)
    #text(size: 20pt, fill: muted)[#subtitle]
    #v(20pt)
    #line(length: 45%, stroke: 2pt + accent)
    #v(18pt)
    #text(size: 15pt)[#author]
    #v(4pt)
    #text(size: 12pt, fill: muted)[#date]
  ]
]

// Reusable content slide. Keep the body short enough to fit one page.
#let slide(title, body) = align(horizon)[
  #block(width: 100%, breakable: false)[
    #text(size: 30pt, weight: "bold", fill: accent)[#title]
    #v(7pt)
    #line(length: 100%, stroke: 1.5pt + accent)
    #v(16pt)
    #body
  ]
]

// Big-number statistic.
#let stat(value, label) = align(center)[
  #text(size: 42pt, weight: "bold", fill: accent)[#value]
  #v(3pt)
  #text(size: 13pt, fill: muted)[#label]
]

// ── Slides ────────────────────────────────────────────────────────
#cover(
  [Project Title],
  [A concise presentation subtitle],
  [Author or organization],
  [Month YYYY],
)

#pagebreak()
#slide(
  [Agenda],
  [
    #grid(
      columns: (auto, 1fr),
      gutter: 14pt,
      [#circle(radius: 14pt, fill: accent)[#align(center + horizon)[#text(size: 11pt, fill: white, weight: "bold")[1]]]],
      [Context and objective],
      [#circle(radius: 14pt, fill: accent)[#align(center + horizon)[#text(size: 11pt, fill: white, weight: "bold")[2]]]],
      [Main findings],
      [#circle(radius: 14pt, fill: accent)[#align(center + horizon)[#text(size: 11pt, fill: white, weight: "bold")[3]]]],
      [Recommendation and next steps],
    )
  ],
)

#pagebreak()
#slide(
  [A clear before-and-after comparison],
  [
    #grid(
      columns: (1fr, 1fr),
      gutter: 22pt,
      [#card([*Before* — The initial situation, described briefly.], fill: soft)],
      [#card([*After* — The improved situation and its measurable effect.], fill: rgb("#fdf1e0"))],
    )
  ],
)

#pagebreak()
#slide(
  [Three numbers worth remembering],
  [
    #grid(
      columns: (1fr, 1fr, 1fr),
      gutter: 18pt,
      [#stat([42%], [faster])],
      [#stat([3×], [more efficient])],
      [#stat([0], [external packages])],
    )
  ],
)

#pagebreak()
#slide(
  [Closing],
  [
    #align(center)[
      #text(size: 24pt, weight: "bold", fill: accent)[Thank you]
      #v(8pt)
      #text(size: 16pt, fill: muted)[Questions?]
    ]
  ],
)
```

### Optional image example (not part of the starter template)

Only add this to a project after placing the actual file at the stated path:

```typ
#figure(
  image("assets/diagram.svg", width: 80%, fit: "contain", alt: "A diagram of the system components and their connections"),
  caption: [System overview],
)
```

The helper `slide()` uses a non-breakable block to discourage a slide from splitting. It does **not** automatically prove that the content fits within the page. Always inspect the exported pages and correct overflow or unexpected extra pages.

## 10. Design rules for polished slides

The agent should design a coherent presentation, not merely put text into a template.

- **One main idea per slide.** Prefer assertion-style titles such as “Costs fell by 42%” over generic labels such as “Results”. Do not state a conclusion stronger than the evidence.
- **Build a clear narrative.** A useful default sequence is context/problem → evidence/analysis → implication → recommendation/next step. Adapt it to the user's purpose rather than forcing every deck into the same structure.
- **Use visual variety with restraint.** Combine concise text, diagrams, charts, tables, images, and whitespace. A deck made entirely of bullet lists usually reads like a document rather than a presentation.
- **Use consistent margins and rhythm.** Start with 18–24pt outer horizontal margins on a standard 16:9 page, grid gutters around 18–28pt, and card insets around 12–18pt. Adjust when content or aspect ratio requires it.
- **Create a limited palette.** Use one primary accent, optionally one secondary accent, neutrals, and background/surface colors. Define color tokens once and reuse them; avoid arbitrary new colors on each slide.
- **Maintain sufficient contrast.** Normal text should meet at least a 4.5:1 contrast ratio against its background; large text should meet at least 3:1. Do not use pale gray body text on a pale background. Check contrast for every color pairing, not only the main theme.
- **Keep slides readable.** Aim for no more than 3–5 short bullets in a typical slide and no more than 6 unless the content genuinely requires it. Keep bullets to one line where practical; avoid deep nesting.
- **Avoid overcrowding.** If a slide does not fit, split it, simplify it, replace prose with a diagram, or move detail into an appendix or companion handout. Do not assume a plain Typst PDF has speaker notes; use a slide package that explicitly supports them if they are required.
- **Use whitespace intentionally.** Center sparse layouts when appropriate, but do not force vertical centering on dense slides. Align repeated elements consistently.
- **Use shapes for structure.** Use number markers for steps, cards for comparisons, tables for specifications, and diagrams for relationships. Do not add decorative shapes that compete with the content.
- **Make the cover purposeful.** Include a clear title and useful subtitle. Add author, organization, or date only when relevant.
- **Keep recurring elements consistent.** Reuse the same title treatment, footer, page numbering, chart colors, and spacing system throughout the deck.
- **Use meaningful visuals.** Every visual should explain, evidence, orient, or establish context. Avoid generic stock imagery that adds no information.
- **Avoid false precision.** Label units, date ranges, assumptions, estimates, and uncertainty.

If unsure about style, choose a restrained editorial or product-design direction: strong typography, high contrast, a limited palette, generous whitespace, and a few purposeful visuals. Do not imitate a brand's protected logo or proprietary assets.

## 11. Sources, research, and factual integrity

When a presentation contains factual or externally sourced claims:

- Use the user's supplied documents and data as the primary basis when the task is based on those materials.
- If research is requested or current facts materially matter, verify claims with trustworthy, preferably primary sources. Record source title, publisher, URL, publication date, and access date where appropriate.
- Put concise source references on the slide that contains the claim, chart, image, or quotation. Include a readable references slide at the end for full citations when the deck has multiple sources.
- Attribute data visualizations to the dataset or organization that published the data. Include units, measurement period, and relevant methodology or caveats.
- Distinguish quoted wording from paraphrase. Use quotation marks for direct quotations and avoid reproducing long copyrighted passages.
- Never invent citations, page numbers, quotes, statistics, image origins, or publication dates. If a source cannot be verified, say so or omit the claim.
- Separate facts from estimates, assumptions, projections, and recommendations. Make important assumptions visible.
- Do not silently resolve contradictions between supplied sources. Flag material inconsistencies and explain which source the slide uses.

A compact slide citation can use a numbered reference such as `[1]` with a matching full entry on the final references slide. For academic or regulated contexts, follow the citation style the user specifies.

## 12. Quality assurance and delivery

The agent is responsible for producing a usable deliverable, not just a `.typ` text file.

### Compile and inspect

1. Create the project folder and write `main.typ`. Include only assets that are actually used.
2. Check the compiler version and available fonts:
   ```bash
   typst --version
   typst fonts
   ```
3. Compile the complete deck:
   ```bash
   typst compile main.typ /tmp/check.pdf
   ```
4. Fix every compilation error. Review every warning, especially font, missing asset, unsupported feature, and package warnings. Do not ignore a warning merely because a PDF was produced. If compilation fails, treat the error output as unfinished work: locate the actual innermost syntax/layout issue, edit the `.typ` source, and compile again. Never leave the user with a status such as “compiled with errors” while presenting the deck as complete.
5. Export **all** slides to PNG:
   ```bash
   mkdir -p preview
   typst compile --format png --ppi 160 main.typ "preview/slide-{0p}.png"
   ```
6. Inspect every slide at presentation size. A contact sheet is useful for checking consistency, but also inspect slides individually for small text, clipped elements, awkward line breaks, image crops, and source legibility.
7. Compare the actual page count to the intended slide count. Investigate extra blank pages, split slides, and any unexpected page breaks.
8. Correct and recompile until the PDF and image previews are clean.

### Visual inspection checklist

- No text, images, or shapes are clipped or outside the page.
- No slide unexpectedly spills onto a second page.
- No unintended blank pages or duplicate slides exist.
- Titles and body text are legible at presentation size.
- Fonts render as intended and required glyphs are present.
- Grid columns, gutters, margins, and alignment are consistent.
- Charts have readable labels, units, scales, and source information.
- Image crops are intentional and alt descriptions are meaningful.
- Footers and page numbers are correct and unobtrusive.
- The opening and closing slides feel intentional and the narrative is coherent.

### Delivery checklist

- Deliver the PDF as the primary presentation output unless the user asks for another format.
- Include `main.typ` so the user can edit the source.
- Include all required local assets, ideally in a project ZIP if there are multiple files.
- If the user wants to continue editing locally, explain the workflow from §2: run `typst watch main.typ` from the project folder; it creates/updates `main.pdf`; stop it with Ctrl+C.
- If Typst is not installed or compilation cannot be run, state that clearly. Provide the source and the exact compile command, but do not claim that the deck was compiled or visually verified.
- If any asset must still be supplied by the user, name the exact missing file and the expected folder path. Prefer avoiding such dependencies when a native diagram can be used instead.

## 13. Packages and advanced presentation features

The base template should remain package-free unless the user needs a specific feature that justifies a dependency.

Typst packages are imported from the Typst Universe, for example:

```typ
#import "@preview/NAME:VERSION": ...
```

Rules for packages:

- Do not guess package names, versions, APIs, or function signatures.
- Before using a package, check its current Typst Universe page and documentation. Pin the exact version in the import.
- Explain that the first use may need internet access to download the package; subsequent use can work from the local cache.
- Keep a package-free alternative when practical.
- Recompile and visually inspect the complete deck after adding or updating a package.

For slide-specific features such as incremental reveals, animations, themes, or speaker notes, evaluate a presentation package such as [Touying](https://typst.app/universe/package/touying/) only after checking its current documentation and version. At the time of this revision, the Typst Universe listing shows Touying 0.8.0 and a minimum Typst version of 0.15.0; that information can change, so re-check it before importing. Touying documents support for themes, incremental reveals, animations, and speaker notes. Do not imply that a normal package-free Typst PDF has animation or interactive controls. Typst's built-in web-app presentation mode is separate from the PDF export workflow.

## 14. Common pitfalls and fixes

| Symptom | Likely cause and fix |
|---|---|
| `unknown variable: vspace` or `vgap` | These are not the spacing calls used in the current standard library. Use `#v(...)` for vertical space or a `block(inset: ...)` for padding. |
| `unknown variable` for a theme color in a footer | Define theme variables before the `#set page(...)` rule that references them. Check content and function scope. |
| `unexpected argument` | Check the function signature and use named arguments for named parameters. Separate content arguments with commas inside function-call parentheses. |
| `unclosed delimiter` at `#slide(` or the body's `[` | The slide call is incomplete somewhere below the reported line. Match the call's opening `(` with `)`, the title's `[` with `]`, and the body-opening `[` with `]`. Also close every nested function/content block. Use the multiline call pattern in §3 and inspect to the end of that slide before changing delimiters. |
| `unclosed delimiter` reported at an outer `#slide(` even though that call looks complete | A nested helper may be closed in the wrong order. For `[#card([ ... ])]`, close the inner content `]`, then the function `)`, then the outer cell `]` — never `)]`. Expand nested calls over multiple lines and inspect each delimiter from the inside out. |
| `unknown escape sequence` involving `\n`, or an unwanted `n` appears | `\n` is for strings, not markup. Use `#linebreak()` or the markup line-break syntax (backslash followed by whitespace); use a blank line for a paragraph. |
| Literal `, name: [...]` appears in the PDF | A markup-mode content call may have ended before the comma. Use a regular parenthesized function call, e.g. `#f([x], y: [z])`. |
| `expected function, found content` | Adjacent content blocks inside a code-mode call may be parsed as indexing. Separate cells/arguments with commas: `#grid(..., [a], [b])`. |
| Content is missing or a slide is empty | Inspect positional vs named parameters and the body argument. Confirm that content is passed in the function's actual body parameter. |
| A loop emits nothing | Add a content block after the loop: `#for item in items [ #item ]`. |
| `unknown font family` or unexpected font substitution | Run `typst fonts`, select an installed family, or add a licensed font using `--font-path`. Inspect the resulting PDF for glyph and line-break changes. |
| Image not found | Check spelling, case, relative path, project root, and whether the asset is included in the project folder. |
| HTML export fails | Enable the experimental feature with `--features html`; HTML export is not a production-stable target. |
| A slide creates an extra page | Reduce or restructure the content, split it into multiple slides, or adjust spacing and layout. Do not rely on `pagebreak()` alone to control overflow. |
| Footer page numbers are wrong or unavailable | Use `context [...]` with `counter(page)` and verify the page numbering after compilation. |
| The page ratio is wrong | Confirm the `paper` preset or explicit `width`/`height` values and check that a later page set rule does not change the geometry. |

Do not copy outdated API advice from old tutorials without checking it against the installed Typst version and the current reference documentation.

## 15. Official references

Use these official sources when a detail is unclear or may have changed:

- Typst documentation overview: https://typst.app/docs/
- Syntax and the three modes: https://typst.app/docs/reference/syntax/
- Scripting and functions: https://typst.app/docs/reference/scripting/
- Page setup: https://typst.app/docs/reference/layout/page/
- Page breaks: https://typst.app/docs/reference/layout/pagebreak/
- Grid layout: https://typst.app/docs/reference/layout/grid/
- Images and image parameters: https://typst.app/docs/reference/visualize/image/
- Text and font selection: https://typst.app/docs/reference/text/text/
- Experimental HTML export: https://typst.app/docs/reference/html/
- Typst CLI and installation: https://typst.app/open-source/
- Typst changelog: https://typst.app/docs/changelog/
- Typst Universe packages: https://typst.app/universe/
