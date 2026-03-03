# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Goal

A personal portfolio site built with Jekyll, hosted on GitHub Pages. It presents the owner and their GitHub projects. No backend — purely static front-end.

## Commands

```bash
# Install dependencies
bundle install

# Serve locally with live reload (http://localhost:4000)
bundle exec jekyll serve --livereload

# Build static site into _site/
bundle exec jekyll build

# Build with drafts visible
bundle exec jekyll serve --drafts
```

> **Note:** `_config.yml` changes require a server restart — they are not hot-reloaded.

## GitHub Pages Deployment (via GitHub Actions)

The site deploys via GitHub Actions using Jekyll 4 directly — no `github-pages` gem required.
The workflow is at `.github/workflows/deploy.yml` and triggers on every push to `main`.

**One-time setup:**

1. In `_config.yml`, set `baseurl: "/repo-name"` (e.g. `"/portfolio-j"`) and `url: "https://<username>.github.io"`
2. Go to repo **Settings → Pages → Source** and select **GitHub Actions**
3. Push to `main` — the Actions tab will show the workflow running
4. After a successful deploy, the site is live at `https://<username>.github.io/<repo-name>`

> For a user/org page (`username.github.io`), keep `baseurl: ""`.

## Architecture

- **`_config.yml`** — site-wide settings: `title`, `github_username`, `baseurl`, `url`, theme, plugins
- **`index.markdown`** — home page, uses `layout: home` from the Minima theme
- **`about.markdown`** — about page at `/about/`, uses `layout: page`
- **`_posts/`** — blog posts (if used); filename format: `YYYY-MM-DD-title.md`
- **`_layouts/`** — custom layout overrides (not yet created; Minima defaults apply)
- **`_includes/`** — reusable HTML partials to override Minima defaults
- **`assets/`** — static files (CSS, images, JS)

## Theme: Minima

The site uses [Minima](https://github.com/jekyll/minima) (~2.5). To override a theme file:

1. Find the source: `bundle info --path minima`
2. Copy the file into the matching path in this repo (e.g. `_layouts/home.html`, `_includes/header.html`)
3. Edit the copy — Jekyll uses local files over gem files

## Fetching GitHub Projects

There is no build-time GitHub API integration yet. Options:
- **Client-side JS**: Fetch `https://api.github.com/users/<username>/repos` at runtime and render with JS
- **Jekyll data file + CI**: Use a script to write `_data/projects.yml` before build, then use `site.data.projects` in templates

## Key Config Values to Set

In `_config.yml`:
```yaml
title: <your name or site title>
email: <your email>
github_username: <your GitHub handle>
baseurl: ""          # "" for user/org pages (username.github.io), "/repo-name" for project pages
url: "https://<username>.github.io"
```
