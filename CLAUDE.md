# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a personal technical blog at [910.jp](https://910.jp) built with [Jekyll](https://jekyllrb.com/) using the [Chirpy theme](https://github.com/cotes2020/jekyll-theme-chirpy). It also serves as the official website for VS Code publisher **long-kudo**.

## Commands

**Local development server:**
```sh
bundle exec jekyll s
```
Access at <http://127.0.0.1:4000>

**Production build:**
```sh
bundle exec jekyll b
```

**Run HTML proofer (link/HTML validation, same as CI):**
```sh
bundle exec htmlproofer _site \
  --disable-external=true \
  --ignore-urls "/^http:\/\/127.0.0.1/,/^http:\/\/0.0.0.0/,/^http:\/\/localhost/"
```

**Install dependencies:**
```sh
bundle install
```

## Architecture

**Content structure:**
- `_posts/` — Blog posts, named `YYYY-MM-DD-slug.md`. Posts are written in Markdown with YAML front matter.
- `_tabs/` — Static pages shown in the sidebar (About, Archives, Categories, Tags, Extensions, Privacy Policy). Files in `_tabs/ja/` are Japanese translations of the same pages.
- `_layouts/home.html` — Overrides the Chirpy home layout to add a small extensions link above the post list.
- `_includes/footer.html` — Overrides the Chirpy footer to add "Official VS Code Extension Publisher: long-kudo".
- `_data/contact.yml` — Sidebar social/contact links. `_data/share.yml` — Post share buttons.
- `_plugins/posts-lastmod-hook.rb` — Jekyll hook that auto-sets `last_modified_at` from git history for posts with more than one commit.

**Multilingual support:**
The extensions page (`_tabs/extensions.md`) contains both English and Japanese content in a single file, toggled client-side via JavaScript. The selected language is persisted in `localStorage` and auto-detected from `navigator.language` on first visit.

**Deployment:**
Pushes to `main` trigger `.github/workflows/pages-deploy.yml`, which builds with `JEKYLL_ENV=production` and deploys to GitHub Pages automatically.

**Theme:** `jekyll-theme-chirpy ~> 6.5`. Static assets (JS/CSS) come from the `assets/lib` git submodule (`cotes2020/chirpy-static-assets`).

## Post Front Matter

Typical post front matter:
```yaml
---
layout: post        # optional, defaults to 'post' for _posts
title: "Post Title"
date: 2026-01-01 12:00:00 +0900
categories: [Category]
tags: [tag1, tag2]
img_path: /assets/img/screenshots  # optional image base path
image:
  path: image.png
  alt: description
---
```

Posts cross-posted from Zenn include a note linking back to the original Zenn article.
