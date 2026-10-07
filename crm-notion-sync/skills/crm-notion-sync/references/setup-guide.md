# Setup Guide

Run this guided first-time setup only when the user has said they don't
have a CRM sync settings page yet (see "Load settings first" in `SKILL.md`).
It discovers what it can, confirms every value with the user, and then
creates the settings page. Nothing is saved until the user has confirmed every value.

Every setting is documented in `references/settings-template.md`. Start from
the structure in `references/settings.example.json` and keep its keys
exactly.

## 0. Confirm there's no existing settings page

Don't search Notion for one. Unless the user already answered this in the
current run, ask: "Have you already created your CRM sync settings page in
Notion? If so, paste the link here. If not, I can walk you through
first-time setup." Options: "Yes, I'll paste the link" / "No, set it up for
me". If they paste a link, stop setup and load that page as a normal run
would. Continue only on "No", and don't write anything before they've
answered.

## 0.5. Connect Salesforce + Notion

Make sure both connectors are available in this session before starting. If
a tool call fails with an auth error, ask the user to reconnect through their
organization's normal connector flow, then resume setup.

## 1. Discover and confirm each value

Work through these in order. For each one, show what you found and how you
found it, and get the user's explicit confirmation (or correction) before
moving on. Never save a value the user hasn't confirmed.

1. **`salesforce.technical_owner_field`** — run `describe_object` on
   Opportunity and list fields whose label or API name mentions "technical
   owner." Show label, API name, and type for each candidate. If there's
   exactly one, propose it; if there are several, ask the user to pick.
2. **`sa_name`** — ask the user for their name, then find the exact string
   Salesforce uses:
   ```sql
   SELECT {technical_owner_field}, COUNT(Id)
   FROM Opportunity
   WHERE {technical_owner_field} LIKE '%<user's last name>%'
   GROUP BY {technical_owner_field}
   ```
   Then show a sample of up to 5 opps for the candidate value (Name,
   Account.Name, StageName, CloseDate) so the user can confirm they're theirs.
   The confirmed value must match the field exactly, character for character.
3. **`salesforce.technical_lead_user_id`** —
   ```sql
   SELECT Id, Name, Email FROM User WHERE Name = '{sa_name}' AND IsActive = true
   ```
   Show the result and confirm. If zero or several rows come back, ask the
   user rather than picking one.
4. **`salesforce.churn_risk_field`** — from the same `describe_object`
   result, list **every** field whose label or API name mentions churn,
   contraction, or risk, with label, API name, and type. Some orgs have two
   similarly named fields (a personal one and an org-wide one). **Don't
   pre-select or recommend one.** The user must name the field that's theirs,
   or say they don't track it (save `null`).
5. **`notion.database_name` and `notion.data_source_url`** — search Notion
   for the user's customer-tracking database (try "Customer Hub", the user's
   name, or ask what it's called). Fetch the candidate and read its
   `collection://...` data source id from the result. Show the database name,
   link, and data source id, and confirm. Then check its schema against
   section 2 below and list any missing properties.
6. **`excluded_fields`** — show the Technical Sales Process fields from
   `references/field-mappings.md` and ask which, if any, aren't the user's
   responsibility. Default: `[]`.
7. **`known_account_aggregations`** — ask whether the user already knows of
   SF accounts that should roll up to a single Notion page. Default: `{}`.
   The skill adds more later as it confirms them.
8. **`coverage_area_tracking`** — ask whether they're mid-reassignment
   between Sales Directors. If yes, set `enabled: true` and collect the Owner
   names they're picking up coverage for in `owners_in_scope`. Default:
   `{"enabled": false, "owners_in_scope": []}`.

## 2. Confirm the Notion schema matches

The customer-tracking database needs these properties for Job 1 / Job 2 to
work:

**Core/account fields**: Account Name (title), Customer Journey (select),
Platform (multi_select), SA Account Health (select), Projected Opp/Renewal
Close (date), No Opp/Renewal Open (checkbox), My Account (checkbox),
Deprioritize (checkbox), CSA (person), Sales Director (person), Current ARR
(number), Next Opp Delta ARR (number), Largest Delta ARR (number), Sales Play
(select), Technical Sales Process (select — your own internal stage tracker,
separate from Salesforce StageName).

**Technical Sales Process fields**: see
`references/field-mappings.md` for the full list and exact types. Every
select field's options need to match your live Salesforce picklist — run
Maintenance Check once you're connected to backfill any gaps (it auto-adds
missing options, never auto-removes).

Missing properties don't block saving settings, but tell the user which jobs
will fail until they're added.

## 3. Create the settings page

1. Show the complete JSON, with every confirmed value, and ask for a final
   confirmation.
2. Create a private page in the user's Notion workspace (no parent) titled
   exactly **CRM sync settings**. Content: one sentence explaining the page
   is read by the crm-notion-sync skill and should stay a single valid JSON
   code block, followed by the JSON in one code block with language JSON.
3. Fetch the new page and parse it back, exactly as a normal run would, to
   confirm it loads. Give the user the page link.
4. **Save-the-link tip (final step).** Display: "Tip: add this to your
   Claude preferences (Settings > Profile / About me) so I find it
   automatically next time: 'My crm-notion-sync settings page is <URL>.
   Fetch it directly; don't search.'" with the new page's URL filled in.
   Never write to preferences yourself.

## 4. First run

Start with "Run Job 1" on its own before trying Full Sync — it's read-mostly
(it only auto-writes numeric/date fields and flags person fields) and is the
easiest way to confirm the account matching is working before Job 2 starts
writing into Salesforce.

## Editing settings later

The user can edit the JSON on the settings page directly; it's re-read at the
start of every run. If the skill confirms something new during a run (an
account aggregation, an excluded field), it updates that page after the user
confirms — never a file in this skill.
