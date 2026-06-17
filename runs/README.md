# Runs

Audit trail for each lead-search run. The pipeline appends to `log.md` at the end of every run
(RUNBOOK step 9): date, counts (scanned / net-new / reported), the Google Sheet URL, and the list
of companies excluded because they were already in Salesforce (name + matched Account Id + status).

Keeping this trail means we don't re-surface the same companies and can show our work.
