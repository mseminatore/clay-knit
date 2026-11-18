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

```clay
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

| a | b |
|---|---|
| 1 | 2 |

That's all.
