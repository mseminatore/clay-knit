---
title: "A first **knit** report"
author: Clay
date: `{clay} Date now format("%m-%d-%Y")`
---

# Introduction

This report is *Markdown* with **Clay chunks**. Each chunk runs, and the report shows
its code, what it printed and its figure. Inline `code`, [links](https://example.com)
and ***bold italics*** work too.

- chunks share variables, like one script
- output is captured with `Console capture`
  - nested items are indented
- figures come from the `plot` brick

1. first
2. second

## Data

```clay
xs = (0..40) map { |i| i / 4.0 }
ys = xs map { |x| Math sin(x) * Math exp(0 - x / 6) }
("points: " + xs size toString) println
("peak: " + (ys reduce(0) { |m, y| Math max(m, y) }) toString) println
```

The data has `{clay} xs size` points and peaks at `{clay} Knit fixed(ys reduce(0) { |m, y| Math max(m, y) }, 3)`.
The next chunk plots it, as @fig-sine.

```{clay #fig-sine caption="A damped sine, from the data above."}
Plot figure(#{figsize: [6, 3]})
Plot plot(xs, ys, #{label: "damped sine"})
Plot title("A damped sine")
Plot xlabel("x"); Plot legend; Plot grid(true)
```

> A quote: chunks run in order, top to bottom.

### Hidden code

```{clay echo=false}
"this chunk's code is hidden; only its output shows" println
```

```clay eval=false
// shown, not run
System exit(1)
```

---

### A table

| Chunk option | Default | Effect |
|:-------------|:-------:|--------|
| `echo` | true | show the chunk's **code** |
| `output` | true | show what it printed, and its figure |
| `eval` | true | run it; `norun` is short for `eval=false` |

Table: Chunk options. {#tbl-options}

Columns are left, centered or right aligned by the colons in the line under the header:

| x | x² | x³ |
|--:|---:|---:|
| 1 | 1 | 1 |
| 2 | 4 | 8 |
| 10 | 100 | 1000 |

@tbl-options lists the chunk options, and @fig-sine is the plot from earlier.
A chunk with `results=asis` prints Markdown, and `Knit table` makes a table from data:

```{clay results='asis'}
rows = (1..5) map { |i| #{n: i, root: Math sqrt(i), even: (i % 2 == 0) ifTrue { "yes" } ifFalse { "no" }} }
Knit table(rows, #{digits: 3, caption: "Square roots, from `Knit table`."}) println
```

### Math

Math goes between dollar signs, in TeX's notation: the curve above is
$y = \sin(x)\, e^{-x/6}$, it peaks at $x \approx 1.4$, and the mean squared error of a fit is

$$\mathrm{MSE} = \frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2$$

Sets and logic work too: $\forall \epsilon > 0\ \exists \delta$, $x \in \mathbb{R}$,
$A \cup B \subseteq C$. Prices like $5 and $10 stay text, as does \$x\$.

That's all.
