# CRM ↔ Notion Sync

Keeps a dbt Labs Solutions Architect's Salesforce opportunities and personal
Notion customer hub in sync — pulling pipeline/ARR data into Notion, pushing
your "Technical Sales Process" notes (next steps, POV health, deployment
details, etc.) back into Salesforce, and auditing for drift in between.

## Install

1. Get the packaged `crm-notion-sync.plugin` file (built from this folder),
   or accept the plugin if a colleague has shared it with you.
2. Upload or enable it in claude.ai.
3. Connect the **Salesforce** and **Notion** connectors in claude.ai. The
   plugin doesn't bundle its own connectors.

## First-run setup

Your settings live on a Notion page in your own workspace, titled exactly
**CRM sync settings**, as a single JSON code block. They aren't stored in the
plugin, so the plugin stays the same for every SA and you never need to edit
it.

Each time you say a trigger phrase, unless your settings link is saved in
your Claude preferences (see below), the skill asks whether you've already
created that page. It never searches Notion for it, because search can
miss the page and send you back into setup. If you have a page, paste its link
and the skill loads it. If you don't, it walks you through setup:

- finds your technical owner field and the exact way your name appears on
  your opps
- looks up your Salesforce User Id
- asks you to pick your churn-risk field (it never guesses — some orgs have
  two similarly named ones)
- finds your Notion customer-tracking database
- asks about fields that aren't your responsibility, known account
  aggregations, and coverage reassignments

It confirms every value with you before creating the page. You can also
create the page yourself: copy
`skills/crm-notion-sync/references/settings.example.json`, fill it in using
`skills/crm-notion-sync/references/settings-template.md`, and paste it into
a JSON code block on a page titled **CRM sync settings**.

To skip the question on future runs, add this to your Claude preferences
(Settings > Profile / About me): "My crm-notion-sync settings page is <URL>.
Fetch it directly; don't search." The skill shows this tip after setup or
after you paste a link, and never edits your preferences itself.

If settings can't be loaded or are incomplete, no job runs.

Then say **"Run Job 1"** to do a first read-mostly sync and confirm your
account matching looks right before anything writes to Salesforce.

## Trigger phrases

| Say this | It runs |
|---|---|
| "Run Job 1" | Salesforce → Notion sync, plus optional coverage-reassignment detection |
| "Run Job 2" | Notion → Salesforce sync for your Technical Sales Process fields |
| "Run maintenance" / "Check my system" | POV document link check + Salesforce/Notion picklist drift check |
| "Review my fields" | Guided, one-by-one field review (no automated diffing) |
| "Full sync" / "Run everything" | All of the above in sequence, one combined summary |
| "Run my sync" / "Check my opps" | Job 1 + maintenance check |

## The one rule that never bends

This plugin will never write to a Salesforce opportunity unless your name is
the one in the technical-owner field, checked fresh immediately before every
single write. No exceptions, regardless of how the job is run.

## Customizing for your book of business

Everything specific to you lives on your **CRM sync settings** Notion page —
your name, your Notion database, excluded fields, known multi-entity account
aggregations, and (optionally) a temporary coverage-reassignment tracker.
When a run confirms something new, like an account aggregation, the skill
updates that page after you approve the change. The skill logic itself
(`SKILL.md` and its `references/`) stays generic across SAs.

## Layout

```
crm-notion-sync/
├── .claude-plugin/plugin.json
├── README.md
└── skills/crm-notion-sync/
    ├── SKILL.md
    └── references/
        ├── field-mappings.md
        ├── settings-template.md
        ├── settings.example.json
        └── setup-guide.md
```

Everything the skill reads at runtime is inside `skills/crm-notion-sync/`,
because that's the only part claude.ai chat loads.
