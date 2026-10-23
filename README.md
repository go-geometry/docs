# docs

[![docs](https://github.com/go-geometry/docs/actions/workflows/docs.yml/badge.svg)](https://github.com/go-geometry/docs/actions/workflows/docs.yml)
[![License](https://img.shields.io/badge/license-BSD--3--Clause-0B6B87?style=flat-square)](LICENSE)

**The go-geometry documentation site**, published at
<https://go-geometry.github.io/docs/>.

A [Hugo](https://gohugo.io/) site with the [tannevaled/hextra](https://github.com/tannevaled/hextra)
fork of the [Hextra](https://github.com/imfing/hextra) theme (pinned to v0.16.0),
built the same way as the [go-bignum documentation](https://github.com/go-bignum/docs),
with go-geometry's branding. The organization is **planned (wave 2)** and has no
library yet: the site gives its scope, its plan (a link to
[go-scikit/roadmap](https://github.com/go-scikit/roadmap)) and one page per
planned package.

## Versions

`params.libraries` in `hugo.yaml` names the release of each package the pages
describe, or `main` while it has no tag. A version of the documentation is
**the newest minor version among the tagged ones**, or `main` while none is
tagged:

| Source | Published under |
| --- | --- |
| branch `main`, with `params.libraries` in `hugo.yaml` | `https://go-geometry.github.io/docs/<MAJOR.MINOR>/`, or `/docs/main/` |
| the newest version (else `main`) | also `https://go-geometry.github.io/docs/latest/` |

`https://go-geometry.github.io/docs/` redirects to `latest/`. Each push to `main`
replaces the directory of the version it describes, and nothing else. No package
exists yet, so the pages are published under `main/`.

## Layout

| Path | What |
| --- | --- |
| `content/en/` | the pages; `_index.md` is the home, one directory per planned package, `weight` orders the sidebar |
| `i18n/en.yaml` | the site's strings, beside the theme's |
| `hugo.yaml` | `baseURL`, the packages described, the navbar (version › theme › search › GitHub), the brand mounts |
| `layouts/_partials/custom/version-select.html` | the version selector (reads `versions.json`) |
| `layouts/_partials/custom/content-begin.html` | the outdated-version banner (reads `versions.json`) |
| `layouts/_partials/favicons.html` | the brand's favicons |
| `assets/css/custom.css` | the brand colour as Hextra's primary colour, the table headers |
| `themes/hextra` | submodule: [tannevaled/hextra](https://github.com/tannevaled/hextra), pinned to v0.16.0 |
| `branding` | submodule: [go-geometry/brand](https://github.com/go-geometry/brand) (the mark, its PNGs and ICO) |
| `scripts/publish-version.sh` | puts one build into a `gh-pages` checkout and rewrites `versions.json`, `latest` and the root redirect |

## Working locally

```sh
git clone --recurse-submodules https://github.com/go-geometry/docs.git
cd docs
mkdir -p data && sh themes/hextra/scripts/page-history.sh > data/pagehistory.json
hugo server --baseURL http://localhost:1313/docs/main/
```

then open <http://localhost:1313/docs/main/>. Hugo 0.146 or later; no Python,
no Node.

## Publication

`.github/workflows/docs.yml`:

| Job | When | What |
| --- | --- | --- |
| `build` | pull requests, `main` | builds with `--baseURL …/docs/<version>/`, checks internal links and anchors (lychee, offline), uploads the site |
| `links` | pull requests, `main` | fetches every external URL the pages name |
| `deploy` | `main` | `scripts/publish-version.sh` into `gh-pages` (started empty on the first deploy), then a commit and a push |

GitHub Pages serves the `gh-pages` branch.

## Licence

BSD-3-Clause.
