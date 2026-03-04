# FSilestre AI — Blog

A blog about AI, agents, MCP, LLMs, and software engineering by Felipe Silvestre Morais.

Built with [Jekyll](https://jekyllrb.com/) and the [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) theme, hosted on GitHub Pages.

## 🚀 Running Locally

```bash
# Install dependencies
bundle install

# Serve locally
bundle exec jekyll serve

# Open http://localhost:4000
```

## ✍️ Writing a New Post

Create a file in `_posts/` with the format:

```
_posts/YYYY-MM-DD-your-post-title.md
```

With this front matter at the top:

```yaml
---
title: "Your Post Title"
date: YYYY-MM-DD HH:MM:SS -0300
categories: [Category, Subcategory]
tags: [tag1, tag2, tag3]
---
```

Then write your content in Markdown below the `---`.

## 📦 Deployment

Pushing to `main` automatically triggers the GitHub Actions workflow (`.github/workflows/pages-deploy.yml`) which builds and deploys the site to GitHub Pages.

> Make sure GitHub Pages source is set to **GitHub Actions** in your repo Settings → Pages.
