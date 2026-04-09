# AGENTS.md

## Overview

Pelican static blog deployed to GitHub Pages at https://blog.rnstlr.ch.
Source branch: `main`, built output pushed to `gh-pages`.

## Setup

```sh
asdf install                  # Python 3.11 via .tool-versions
uv sync                       # Install deps into .venv/
source .venv/bin/activate
```

## Commands

- `make html` — build site (dev settings)
- `make devserver` — serve + auto-regenerate at localhost:8000
- `make publish` — build with production settings (publishconf.py)
- `make github` — publish + deploy to gh-pages branch
- `make clean` — remove output/

Theme is passed via `PELICANOPTS=-t theme` in the Makefile.

## Content

Articles live in `content/articles/YYYY/` with filename `YYYY-MM-DD_slug.{rst,md}`.

**Markdown** uses Pelican metadata (NOT YAML frontmatter):

```
Title: My Post Title
Tags: tag1, tag2
Language: en
Summary: Brief description
Status: draft
```

**RST** uses underlined title + field-list metadata:

```rst
My Post Title
=============

:tags: tag1, tag2
:language: en
:summary: Brief description
:status: draft
```

Newer posts (2022+) prefer Markdown. Older posts are RST. Both are supported.

Images go in `content/images/` and are referenced as `{static}/images/...`.

## Theme

Vendored in `theme/` (customized fork of Pelican's "notmyidea"). Edit directly — not a submodule or package.

## Deploy

Manual only: `make github` builds with production settings and pushes to `gh-pages`. The CI workflow (`.github/workflows/main.yml`) exists but is non-functional.

## Gotchas

- `content/extra/CNAME` is copied to output root for the custom domain — do not delete it.
