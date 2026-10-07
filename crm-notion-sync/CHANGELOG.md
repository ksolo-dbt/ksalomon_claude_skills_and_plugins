# Changelog

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
