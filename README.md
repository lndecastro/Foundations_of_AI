# Foundations of AI — CAI 4002

Companion Jupyter Book for **CAI 4002 Foundations of AI** (Section 87714), Fall 2026
U.A. Whitaker College of Engineering · Computing and Software Engineering · Florida Gulf Coast University

**Instructor:** Leandro Nunes de Castro, Ph.D.

[Link to Book](https://lndecastro.github.io/Foundations_of_AI/intro.html)

## About

Thirteen units following the course schedule, from the history of AI through policy and
governance. Every unit pairs conceptual material with **runnable Python snippets** students
can copy into Google Colab — no installation and no prior programming background required.

All code runs on the stock Colab environment (`numpy`, `pandas`, `matplotlib`,
`scikit-learn`, `scipy`). Only Unit 7 requires an install, and the snippet includes it.

## Structure

```
book/
  _config.yml     Book settings (Colab launch buttons enabled)
  _toc.yml        Table of contents
  intro.md        Landing page
  Unit_01 .. Unit_13    Course units
  Appendix_A      Python and Colab primer
  Appendix_B      Capstone project guide
  Appendix_C      AI use and disclosure policy
  references.bib
Data/             Images and datasets
```

## Building Locally

```bash
pip install -r requirements.txt
jupyter-book build book
open book/_build/html/index.html
```

## Deployment

Pushing to `main` triggers `.github/workflows/deploy.yml`, which builds the book and
publishes it to GitHub Pages via `ghp-import`.

When renaming the repository, update the `url` in `book/_config.yml` and the paths in
`.github/workflows/deploy.yml`.

## Related

Builds on the [AI Literacy Program](https://lndecastro.github.io/AI_Literacy/intro.html)
of the FGCU AI Academy, Dendritic Institute for Human-Centered AI & Data Science.
Units 7 and 11 reference AI Literacy modules directly as primary reading.
