# Stage Alignment

Used by the stage alignment check (Job 1, Full Sync, and "Check stage
alignment"). **Read-only: this check never writes to Salesforce or Notion.**
Stage is owned by the rep (the Sales Director or account manager), so the
check only alerts the SA.

## What it catches

Reps sometimes move an opp into Scope, Technical Validation, or a later
stage when technical validation hasn't started or happened, and sometimes
leave it there when the deal belongs back in Discover. The SA's Notion
`Technical Sales Process` is the SA's own view. When Salesforce is ahead of
it, the SA should talk to the opp owner, or update Notion if Salesforce is
right.

## Comparison

- SA's view: Notion `Technical Sales Process` on the account page.
- Salesforce's view: Opportunity `StageName`. There is no separate
  technical-stage field on Opportunity.

### Order of `Technical Sales Process` values (earliest to latest)

| Rank | Value |
|---|---|
| 0 | Pre-Engagement |
| 1 | Discovery |
| 2 | Tech Validation Scoping |
| 3 | In Tech Validation |
| 4 | Tech Win |

### Minimum value each Salesforce stage implies

| Salesforce `StageName` | Notion value must be at least | Result if lower |
|---|---|---|
| Scope | Tech Validation Scoping (2) | ⚠️ |
| Technical Validation | In Tech Validation (3) | ⚠️ |
| Business Justification | Tech Win (4) | ⚠️ |
| Proposal | Tech Win (4) | ⚠️ |
| Negotiate | Tech Win (4) | ⚠️ |
| Signature Ready | Tech Win (4) | ⚠️ |
| Engage, Discover | no expectation | ✅ (not checked), except Tech Loss above |
| Renewal Planning, Renewal Discussion | no expectation | ❓ can't compare |

**Tech Win is where the SA's tracking usually ends; the rep's stages continue.**
With Notion at Tech Win, Salesforce at Business Justification, Proposal,
Negotiate, Signature Ready, Closed Won or Closed Lost is correct (✅). The
⚠️ case is the rep moving ahead before technical validation is complete: for
example Notion says In Tech Validation (or earlier) while Salesforce says
Business Justification, Negotiate or Signature Ready. Closed opps aren't
open opps, so Closed Won and Closed Lost are never flagged.

**Tech Loss**: if the SA recorded Tech Loss, the deal should be Closed Lost
and the opp closed. Any still-open opp (at any stage, including Engage and
Discover) is ⚠️: the SA and the opp owner need to get on the same page, and
whichever side is stale updates. The alert says "SA recorded Tech Loss but
the opp is still open at {stage}". This is a separate case from the
Salesforce-ahead rule above.
Notion `Post-Sale Support`, `Re-engage Later`, or an empty value is
❓ can't compare. A Notion value at or above the minimum is ✅, including
when Notion is ahead of Salesforce (that isn't the problem this check looks
for).

## Edit here

Update the tables above if the org's stage names change or the mapping needs
correcting. The stage names are org-wide, so this file is shared across SAs.
