# AGENTS

## Repository purpose
This repository is a Quartz 5-based personal website for `kitmenke.com`, with migrated content from Hugo stored under `content/`. It also contains the local Quartz source and configuration used to build the site.

## What to edit
- `content/` — source markdown, blog posts, pages, uploads, and site content.
- `quartz.config.yaml` — site configuration, plugins, theme, analytics, and page settings.
- `quartz.ts` — optional TypeScript override for Quartz config/layout when YAML is insufficient.
- `quartz/` — Quartz engine source and plugin loader code if modifying generator behavior.

## What not to edit directly
- `public/` — generated build output.
- `docs/` when it is used as generated preview output by the build scripts.
- `.quartz/plugins/` — plugin install artifacts.

## Key commands
- `npm install` — install repository dependencies.
- `npm run install-plugins` — install Quartz plugins defined in `quartz.config.yaml` to `.quartz/plugins/`.
- `npm run prebuild` — runs `install-plugins` before build.
- `npm run quartz -- build -d public` — build the site output into `public/`.
- `npm run docs` — build and serve documentation output to `docs/`.
- `npm run check` — typecheck and run Prettier style checks.
- `npm run format` — format files with Prettier.
- `npm test` — run the repository test suite.

## Important conventions
- Site source lives in `content/`; generated files belong in `public/` or `docs/` only.
- New content should generally be added under `content/blog/` or in top-level markdown pages inside `content/`.
- Image and asset uploads belong in `uploads/` and should be referenced from markdown with their site-relative paths.
- Quartz plugin configuration is defined in `quartz.config.yaml`; any plugin additions or changes must be followed by `npm run install-plugins`.
- `quartz.config.yaml` already enables common Quartz plugins such as syntax highlighting, obsidian-flavored markdown, table of contents, backlinks, search, and more.
- The site uses `ignorePatterns` to skip `private`, `templates`, and `.obsidian`.

## Helpful references
- [Quartz config documentation](docs/configuration.md)
- [Quartz layout documentation](docs/layout.md)
- [Quartz hosting documentation](docs/hosting.md)
- `quartz.config.yaml`
- `quartz.ts`

## Notes for agents
- If asked to fix or extend the website, prefer content changes in `content/` and config changes in `quartz.config.yaml`.
- If asked to change build behavior, inspect `quartz.ts` and `quartz/` source.
- Avoid editing generated directories and installed plugin artifacts.
