---
title: "A first **knit** report"
author: Clay
date: 2026-11-17
---

# Introduction

This report is *Markdown* with **Clay chunks**. Each chunk runs, and the PDF shows
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

The next chunk plots the data from the one above.

```{clay caption="A damped sine, from the data above."}
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

Table: Chunk options.

Columns are left, centered or right aligned by the colons in the line under the header:

| x | x² | x³ |
|--:|---:|---:|
| 1 | 1 | 1 |
| 2 | 4 | 8 |
| 10 | 100 | 1000 |

A chunk with `results=asis` prints Markdown, and `Knit table` makes a table from data:

```{clay results='asis'}
rows = (1..5) map { |i| #{n: i, root: Math sqrt(i), even: (i % 2 == 0) ifTrue { "yes" } ifFalse { "no" }} }
Knit table(rows, #{digits: 3, caption: "Square roots, from `Knit table`."}) println
```

That's all.
