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
- `content/research/` - publications at `/research/<directory>/`. `venue` in the front
  matter is what shows on the front page. The section's own list page is switched off via
  `build: render: never`; `/research/` and `/about/` redirect to `/` via aliases on
  `content/_index.md`.
- `content/_index.md` - the bio on the front page.
