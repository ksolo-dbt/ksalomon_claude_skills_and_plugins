# Fivetran cross-sell fields

> **Optional, SA West Fivetran cross-sell. These fields may not be permanent.**

Two optional Customer Hub properties support the SA West Fivetran cross-sell
motion. They are **Notion-only and maintained by the SA by hand**. The
plugin never reads them from or writes them to Salesforce, and Job 1, Job 2,
Maintenance Check and Full Sync never touch them. The only thing that writes
them is the optional starter below, after the SA confirms.

This is a starter guide, not an automation. A first pass takes about 5-10
minutes. The SA's own judgment is the final word.

| Notion property | Type | Meaning |
|---|---|---|
| `Fivetran Customer` | Checkbox | Checked if Account Intelligence shows an existing Fivetran account. Route to the account team, no lead pass. |
| `Fivetran Fit Tier` | Select | `Strong`, `Maybe`, `Not a fit` |

A missing property never blocks anything. If either is missing, skip it and
say so.

## Rubric

Source: the [SA West Fivetran Cross-Sell Playbook](https://claude.ai/artifact/STRhh5fHYmCedLctbw3HT7).
This is a condensed version.

- **Strong**: complex SaaS/ERP sources (Workday, Salesforce, Jira, NetSuite)
  feeding a homegrown pipeline with accuracy or consistency pain, plus at
  least one secondary signal.
- **Maybe**: good sources but ingestion is unknown, or homegrown ingestion
  that works fine today.
- **Not a fit**: a disqualifier is present (competing ingestion tool signed
  or renewed in the last 12 months, strong build-it-ourselves culture,
  already a Fivetran customer) or the sources aren't ones Fivetran serves
  well.
- **Secondary signals**: small data engineering team with a backlog,
  pipeline breakage or stale dashboards mentioned on calls, warehouse
  migration underway, engineers maintaining connectors they'd rather not own.

## Pass 1: Claude-assisted first draft ("Set up my Fivetran fields")

Optional, one-time, on request only. Writes to Notion only (these two
properties), never to Salesforce.

**Data source:** Ryan Waters' dashboard, ["West cross-sell: Fivetran × dbt"](https://claude.ai/artifact/PSXGDRX5AkdwvdkthRmpHo?sk=PgjWLIXizgxEvzUUDsB2gg).
Open it and read its data. It is a West-territory-only snapshot dated
2026-10-12, with one row per dbt account matched to a Fivetran Salesforce
account (about 276 rows). SAs outside West, or working well after the
snapshot date, will need a fresh pull; say so instead of using stale data.

Columns: `account_name`, `dbt_account_owner_name`, `ft_account_owner_name`,
`dbt_account_status`, `ft_account_status`, `dbt_arr`, `model_ft_all_arr`,
`ft_account_id`, `dbt_account_id`, `has_direct_dbt_core_90d`,
`has_hosted_dbt_core_90d`, `dbt_subscription_end_date`,
`ft_subscription_end_date`.

- `ft_account_status`: `Customer`, `Prospect`, `Previous Customer`.
- `dbt_account_status`: `Customer`, `Prospect - Pipeline`,
  `Prospect - Pre-Pipeline`, `Churned Customer`.
- The dashboard groups accounts into four motions: Fivetran cross-sell (dbt
  customer, not on Fivetran), dbt cross-sell, joint customer, joint pursuit.

For each Notion page where `My Account` = YES:

1. Match to dashboard rows with the plugin's account-matching logic (strip
   suffixes and parentheticals, honor `known_account_aggregations`, ask when
   ambiguous). Several dbt accounts can map to one Fivetran account.
2. Propose `Fivetran Customer` = checked when `ft_account_status` =
   `Customer`. For `Previous Customer` (a win-back), do **not** check it; show
   it as a hint.
3. Propose `Fivetran Fit Tier` = `Not a fit` only for accounts that are
   already Fivetran customers. **Never assign Strong or Maybe from this
   data**: it has no ingestion or pain information. Show hints per account
   instead: dbt Core in use in the last 90 days, hosted dbt Core in use,
   Fivetran status (`Prospect` vs `Previous Customer`), number of dbt
   accounts mapped, ARR, and renewal dates.
4. Show one table (account, current value → proposed value, reason). Write
   only after the SA confirms. Only fill properties that are currently empty
   unless the SA explicitly says to overwrite. List unmatched accounts
   instead of guessing.

## Pass 2: the SA's manual review (where the real scoring happens)

1. Correct any `Fivetran Customer` flag that looks wrong. Account
   Intelligence is the source of truth if it disagrees with the snapshot.
2. Score `Fivetran Fit Tier` for the remaining accounts with the rubric. Per
   account: search Gong for ingestion mentions (Airflow, Airbyte, Stitch,
   custom scripts, "pipelines break", "stale data"), list the SaaS/ERP
   sources feeding the dbt project, then assign a tier.
3. Record reasons in the account's regular notes, not as extra fields.
4. After that, maintain both fields by hand. Re-run the starter only for new
   accounts.

## Supplemental check

The [Hex dbt × Fivetran cross-sell dashboard](https://app.hex.tech/fivetran/app/dbt-x-Fivetran-Cross-Sell-033J0RRC0cpdBQ8qX2ctNv/latest?tab=-account-coverage),
**Account Coverage** tab, is a second reference for the SA's own account
coverage. Use it to double-check the fields in Notion after the starter.
