# CLAUDE.md — How to operate this repo

This repository is an **agent-orchestrated playbook**, not a standalone app. Its job: find CPG
companies that fit DOSS's ICP, exclude any already in Salesforce, enrich the rest with ZoomInfo,
and deliver a Google Sheet + email report.

## To run a lead search
When asked to "run the DOSS lead search" (or similar), **follow
[`playbook/RUNBOOK.md`](playbook/RUNBOOK.md) step by step.** Read the three config files first:
- `config/icp.yaml` — the ICP gate (from the official one-pager; see `docs/icp_source.md`)
- `config/sources.yaml` — publications + web searches to scan **(Max owns this list)**
- `config/settings.yaml` — delivery targets, run size, dedup keys

Supporting specs: `playbook/dedup.md` (Salesforce dedup + domain normalization),
`playbook/scoring.md` (ICP fit rubric), `templates/` (Sheet schema + email body).

## Required MCP integrations
Salesforce (read-only), ZoomInfo, Web search/fetch, Gmail, Google Drive.

## Hard rules
- **Salesforce is read-only** — only read to dedup; never attempt writes.
- **ZoomInfo:** individual B2B research only; enrich only the net-new companies needed. No mass export.
- Quality over quantity; mark unknowns as "unknown" instead of guessing.
- Log every run in `runs/log.md` with the audit trail of excluded (already-known) companies.

## Editing the pipeline
Behavior is configuration, not code — adjust the YAML in `config/` and the specs in `playbook/`.
Keep `docs/icp_source.md` in sync with `config/icp.yaml` if the ICP changes.
