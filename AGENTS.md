# homelab-ngx

A home server whose core is not touched when a service updates.

A static site built with [seite](https://seite.sh): markdown in, HTML + markdown + `llms.txt` out.

## Workflow

1. Run `seite context` once per session: config, collections, theme, dev server, pending upgrades.
2. Work through the `seite` skill: `/seite <command>` (Codex: `$seite <command>`), e.g. `check`, `new`, `preview`, `theme`, `deploy`.
3. Reference: `.seite/docs/` (docs for this seite version, written by seite and gitignored) or `seite docs <query>`.

## Layout

```
content/<collection>/  markdown + YAML frontmatter
templates/             Tera templates; base.html is the theme
data/                  YAML/JSON/TOML -> {{ data.<file> }}
static/                copied to dist/static/
public/                copied to the dist/ root (favicon.ico)
seite.toml             site config
dist/                  build output (generated)
```

## Collections

| Collection | Files | URL | Template |
|---|---|---|---|
| Pages (`seite new page`) | `content/pages/slug.md` | `/<slug>` | `page.html` |

## Content format

```yaml
---
title: "Title"           # required
description: "..."       # meta, Open Graph, JSON-LD
image: /static/og.png    # social preview
updated: 2026-02-01      # last modified
tags: [a, b]             # tag pages and feeds
draft: true              # left out of builds
weight: 1                # order in undated collections
extra: {key: value}      # -> {{ page.extra.key }}
---
```

## Rules

- Link between content files with relative `.md` paths (`[intro](../docs/intro.md)`); the build turns them into URLs.
- Never edit `dist/`.
- Templates extend `base.html`; render content with `{{ page.content | safe }}`.
- Themes: keep the `{% include "seite/…" %}` partials; seite updates them.
- Shortcodes: `{{< name(arg="v") >}}`, or `{{% name() %}}…{{% end %}}` with a body. No `{{< /name >}}`.

## Verify

Run `seite check` after changes. Fix every error at its `file:line`, run `seite check` once more, then stop and report what is left. Hooks (`.claude/settings.json`) run `seite check --file` after each edit and `seite check` when a turn ends, and hand errors back to you; `seite hooks status|on|off` manages them.

## Agent setup

<!-- seite:agent-setup -->
| Agent | Config | One-time step |
|---|---|---|
| Claude Code | `.claude/settings.json`, `.claude/launch.json` | Accept the workspace trust prompt once (until then Claude Code ignores `.claude/settings.json`) |
<!-- /seite:agent-setup -->

## Context Rules

<!-- seite:context-rules -->
Detailed guides live in `.claude/rules/`. Claude Code loads them automatically for matching files; other agents should read the matching guides before editing:

- `templates/**`: `seo-requirements.md`, `templates.md`, `i18n.md`, `features.md`
- `content/**`: `i18n.md`, `shortcodes.md`, `features.md`, `private-collections.md`
- `data/i18n/**`: `i18n.md`
- `data/**`: `data-files.md`
- `templates/shortcodes/**`: `shortcodes.md`
- `seite.toml`: `config-reference.md`, `private-collections.md`
<!-- /seite:context-rules -->

## This repo

- **seite is pinned to v0.21.3** (pre-1.0). The deploy workflow installs that
  release by checksum. Locally use the same version, with `DO_NOT_TRACK=1` set:
  seite sends telemetry outside CI otherwise. Upgrade the workflow pin and the
  local binary together.
- **Deploys**: a push to `main` builds and deploys to GitHub Pages through
  `.github/workflows/deploy.yml`. Pages' source is GitHub Actions, never a
  `gh-pages` branch, so `seite deploy` is not used.
- The repo, Pages and the domain's DNS belong to `homelab-ngx/infra`
  (OpenTofu). Change them there, not in the settings UI.
- `public/` is copied to the site root: the `go-import` pages for
  `homelab-ngx.com/daemon` and `/backup-runner` go there (infra #6).
