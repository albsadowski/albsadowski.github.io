# albert's notes

Source for [asadowski.com](https://asadowski.com). Built with [Hugo](https://gohugo.io/);
the theme lives in `layouts/` and `assets/css/` - there is no external theme.

## Local setup

Needs **Hugo 0.158 or newer** (the `build` front-matter key needs ≥ 0.145, `locale`
needs ≥ 0.158). CI is pinned to 0.166.0 in `.github/workflows/publish.yml`; keep the two
roughly in step. Distro packages are often years behind - grab a release binary from
[gohugoio/hugo](https://github.com/gohugoio/hugo/releases) if `hugo version` is older.

```sh
hugo server
```

## Layout

- `content/posts/` - notes. URLs come from the **title** (`permalinks.posts = "/:title/"`),
  not the directory, so retitling a note moves its URL; add an `aliases` entry if you do.
- `content/research/` - one file per publication, **metadata only**. Nothing under
  `/research/` is rendered as a page: `content/research/_index.md` carries a `cascade`
  that sets `build: render: never` on the section and every paper in it, so the front-page
  list (see below) is the whole of it. The Markdown bodies are kept as notes and go
  nowhere. `/research/` and `/about/` redirect to `/` via aliases on `content/_index.md`.
- `content/_index.md` - the bio on the front page.

## Publication front matter

`layouts/partials/publication-meta.html` renders the front-page entry for a publication
from these keys; only `title`, `date` and `venue` are required. The year comes from
`date`. This front matter is the only thing that reaches the site.

```yaml
---
title: "Solving versus Verifying: Catching Contradictions in Tax Reasoning Systems"
date: 2026-09-09
venue: "ICTAI 2026"             # short tag
venue_full: "38th IEEE International Conference on Tools with Artificial Intelligence"
venue_url: "https://ictai.computer.org/2026/"
colocated: "FLoC 2026"          # optional, for co-located workshops
colocated_full: "Federated Logic Conference"
colocated_url: "https://www.floc26.org/"
status: "to appear"             # optional; drop it once published
citation: "Procedia Computer Science 270 (2025) 2166-2175"   # optional
links:                          # optional; array order is render order
  - name: arXiv
    url: "https://arxiv.org/abs/2609.05928"
  - name: Code
    url: "https://github.com/albsadowski/sara-abstention"
draft: false
---
```

A venue renders as the full name followed by the acronym in parentheses, with only the
acronym linked - `38th IEEE International Conference on Tools with Artificial
Intelligence (ICTAI 2026)`. So `venue_full` carries no acronym and `venue` carries the
year. Without `venue_full` the short tag stands alone and the year from `date` is
appended instead (`Preprint, 2026`).

Keep `links` names to the house vocabulary, in this order: **Proceedings, DOI, arXiv,
Code, Dataset, Slides**.
