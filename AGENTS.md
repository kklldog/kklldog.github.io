# Repository Guidelines

## Project Structure & Module Organization

This repository is a Jekyll site built with the Chirpy theme. Blog articles live in `_posts/` and follow Jekyll's `YYYY-MM-DD-title.md` filename format. Navigation pages are stored in `_tabs/`, shared metadata in `_data/`, and custom Ruby hooks in `_plugins/`. Site-wide settings, including locale, URL, pagination, and theme options, belong in `_config.yml`. Static dependencies are tracked through the `assets/lib` Git submodule; generated output is written to `_site/` and must not be committed. Helper scripts are in `tools/`, while deployment configuration is in `.github/workflows/pages-deploy.yml`.

## Build, Test, and Development Commands

- `bundle install` installs the Ruby gems declared in `Gemfile`.
- `git submodule update --init --recursive` initializes the Chirpy static assets after cloning.
- `bash tools/run.sh` starts the local Jekyll server with live reload at `127.0.0.1`.
- `bash tools/run.sh --host 0.0.0.0` exposes the preview to the local network or a container host.
- `bash tools/test.sh` creates a production build and validates internal links and generated HTML with `html-proofer`.
- `bundle exec jekyll build` performs a standard local build into `_site/`.

Use Ruby 3.3 when matching the GitHub Pages workflow.

## Coding Style & Naming Conventions

Follow `.editorconfig`: UTF-8, LF line endings, final newlines, two-space indentation, and trimmed trailing whitespace except where Markdown uses it intentionally. Use single quotes in JavaScript, CSS, and SCSS, and double quotes in YAML. Keep post front matter valid YAML and use existing keys such as `layout`, `title`, `categories`, and `tags`. Preserve Chinese content as UTF-8. Name new posts `YYYY-MM-DD-descriptive-title.md` and keep tab filenames lowercase.

## Testing Guidelines

There is no unit-test suite or coverage threshold. Treat `bash tools/test.sh` as the required pre-merge check. Preview changed posts locally and verify front matter, code blocks, images, navigation, and links. The link check deliberately skips external URLs, so inspect newly added external references manually.

## Commit & Pull Request Guidelines

Recent history uses short messages such as `post` and `update`; prefer a concise imperative message with more context, for example `post: add OpenTelemetry metrics guide`. Keep each commit focused. Pull requests should summarize the content or configuration change, identify affected pages, and report the test command used. Link relevant issues and include screenshots for layout, theme, image, or navigation changes. Do not commit `_site/`, `.jekyll-cache/`, `vendor/`, or `node_modules/`.
