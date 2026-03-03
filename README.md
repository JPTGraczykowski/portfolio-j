# JPTGraczykowski Portfolio

Personal portfolio site built with [Jekyll](https://jekyllrb.com/) and hosted on [GitHub Pages](https://pages.github.com/).

Live at: **https://JPTGraczykowski.github.io/portfolio-j**

## Stack

- Jekyll 4 + Minima theme
- GitHub Actions for CI/CD
- GitHub Pages for hosting

## Local development

```bash
bundle install
bundle exec jekyll serve --livereload
```

Opens at `http://localhost:4000/portfolio-j`.

> `_config.yml` changes require a server restart.

## Deployment

Pushes to `main` trigger the workflow in `.github/workflows/deploy.yml`, which builds the site and deploys it to GitHub Pages automatically.
