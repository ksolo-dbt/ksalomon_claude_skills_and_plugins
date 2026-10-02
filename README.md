# Claude skills and plugins

Claude skills and plugins for dbt Labs solutions architects.

## Plugins

| Plugin | What it does |
|---|---|
| [crm-notion-sync](plugins/crm-notion-sync/) | Keeps an SA's Salesforce opportunities and personal Notion customer hub in sync. Per-SA settings live on a Notion page, not in the plugin. |

## Repo layout

- `plugins/`: one folder per plugin, each with its own README.
- `dist/`: packaged `.plugin` files for upload to claude.ai (not committed).
- `private/`: personal values and notes (not committed).

## Packaging a plugin

Zip the contents of a plugin folder (not the folder itself), leaving out
`.DS_Store`:

```bash
cd plugins/crm-notion-sync && zip -r ../../dist/crm-notion-sync.plugin . -x '*.DS_Store'
```
