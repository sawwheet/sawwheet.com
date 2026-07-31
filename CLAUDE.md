# CLAUDE.md

Hugo blog published to <https://sawwheet.com> via GitHub Pages.

## Commands

```bash
hugo server -D          # local preview with drafts, http://localhost:1313
hugo --gc --minify      # what CI builds; output to public/ (gitignored)
```

There are no tests, no linter, and no package manager. A clean `hugo` run with
no `WARN` lines is the bar.

## Posts are generated, and also hand-edited

`content/posts/*.md` is [ox-hugo](https://ox-hugo.scripter.co/) output exported
from Org sources kept outside this repo.

Most posts have since been edited here and no longer match those sources, so
**re-exporting is not a safe fix** — for older posts it reflows every paragraph,
demotes headings, and collapses poems into single paragraphs, because their hard
line breaks exist only in the Markdown committed here.

- Treat the committed Markdown as authoritative. Edit it in place.
- Never re-export a post to "fix" it without being asked.
- Don't bulk-rewrite `content/posts/*.md`. The prose is the author's; leave the
  writing alone unless asked. Fix front matter or build issues freely.
- A couple of posts have no source outside this repo at all.

Post filenames are clean, title-derived slugs. Renamed posts carry an `aliases`
entry pointing at their previous URL.

## Theme is a Hugo Module, not a submodule

`hugo-blog-awesome` v1.20.0 comes from `go.mod` + `[module]` in `hugo.toml`.
There is deliberately **no `themes/` directory** and no `theme =` setting — the
old, dead submodules were removed. Don't "restore" them.

Building requires `go` on PATH so Hugo can resolve the module. Theme sources
live in Hugo's module cache (`hugo config | grep cachedir`), under
`modules/filecache/modules/pkg/mod/github.com/hugo-sid/`.

## `layouts/` shadows the theme

A file in `layouts/` overrides the theme file at the same path. Before editing
one, read the theme's version to see what you're diverging from.

`layouts/partials/footer.html` **intentionally renders nothing** — that is how
the theme's copyright/social footer is suppressed. Deleting it *restores* the
theme footer instead of removing it. It is not dead code.

## `public/` is gitignored

CI rebuilds and deploys it on every push to `main`. Never commit it, and never
edit files in it to change the site — edit `content/`, `layouts/`, or
`static/` instead.

## Config notes

- `markup.goldmark.renderer.unsafe = true` is **required**. ox-hugo emits
  literal `<span class="figure-number">` in figure captions; without it Goldmark
  strips them and the build warns "Raw HTML omitted while rendering".
- `params.mainSections = ['posts']` is set explicitly. `layouts/index.html`
  filters the homepage feed on it.
- `params.description`, `params.sitename`, and `params.author.*` are commented
  out placeholders awaiting the author's own words. Don't invent bio or
  description copy — ask.

## Changing published URLs

Post URLs are live. If a change would move one, add an `aliases` entry for the
old path in the front matter, and confirm first.
