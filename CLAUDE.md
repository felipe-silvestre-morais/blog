# Felipe Silvestre's Blog

A personal technical blog at https://felipe-silvestre-morais.github.io/blog, built with Jekyll using the `jekyll-theme-chirpy` theme.

## Project Structure

- `_posts/` — all blog posts (the main working area)
- `_tabs/` — static pages (about, archives, categories, tags, cv)
- `assets/` — images and static files
- `_config.yml` — Jekyll site configuration

## Writing Posts

Posts live in `_posts/` with the filename format `YYYY-MM-DD-slug.md`.

Front matter template:
```yaml
---
title: "Post Title"
date: YYYY-MM-DD HH:MM:SS
author: Felipe Silvestre
categories: [ai, tech]  # or just [ai]
tags: [lowercase, hyphen-separated, tags]
---
```

Use the **`new-blog-post` skill** (`/new-blog-post`) to create posts. It handles voice, structure, file naming, and supports two modes: from a YouTube URL or from a topic/prompt.

## Key Config

- Theme: `jekyll-theme-chirpy`
- Timezone: `Europe/Dublin`
- Markdown: kramdown with Rouge syntax highlighting (line numbers enabled on blocks)
- Plugins: jekyll-feed, jekyll-sitemap, jekyll-seo-tag, jekyll-archives
- Pagination: 10 posts per page
- Comments: disabled
