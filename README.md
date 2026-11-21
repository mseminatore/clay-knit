# clay-knit

Reports from Markdown with [Clay](https://github.com/mseminatore/clay) code chunks, written as
PDF or as a web page, the way R Markdown's knitr does it for R. The chunks run in order as one
program; the report shows each chunk's code, what it printed and the figure it drew with
[clay-plot](https://github.com/mseminatore/clay-plot), between the report's text. In the web
page the figures are interactive: hover for values, click a legend entry to hide its series.

````markdown
---
title: Sales
author: Me
---

# Results

Totals by **quarter**:

```clay
sales = [12, 30, 7, 18]
("total: " + (sales sum) toString) println
Plot bar(["Q1", "Q2", "Q3", "Q4"], sales)
Plot title("Sales by quarter")
```

Q2 was the best quarter.
````

```clay norun
import("knit")
Knit render("sales.md")                     // writes sales.pdf
Knit render("sales.md", #{format: "html"})  // writes sales.html
```

`examples/report.md` shows everything; `clay examples/knit.clay` knits it into
`examples/report.pdf`, and `clay examples/knit.clay --html` into `examples/report.html`.

## Chunks

A chunk is a fenced block whose language is `clay`. Options go after it, in either form:

````markdown
```{clay echo=false}
```clay output=false, eval=false
````

| Option | |
|---|---|
| `echo=false` | Run the chunk, but don't show its code |
| `output=false` | Run it, but don't show what it printed or its figure |
| `eval=false` (or `norun`) | Show the code without running it |
| `results=asis` | Lay out what it prints as Markdown: text, lists, tables... |
| `results=hide` | Don't show what it prints (its figure still shows) |
| `caption="..."` (or `fig.cap`) | The figure's caption |
| `fig.width=5`, `fig.height=3` | The size (inches) the chunk's figures are made at; one alone keeps the default's shape |
| `fig.asp=0.5` | The height as a fraction of the width |
| `out.width="50%"` | How wide the figure is shown: `50%` of the text, `3in`, `8cm`, `60mm`, `200pt` or `300px`; it is scaled, text and all |
| `fig.align=left` | `center` (the default), `left` or `right` |
| `interactive=false` | HTML: a static figure, without tooltips and legend toggles |

The size options can also be spelled as Quarto does (`fig-width`), and set for the whole report
in the front matter:

```markdown
---
title: Sales
fig-width: 5
fig-height: 3
---
```

A figure made with its own size (`Plot figure(#{figsize: [8, 3]})`) keeps it: the options set
the size figures get when the code doesn't say. A figure is never shown wider than the text.

Values can be quoted, as knitr's are (`results='asis'`), and must be if they have spaces or
commas: `{clay caption="Sales, by quarter", echo=false}`.

Chunks share their variables, as lines of one script do. What a chunk prints (`println`,
`Console print`) appears below its code. If it leaves a figure with something plotted in it
(`Plot plot(...)`, `Plot figure`, `Plot subplots`...), the figure is drawn after the output,
as vector graphics at its own size (narrowed to the text width), and the next chunk starts
with no figure. A caption (`caption="..."`, Markdown) goes under it as "Figure 1: ...";
captioned figures are numbered in order. `Plot` is there without an import. Don't call
`Plot show` in a report: it opens a viewer.

Chunks run in the folder you run knit from, so they find the same bricks; `Knit dir` is the
report's folder, for files beside it: `File read(Knit dir + "/data.csv")`.

An error stops the report. It is raised by `Knit render` with the place in the report:

```
knit: sales.md failed:
sales.md:14: RuntimeError: no data
```

## Inline code

`` `{clay} expr` `` in the text is replaced by the expression's value, so numbers in sentences
follow the data:

````markdown
```clay
sales = [12, 30, 7, 18]
```

Total sales were `{clay} sales sum` units, a mean of `{clay} Knit fixed(sales sum / 4.0, 2)`.
````

gives "Total sales were 67 units, a mean of 16.75." An expression runs where it is in the
report, after the chunks above it, and its value (`toString`, `nil` as "nil") goes in as plain
text in the style around it: `` **`{clay} x`** `` is bold. `Knit fixed(x, digits)` writes a
number with that many decimal places. It works in paragraphs, headings, lists, table cells and
the front matter's title, not in fenced code; an error in one is reported at its line. To show
the syntax itself, use two backticks (``` ``{clay} x`` ```) or escape the first one.

## Markdown

Front matter (`title`, `author`, `date`, between `---` lines), headings, paragraphs,
**bold**, *italic*, `code` (and ``` ``code with ` in it`` ```), links (shown in blue) and
[math](#math) (`$x^2$`), bulleted and numbered lists (nested by
indenting), quotes, horizontal rules, other fenced code, tables, and images on a line of their
own (`![alt](plot.png)`, PNG or JPEG, relative to the report; the alt text, if any, is its
caption, numbered with the figures; `{width=50% align=left}` after it sizes and places it, as
`out.width` and `fig.align` do). Pages are US Letter with 1-inch
margins and page numbers.

Tables are GitHub's: a header row, a line of `---` cells (`:--`, `:-:` and `--:` align a
column left, center or right), and rows; the `|`s at the ends are optional, and `\|` is a `|`
in a cell. Cells can be styled. A table is centered, each column as wide as its text; one too
wide for the page has its columns narrowed toward their longest words and wraps their text. A
long table continues on the next page under its header again.

```markdown
| Quarter | Sales | Note        |
|:--------|------:|-------------|
| Q1      |    12 | first       |
| Q2      |    30 | **best**    |

Table: Sales by quarter
```

A paragraph `Table: text` (or `: text`) just after a table, or just before it, is its caption,
shown above the table as "Table 1: text"; captioned tables are numbered in order.

### Math

Text between `$` signs is math, and a paragraph that is only `$$...$$` is display math, centered
on a line of its own, as in R Markdown and Quarto:

```markdown
The curve is $y = \sin(x)\, e^{-x/6}$ and peaks at $x \approx 1.4$. The error is

$$\mathrm{MSE} = \frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2$$
```

It is TeX's notation as [clay-plot](https://github.com/mseminatore/clay-plot) lays it out in
figures: `^` and `_` for super- and subscripts (they nest), Greek letters, operators and
relations (`\leq \neq \approx \pm \times`), big operators (`\sum \prod \int \sqrt{x}`),
arrows, sets (`\in \subset \cup \mathbb{R}`), logic (`\forall \exists \neg`), `\sin`, `\log`
and the like upright, and `\frac{a}{b}` as a/b. Math is text, so it can be selected and
searched: in the PDF in Helvetica and the Symbol font, in HTML as raised and smaller text. It
doesn't break across lines.

A `$` opens math only before a non-space, and closes it only after a non-space and not before a
digit, as Pandoc reads it, so `$5 and $10` is text; `\$` is a dollar sign. Math in `code` is
code. There are no stacked fractions, roots over their argument, matrices or accents (`\hat{y}`
shows y): for those, draw the formula in a figure, or write it out.

### Tables from data

`Knit table(rows, opts)` makes the Markdown for a table, like knitr's `kable`; print it in a
`results=asis` chunk:

````markdown
```{clay results='asis'}
sales = [#{quarter: "Q1", sales: 12, growth: nil},
         #{quarter: "Q2", sales: 30, growth: 1.5}]
Knit table(sales, #{header: ["Quarter", "Sales", "Growth"], digits: 2,
    caption: "Sales by quarter"}) println
```
````

Rows are Arrays, or Dicts (the first one's keys are the columns, in order). A cell is its
value's `toString`; `nil` is empty, and `|`s and line breaks are escaped.

| Option | |
|---|---|
| `header` | The column names (default: the Dicts' keys; none for Arrays) |
| `align` | A column's `"l"`, `"c"` or `"r"` (or `left`...); columns of numbers go right, others left |
| `digits` | Decimal places, for columns with numbers that are not whole |
| `caption` | The table's caption |

Text uses the standard PDF fonts (Helvetica, and Courier for code), so it covers
Windows-1252: Western European letters, `°`, `€`, curly quotes and dashes. Other characters
show as `?`.

## Cross-references

Label a figure or table, and write `@label` in the text for its number ("Figure 2",
"Table 1"), before or after it:

````markdown
@fig-sales shows the totals in @tbl-sales.

```{clay #fig-sales caption="Sales by quarter."}
Plot bar(["Q1", "Q2", "Q3", "Q4"], [12, 30, 7, 18])
```

| Quarter | Sales |
|---------|------:|
| Q1      |    12 |

Table: Sales. {#tbl-sales}
````

| Label | |
|---|---|
| A chunk's figure | `{clay #fig-x}` or `label=fig-x`, among its options |
| An image | `![alt](plot.png){#fig-x}` |
| A table | `{#tbl-x}` at the end of its caption: `Table: Sales. {#tbl-x}` |
| A table from data | `Knit table(rows, #{label: "tbl-x"})` |

Labels are letters, digits, `-` and `_`, as Quarto's are; start them with `fig-` and `tbl-`.
A labelled figure or table without a caption is numbered too, its caption just "Figure 3".
Tables and figures are numbered separately, in order. An `@fig-` or `@tbl-` reference with
no such label shows as a red **?@fig-x**, and `Knit render` warns about it (and about a label
used twice) on stderr. Other `@`s, as in `me@example.com`, are left alone.

## HTML

`Knit render(path, #{format: "html"})`, an output ending in `.html`, or `format: html` in the
front matter (R Markdown's `output: html_document` works too) writes the report as one web page
with nothing else to load: its styles are in the page, images are embedded, and each figure is
clay-plot's interactive SVG. Hover over a point, bar, pie wedge or heatmap cell for its values;
click a legend entry to hide or show its series. `interactive=false` on a chunk (or in the front
matter) makes its figures static.

```markdown
---
title: Sales
format: html
---
```

Everything else is as in the PDF: chunk options, figure sizes (`out.width` becomes the figure's
CSS width, so it scales with the window), captions and numbering, cross-references, tables and
inline code. Headings get ids from their text (`## Data and results` is `#data-and-results`),
for links into the page. A figure is drawn at its own size and scaled down to fit a narrow
window; code that is too wide scrolls rather than wraps.

## Not yet

- Stacked fractions, roots over their argument, matrices and accents in math (TeX through LaTeX)
- Keeping going after an error (knitr's `error=TRUE`)

## How it works

`Knit render` writes a Clay program beside the report (`report.md.clay`, deleted afterwards
unless `#{keep: true}`) whose line *n* is the report's line *n*: each chunk's code where it
is, the text as blank lines (but for inline code, which runs on its line), and on the fence
lines calls into `Knit`. It runs that with the
`clay` command, so line numbers in errors are the report's. As the program runs it collects
the report: the text up to a chunk, the chunk's code, then what `Console capture` collected
while it ran, and its figure. When the last chunk has run it numbers the tables and figures,
then lays everything out (figures with clay-plot's `drawPdf`, on
[clay-pdf](https://github.com/mseminatore/clay-pdf) pages, or for HTML as one page with
clay-plot's interactive SVG), so a reference can come before what it refers to. Warnings, and what chunks write to stderr, are passed on to stderr.

| `Knit render(path, opts)` | |
|---|---|
| `format` | `"pdf"` or `"html"` (default: the output's extension, else the front matter's `format`, else PDF) |
| `output` | The file's path (default: the report's, ending in `.pdf` or `.html`) |
| `keep` | Keep the generated program |
| `clay` | The clay command (default: the one running, else `clay` on the PATH) |

## Install

```sh
clay bricks add github.com/mseminatore/clay-knit knit
```

It needs Clay 0.3.7 or later (for `Console capture`), and brings clay-plot, clay-pdf and
clay-image (a native brick, compiled the first time it is used, which needs a C compiler).

## Develop

```sh
clay bricks install                  # fetches plot, pdf and image into bricks/
clay tests/test_knit.clay            # a brick can import itself by name
clay examples/knit.clay              # knits examples/report.md (--html: as a web page)
```

The code is in `src/`: `main.clay` (`Knit`: generating and running the program, and
`KnitDoc`: collecting the report and laying out the PDF), `html.clay` (`KnitHtml`: the web
page) and `markdown.clay` (`KnitMarkdown`: blocks and inline styles).
`.github/workflows/test.yml` runs the tests on Linux, macOS and Windows.

## License

MIT
