# clay-knit

Reports from Markdown with [Clay](https://github.com/mseminatore/clay) code chunks, written as
PDF, the way R Markdown's knitr does it for R. The chunks run in order as one program; the PDF
shows each chunk's code, what it printed and the figure it drew with
[clay-plot](https://github.com/mseminatore/clay-plot), between the report's text.

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
```

`examples/report.md` shows everything; `clay examples/knit.clay` knits it into
`examples/report.pdf`.

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

## Markdown

Front matter (`title`, `author`, `date`, between `---` lines), headings, paragraphs,
**bold**, *italic*, `code` and links (shown in blue), bulleted and numbered lists (nested by
indenting), quotes, horizontal rules, other fenced code, tables, and images on a line of their
own (`![alt](plot.png)`, PNG or JPEG, relative to the report; the alt text, if any, is its
caption, numbered with the figures). Pages are US Letter with 1-inch
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

## Not yet

- Cross-references to tables and figures
- Math (`$x^2$`): planned through LaTeX, as SVG
- HTML output, chunk options for figure size
- Keeping going after an error (knitr's `error=TRUE`)

## How it works

`Knit render` writes a Clay program beside the report (`report.md.clay`, deleted afterwards
unless `#{keep: true}`) whose line *n* is the report's line *n*: each chunk's code where it
is, the text as blank lines, and on the fence lines calls into `Knit`. It runs that with the
`clay` command, so line numbers in errors are the report's. The program lays out the PDF as
it goes: the text up to a chunk, the chunk's code, then what `Console capture` collected
while it ran, and its figure (clay-plot's `drawPdf`, on a
[clay-pdf](https://github.com/mseminatore/clay-pdf) page).

| `Knit render(path, opts)` | |
|---|---|
| `output` | The PDF's path (default: the report's, ending in `.pdf`) |
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
clay examples/knit.clay              # knits examples/report.md
```

The code is in `src/`: `main.clay` (`Knit`: generating and running the program, and
`KnitDoc`: laying out the PDF) and `markdown.clay` (`KnitMarkdown`: blocks and inline styles).
`.github/workflows/test.yml` runs the tests on Linux, macOS and Windows.

## License

MIT
