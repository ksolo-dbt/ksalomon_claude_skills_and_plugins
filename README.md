# Claude skills and plugins

Claude tools for dbt Labs solutions architects. Each project lives in its own
top-level folder with its own README and version.

## Projects

| Project | Runs in | What it does |
|---|---|---|
| [crm-notion-sync](crm-notion-sync/) | claude.ai (uploaded as a `.plugin` file) | Keeps an SA's Salesforce opportunities and personal Notion customer hub in sync. Per-SA settings live on a Notion page, not in the project. |

## Repo layout

- `<project>/`: one folder per project, each with its own README.
- `dist/`: packaged `.plugin` files for upload to claude.ai (not committed).
- `private/`: personal values and notes (not committed).

Tools cloned from other people don't live here. Keep those as their own
clones (for example under `~/Documents/claude-external-shared/`).

## Versioning

Each project tracks its own version (for crm-notion-sync, `version` in
`.claude-plugin/plugin.json`). Tag releases per project, for example
`crm-notion-sync-v0.2.0`, and view one project's history with
`git log -- <project>/`.

## Installing and updating (marketplace)

This repo is a plugin marketplace (`.claude-plugin/marketplace.json`). Each new
project needs one more entry in that file.

1. One time: in Claude Desktop, open Customize > Plugins > Add > Add
   marketplace and point it at this GitHub repo.
2. To update: edit the project, bump `version` in its
   `.claude-plugin/plugin.json`, then commit and push.
3. In Claude Desktop, open Customize > Plugins and select Check for updates.

## Packaging for claude.ai (manual upload)

Zip the contents of the project folder (not the folder itself), leaving out
`.DS_Store`:

```bash
cd crm-notion-sync && zip -r ../dist/crm-notion-sync.plugin . -x '*.DS_Store'
```
