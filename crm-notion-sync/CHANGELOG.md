# Changelog

## 0.4.0

- New stage alignment check (Job 1, Full Sync, or "Check stage alignment").
  Read-only alert when Salesforce `StageName` is Scope, Technical Validation
  or later but the SA's Notion Technical Sales Process is behind, so the SA
  can talk to the opp owner or update Notion. Never writes to Salesforce or
  Notion. Mapping in `references/stage-alignment.md`; switchable with the
  optional `stage_alignment.enabled` setting (default on, schema_version
  still 1).
- New optional Fivetran cross-sell fields guide
  (`references/fivetran-crosssell-fields.md`) and a "Set up my Fivetran
  fields" starter. `Fivetran Customer` and `Fivetran Fit Tier` are
  Notion-only, SA-maintained, and never touched by the sync jobs. Schema
  check lists them as optional.
- Job 1 summary notes accounts with a blank Fivetran Fit Tier when the
  property exists.

## 0.3.1

- README: added a "Before you start" section covering the Notion customer hub
  prerequisite (with a duplicable template and example page) and that notes
  must live in Notion; other note apps aren't supported.

## 0.3.0

- Settings discovery no longer searches Notion for the "CRM sync settings"
  page. Search is semantic and unreliable, and a missed result sent users into
  first-time setup, re-asking confirmed values and creating duplicate pages.
- New pointer-or-ask flow: fetch a known link directly (preferences, memory, or
  command arguments); otherwise ask the user to paste the link or confirm they
  need setup. Nothing runs or writes until they answer.
- Added a save-the-link tip after first-time setup or a pasted link, so the
  page is found automatically next time.
- setup-guide.md gains a step 0 confirming no page exists, and ends by
  displaying the tip.

## 0.2.0

- Baseline: per-SA settings on a Notion page, Jobs 1 and 2, maintenance check,
  field review, and the technical-owner safety rule.
