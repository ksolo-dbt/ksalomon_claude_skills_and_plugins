# Settings Template

Every SA's settings live on a Notion page in their own workspace, titled
exactly **CRM sync settings**. The page holds a single JSON code block
(language: JSON) and nothing else that looks like JSON. Short notes in plain
text above or below the code block are fine.

A ready-to-copy starting point with placeholder values is in
`references/settings.example.json`. Guided first-time setup
(`references/setup-guide.md`) builds this page for you; this file documents
what each value means if you want to fill it in or edit it by hand.

## Validation rules

Settings are rejected (and no job runs) if any of these are true:
- The page doesn't contain exactly one JSON code block, or the JSON doesn't
  parse.
- A **required** value is missing, empty, or still placeholder text (ALL CAPS
  instructional text like `YOUR NAME EXACTLY AS IT APPEARS...`).
- A value has the wrong type (for example, `owners_in_scope` isn't a list).

When rejecting, say exactly which key is wrong and what it should look like.
Never fill in or guess a missing value.

## Settings

### `schema_version` (required)
- **Meaning**: version of this settings format. Currently `1`.
- **Example**: `1`

### `sa_name` (required)
- **Meaning**: your name exactly as it appears in the technical owner field
  on your Salesforce opps. This is the value the hard safety rule compares
  against before every write, so it has to match character for character,
  including punctuation and middle initials.
- **How to find it**: open a few of your own opps and copy the technical owner
  value. First-time setup queries a sample of opps and shows you the exact
  strings it finds.
- **Example**: `"Jane Q. Example"`

### `salesforce.technical_owner_field` (required)
- **Meaning**: API name of the Opportunity field that marks the SA as
  technical owner.
- **How to find it**: run `describe_object` on Opportunity and look for the
  technical owner field. Confirm with Sales Ops if more than one candidate
  exists. Don't assume it matches another org's field name.
- **Example**: `"Technical_Owner_Name__c"`

### `salesforce.technical_lead_user_id` (required)
- **Meaning**: your Salesforce User Id.
- **How to find it**: first-time setup runs
  `SELECT Id, Name, Email FROM User WHERE Name = '{sa_name}' AND IsActive = true`
  and asks you to confirm the result. You can also ask your Salesforce admin.
- **Example**: `"005XXXXXXXXXXXXXXX"`

### `salesforce.churn_risk_field` (required key, value may be `null`)
- **Meaning**: API name of *your* contraction/churn-risk checkbox on
  Opportunity, synced by Job 2.
- **How to find it**: run `describe_object` on Opportunity and list every
  field whose label or API name mentions churn, contraction, or risk. Some
  orgs have two similarly named fields: a personal one that's yours and an
  org-wide one that isn't. **This value must be confirmed by the SA, never
  guessed or pre-selected.** If you don't track churn risk, set it to `null`
  and Job 2 skips that field.
- **Example**: `"Example_Churn_Risk__c"` or `null`

### `notion.database_name` (required)
- **Meaning**: display name of your personal customer-tracking Notion
  database. Used in summaries and to find the database during setup.
- **How to find it**: the title shown at the top of the database in Notion.
- **Example**: `"Jane Example's Customer Hub"`

### `notion.data_source_url` (required)
- **Meaning**: the `collection://...` data source id for that database, used
  for every Notion query.
- **How to find it**: first-time setup searches Notion for your database,
  fetches it, and reads the data source id from the result. It starts with
  `collection://`. A regular `https://` page link isn't the same thing.
- **Example**: `"collection://00000000-0000-0000-0000-000000000000"`

### `excluded_fields` (optional, default `[]`)
- **Meaning**: fields you've said aren't your responsibility. Jobs never
  write to them and Field Review doesn't ask about them. Use Salesforce API
  names.
- **How to find it**: setup shows the Technical Sales Process fields from
  `references/field-mappings.md` and asks which, if any, aren't yours.
  Typical examples are a deployment cloud provider field or manager-only
  forecast notes. Don't copy another SA's list.
- **Example**: `["Example_Cloud_Provider__c", "Example_Manager_Forecast_Notes__c"]`

### `known_account_aggregations` (optional, default `{}`)
- **Meaning**: Salesforce `Account.Name` values that roll up to a single
  Notion page, for multi-entity customers and post-merger or acquisition name
  changes you've already confirmed. Format: `"SF Account Name": "Notion
  Account Name"`. The skill adds entries here (after you confirm them) as it
  discovers them.
- **Example**: `{"Example Corp - Subsidiary A": "Example Corp (Parent)"}`

### `coverage_area_tracking` (optional)
- **`enabled`**: `true` to have Job 1 flag lag between Sales Director (Owner)
  reassignments and the technical owner field catching up. Default `false`.
- **`owners_in_scope`**: list of Opportunity Owner names you're picking up
  coverage for. Required to be non-empty when `enabled` is `true`.
- **Meaning**: a temporary-state feature for mid-reassignment periods. Once
  Sales Ops catches up, set `enabled` back to `false` rather than leaving
  stale names in the list.
- **Example**: `{"enabled": true, "owners_in_scope": ["Alex Director", "Sam Director"]}`
