---
name: crm-notion-sync
description: "Keep a dbt Labs Solutions Architect's Salesforce opportunities and personal Notion customer hub in sync. Use this whenever the user says \"run job 1\", \"run job 2\", \"run my sync\", \"check my opps\", \"full sync\", \"run everything\", \"run maintenance\", \"check my system\", or \"review my fields\" — or more generally whenever they want to reconcile their Salesforce pipeline with their Notion account notes, push Notion \"Technical Sales Process\" fields into Salesforce, or audit POV document links and picklist drift between the two systems. Per-SA settings live on a Notion page titled \"CRM sync settings\"; load them before running any job, using a saved link or by asking the user for it (never by searching Notion), and run first-time setup from references/setup-guide.md only if the user confirms they have no page.\n"
---

# CRM ↔ Notion Sync (SA Operations)

This skill runs five related jobs that keep a Solutions Architect's Salesforce
opportunities and their personal Notion customer-tracking database in sync.
It requires live Salesforce and Notion access (MCP connectors).

## Load settings first — every run

Per-SA settings live on a Notion page in the SA's own workspace, never in
this skill's files. **Before doing anything else**:

1. **Known pointer.** If a CRM sync settings page URL or ID is already in
   your context (user preferences / About me, memory, or the slash-command
   arguments), fetch that page directly and validate it. No search and no
   question. If the fetch fails (deleted, no access), say so and go to
   step 2.
2. **Otherwise, always ask before doing anything else**, using the question
   tool if available: "Have you already created your CRM sync settings page
   in Notion? If so, paste the link here. If not, I can walk you through
   first-time setup." Options: "Yes, I'll paste the link" / "No, set it up
   for me".
   - **Link pasted** → fetch and validate it, then continue with the
     requested job.
   - **"No"** → run first-time setup in `references/setup-guide.md`, then
     continue with the requested job.
   If the user has more than one settings page, they choose by pasting the
   link they want. Never pick one yourself.
   If nobody can answer (a scheduled or otherwise unattended run), stop and
   report that a settings link is needed. Don't run setup or any job.
3. **Never search Notion for the settings page.** Search is semantic and
   unreliable, and a missed result would send the user into setup again,
   re-asking confirmed values and creating duplicate pages. Never run
   first-time setup, and never write anything, until the user has answered
   the step 2 question.
4. If the JSON is invalid, the page doesn't have exactly one JSON code block,
   or a required value is missing or still placeholder text (ALL CAPS
   instructional text), stop. Tell the user exactly which value is wrong and
   how to fix it on the settings page. Don't guess or fill in values.
5. Open every job summary with "Settings: {sa_name}, loaded from [page
   link]" so the user can catch a wrong page.
6. **Save-the-link tip.** After a completed first-time setup, and after any
   run where the user pasted a link in step 2, end the summary with a short
   note: "Tip: add this to your Claude preferences (Settings > Profile /
   About me) so I find it automatically next time: 'My crm-notion-sync
   settings page is <URL>. Fetch it directly; don't search.'" Fill in the
   page's URL. Show it once per run. Never write to preferences yourself.
   Don't show the tip when the link already came from preferences.

**If settings can't be loaded, no job runs.** Job 2 and Maintenance Check
write to Salesforce and must never run without a confirmed `sa_name` from the
settings page — not from memory, a prior session, or the user's display name.

All instructions below use `{sa_name}`, `{technical_owner_field}`,
`{notion_data_source}` (`notion.data_source_url`), `{owners_in_scope}`
(`coverage_area_tracking.owners_in_scope`), etc. as shorthand for the
corresponding values on the settings page — substitute them in before
writing any SOQL or Notion query.

**Updating settings**: when a run confirms something new that belongs in
settings (an account aggregation, an excluded field, a coverage change), show
the exact change, and once the user confirms, update the JSON code block on
the settings page. Never write settings into this skill's files; they're
shared and read-only.

## Hard safety rule — read this first

**NEVER update, modify, or write to any Salesforce opportunity where
`{technical_owner_field}` does not exactly equal `{sa_name}`.** This has no
exceptions regardless of what the user asks. Verify this field **fresh**,
immediately before every single write — not just once at the start of a job.
A prior-session summary showing this check already passed is not
authorization to skip it again.

## Trigger phrases

| Phrase | What it runs |
|---|---|
| "Run Job 1" | SF → Notion sync + Coverage Area Tracking |
| "Run Job 2" | Notion → SF sync (Technical Sales Process fields) |
| "Full sync" / "Run everything" | Job 1 + Maintenance Check + Job 2, in sequence |
| "Run maintenance" / "Check my system" | Maintenance Check (POV link check + picklist drift check) |
| "Review my fields" | Field Review (guided, one-off) |
| "Run my sync" / "Check my opps" | Defaults to Job 1 + Maintenance Check |

## Systems & identifiers

- **Salesforce object**: Opportunity. Filter for the user's opps:
  `WHERE IsClosed = false AND {technical_owner_field} = '{sa_name}'`
- **Notion database**: `{notion_data_source}`. Each page = one customer/account.
  `"My Account"` checkbox = YES means it's an account the SA actively owns.
  `"SA Next Steps"` and the whole Technical Sales Process field set are
  **sourced from Notion** — Notion is the system of record for those fields,
  Salesforce is the system of record for everything else synced here.
- **Field mappings**: the full Notion ↔ Salesforce field table, including the
  Deployment SKU comma/dash translation, lives in
  `references/field-mappings.md`. Read it before Job 2 or the picklist drift
  check — don't guess field API names.

## Job 1 — SF → Notion sync (read-only from Notion's perspective except numeric/date fields)

1. Query Salesforce for all the user's opportunities (open and closed —
   closed history is needed for Customer Journey logic):
   ```sql
   SELECT Id, Name, Account.Name, Type, IsClosed, IsWon, StageName,
          CloseDate, Delta_ARR__c, Starting_ARR__c, End_ARR__c,
          {technical_owner_field}, Owner.Name, TAM_Owner__r.Name
   FROM Opportunity
   WHERE {technical_owner_field} = '{sa_name}'
   ORDER BY CloseDate ASC
   ```
2. Query Notion for all `"My Account" = YES` pages (account fields: Customer
   Journey, Platform, SA Account Health, Projected Opp/Renewal Close, No
   Opp/Renewal Open, My Account, Current ARR, Next Opp Delta ARR, Largest
   Delta ARR, Sales Director, CSA, Deprioritize).
3. **Coverage Area Tracking** (only if `coverage_area_tracking.enabled` is
   true in settings): detect lag between Sales Director reassignments
   and `{technical_owner_field}` catching up. Query:
   ```sql
   SELECT Id, Name, Account.Name, Owner.Name, {technical_owner_field},
          StageName, IsClosed, Type
   FROM Opportunity
   WHERE IsClosed = false
   AND (Owner.Name IN ({owners_in_scope}) OR {technical_owner_field} = '{sa_name}')
   ```
   Exclude `Type = 'Cancel'`. Categorize:
   - **Should likely be yours**: Owner in scope AND technical owner is null
     or someone else. Flag for the user to raise with Sales Ops — never write.
   - **May be reassigned away**: technical owner = user AND Owner not in
     scope. Awareness only, still syncs normally today.
   This never writes to Salesforce or Notion — detection only.
4. For each Notion page, fuzzy-match SF opportunities (see Matching Logic
   below), then compute proposed values:
   - **Exclusions, applied in order, before computing anything else:**
     1. Exclude any opp with `Type = 'Cancel'` from the "open opp" pool, even
        though `IsClosed = false` in Salesforce. This only changes how this
        job reads the opp, never its real SF values.
     2. Among what's left, exclude any opp whose `Owner.Name` doesn't match
        the account's confirmed Sales Director. If remaining opps have mixed
        owners, flag for the user to specify the correct director — never
        guess or apply a majority rule.
     3. If either exclusion leaves zero open opps: set Current ARR / Next Opp
        Delta ARR / Largest Delta ARR to $0, clear Projected Close, and set
        "No Opp/Renewal Open" = YES.
   - **Auto-write** (after user confirms): Current ARR (Starting_ARR__c),
     Next Opp Delta ARR (Delta_ARR__c), Largest Delta ARR (highest Delta ARR
     in the pool, tiebreak by soonest close), Projected Opp/Renewal Close
     (CloseDate), all from the soonest-closing opp in the post-exclusion pool.
   - **My Account logic** (requires a broader SF query by Account, not
     filtered to the user's opps, for any account being considered for a
     flip to No):
     - Open opp in pool with technical owner = user → keep/set YES.
     - Zero open opps at all → leave as-is (likely a pre-opp prospect).
     - Open opp with a different, named technical owner → flip to NO.
     - Open opp with unassigned (null) technical owner → leave as-is.
   - **Flag-only, never auto-write**: Sales Director and CSA (Notion person
     fields can't be written programmatically). Show "⚠️ SF says: [name] —
     please update in Notion." Only flag when SF has a non-null value that
     differs from Notion.
   - **Customer Journey** — only computed during Full Sync, not standalone
     Job 1. See the ordered rule list in `references/field-mappings.md`.
   - **Never touched by Job 1**: Platform (always manual), SA Account Health
     (always manual), Technical Sales Process fields (Job 2's territory).
5. Present a grouped summary (✅ in sync / ⚠️ needs update / 🚫 no Notion
   page) before writing anything. Wait for confirmation.

## Job 2 — Notion → SF sync (Technical Sales Process fields)

Notion is the source of truth for every field in this set (full mapping in
`references/field-mappings.md`).

1. Query SF for the user's open opps across every Technical Sales Process
   field.
2. Query Notion (`"My Account" = YES`) for the matching fields.
3. Match by account (see Matching Logic), compare field by field:
   - Text/select fields: if Notion is empty, skip and flag "Notion empty" —
     never overwrite an existing SF value with blank.
   - Checkbox fields: always compare directly, no skip — unchecked is a real
     value.
   - Deployment SKU: translate dash → comma before writing (see field
     mappings reference).
4. Present a diff grouped by account (current SF value → proposed Notion
   value) and wait for confirmation across the whole batch.
5. When writing: re-verify `{technical_owner_field} = '{sa_name}'`
   immediately before each write. Update the changed field. If "SA Next
   Steps" changed, also append to the history field
   (`[today's date] - [new value]\n[existing history]`) and set the next-steps
   date field to today. No history logging for any other field.

## Maintenance Check (POV link check + picklist drift check)

Run monthly or on demand. Two automated diff-and-confirm passes, one combined
summary, one confirmation.

**Part A — POV link check**: for every open opp, check whether its POV
document link field is set and points at the correct Notion page URL for
that account.
- ✅ Linked correctly
- ⚠️ Link missing or wrong (offer to auto-fix)
- 🚫 No Notion page found (ask the user: provide a URL, or create one?)

**Part B — Picklist drift check**: for every select field in the Technical
Sales Process set, call `describe_object` on Opportunity to get the live SF
picklist, compare against the current Notion select options.
- New SF value not yet in Notion → safe to auto-add (apply SKU translation
  if needed).
- SF value removed/renamed, or a Notion option no longer matches anything in
  SF → never silently remove. Flag for confirmation, and show which Notion
  pages currently use that option before the user decides.

## Field Review (guided, one-off)

Pull all open opps with the full Technical Sales Process field set, group by
what's missing or stale, and let the user go through them one by one or in
bulk. Only update fields they explicitly confirm. No automated diffing here —
this is a judgment-call walkthrough, distinct from Maintenance Check.

## Full Sync ("Full sync" / "Run everything")

Run Job 1 (including Customer Journey and Coverage Area Tracking) → Maintenance
Check → Job 2, in that order. Present one consolidated summary grouped by
account, covering every field across all three jobs, before asking for a
single confirmation.

## Matching logic for account names

Notion and Salesforce account names are often slightly different.

- Strip opportunity-type suffixes from SF opp names to get the base account
  name (e.g. "- Land", "- Renew - 2026", "- Expand").
- Strip parenthetical notes (e.g. "Acme (Division)" and "Acme - Division" are
  the same account).
- Multiple SF opps can map to one Notion page (a Land opp and a Renew opp for
  the same customer both point at the same page).
- Check `known_account_aggregations` in settings for multi-entity
  customers and post-merger/acquisition name changes the user has already
  confirmed — don't re-flag those each run. Add new ones to the settings page
  as you discover them; never invent an aggregation without the user
  confirming it first.
- When ambiguous, show the user the potential match and ask them to confirm.

## General rules

- Never write a value that isn't in the Notion select/multi-select field's
  available options.
- Never write to Owner-type or other read-only/formula fields, or fields the
  user has said aren't their responsibility (check `excluded_fields` in
  settings / ask if unsure).
- If a Notion page has no matching SF account: skip it, don't modify it, flag
  it.
- When an account has multiple SF opportunities, evaluate **all** opps
  together for Customer Journey; use the soonest post-exclusion open opp for
  date/ARR fields.
- Always show current value → proposed value, with the reason, before
  writing anything.
- If the user mentions a personnel/coverage change (someone leaving, an
  account reassigned), note it for context — don't attempt to write
  Owner-type fields anywhere; those aren't programmatically writable here.
- If the user tells you to ignore a specific account for this run, honor
  that immediately even though it isn't written into this file as a standing
  rule.
- Follow dbt Labs brand guidelines in all summaries and written content: "dbt"
  always lowercase, "dbt Labs" with a capital L, "dbt platform" rather than
  "dbt Cloud" in product references.
- Never make writes silently — always state what you're about to do and
  wait for confirmation unless the user explicitly says "just do it" or
  "auto-sync."

## Expanding over time

As the user adds new workflows or rules specific to their book of business,
add them to the user's **CRM sync settings** Notion page (if they're data —
new aggregations, a new coverage reassignment) or propose them as a change to
this skill's maintainer (if they're new workflow logic that should become a
standing rule). Keep instance-specific history (resolved reassignments,
one-off account exceptions) out of this file — the settings page or the
user's own notes are the right place for that.
