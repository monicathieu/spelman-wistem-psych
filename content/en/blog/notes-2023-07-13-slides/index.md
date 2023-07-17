---
title: "Project notes: Making graph presentations"
output: blogdown::html_page
description: "Lecture notes for making graph presentations in R."
excerpt: "Lecture notes for making graph presentations in R."
date: 2023-07-13
lastmod: "2023-07-17"
draft: false
images: []
categories: ["Class notes"]
tags: []
contributors: ["Monica Thieu"]
pinned: false
homepage: false
---

## Making presentation-quality graphs and slides

The slides below are designed to *demonstrate* how to make presentation-quality graphs and slides in R, while also *explaining* technique (how?) and rationale (why?) behind graph and slide design.

The slides are *supposed* to appear in the web page below, but I’m currently having some issues getting the slides preview not to get blocked by browsers!

The slides are hosted [here](https://monicathieu.quarto.pub/guide-to-presenting-graphs/) if they’re not showing up below.

If the slides do show up below, but if you want to see the slides in full-screen, you can right-click and choose “Open frame in new tab”, or whatever option seems like it will do that.

<iframe width="720" height="480" src="https://monicathieu.quarto.pub/guide-to-presenting-graphs/" title="Monica's guide to presenting graphs"></iframe>
<iframe width="720" height="480" src="https://monicathieu.quarto.pub/guide-to-presenting-graphs/" title="Monica&#39;s slides on presenting graphs">
</iframe>

There are a few different languages in play here, which is why sometimes the syntax for setting different presentation aesthetics seems inconsistent:

- **R:** The language in your code chunks, that’s actually doing the data reading/cleaning/analysis/graphing. Anytime you’re setting `ggplot2` aesthetics on your *plots,* that’s R code.
- **Markdown:** The language in the non-code “regular text” chunks. This sets the titles, text, and other *slide content.* Most of the time this doesn’t look like a language at all, but instead like regular text. The “language” features include how setting a title with two pound-hashtag signs like `## Slide title` makes that line of text render as a slide title instead of regular text.
- **Quarto and reveal.js (JavaScript):** The language that actually handles transforming the whole code document into HTML slides. This sets *slide aesthetics and formatting.* When you set settings like `#| fig-asp: 0.6` in your plot code chunks, or `{background-image="image.png"}` to set background images, that syntax is a combo of Quarto and reveal.js code.
