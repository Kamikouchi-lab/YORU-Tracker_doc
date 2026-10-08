# YORU Tracker's documents

The documentation site of [YORU Tracker](https://github.com/Kamikouchi-lab/YORU-Tracker),
published at <https://kamikouchi-lab.github.io/YORU-Tracker_doc/>.

It is a [Jekyll](https://jekyllrb.com/) site in the style of the
[YORU documentation](https://kamikouchi-lab.github.io/YORU_doc/) (Lanyon theme),
with a navy theme (`theme-navy` in `public/css/lanyon.css`).

| Folder / file | Content |
|---|---|
| `index.md` | Home |
| `_guides/` | User Guides (sidebar order: `order:` in the front matter) |
| `_tutorial/` | Step-by-step Protocols |
| `_devnotes/` | Development notes |
| `troubleshootings.md` | Q and A |
| `logos/`, `public/` | Logos, favicon, CSS and JavaScript |

## Preview locally

```
gem install jekyll jekyll-sitemap webrick
jekyll serve
```

and open <http://127.0.0.1:4000/YORU-Tracker_doc/>.

## Publish

`.github/workflows/jekyll-gh-pages.yml` builds and deploys the site on every
push to `main`. In the repository's **Settings → Pages**, set **Source** to
**GitHub Actions**.
