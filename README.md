# DOSS Lead Search V2

An **agent-orchestrated** pipeline that scans CPG industry publications for companies that fit
DOSS's Ideal Customer Profile (ICP), filters out any company that is **already in our Salesforce
instance**, enriches the net-new companies with **ZoomInfo** firmographics, and delivers a
**Google Sheet + email summary** of the results.

> Built for Max Jacobs (Account Executive, Team Michelle) and the DOSS sales org.

---

## What it produces

For every run, a list of **net-new, ICP-fit CPG companies not yet in Salesforce**, each with:

| Field | Source |
|-------|--------|
| Company name, website | Publication / web search → ZoomInfo |
| Why it surfaced (funding, launch, expansion + source link) | Publication / web search |
| Product type / category | ZoomInfo |
| Estimated revenue | ZoomInfo |
| Employee count | ZoomInfo |
| HQ location (city / state / country) | ZoomInfo |
| Founded year | ZoomInfo |
| Funding (total, last round, date, investors) | ZoomInfo scoops / funding |
| Manufacturing model & current systems (if known) | Publication / ZoomInfo / web |
| ICP fit score + tier + rationale | This pipeline (see `playbook/scoring.md`) |

Delivered into a **single master Google Sheet** (`DOSS Lead Search — Master Tracker`) in your
Drive, with **each run added as its own dated section** (newest on top) and **cross-run dedup** so
the same company isn't reported twice — plus a **short email** to `max@doss.com` with the new
highlights and a link.

> The Google Drive integration can't edit a Sheet in place, so each run rebuilds the master sheet
> from the append-only `runs/leads_master.csv`. The file URL changes on each rebuild — **bookmark
> the [DOSS Lead Search folder](https://drive.google.com/drive/folders/1C0jSPsEvAmBmVgdtALCNv22dkR0NxQCK)**, not the file.

## Scheduling (every 3 hours)
Runs are triggered from **Claude Code on the web** — create a scheduled trigger on this repo with
the prompt *"Run the DOSS lead search."* at your desired cadence (`config/settings.yaml →
schedule.cadence`, currently `every_3_hours`). The pipeline **no-ops** (no new section, no email)
when a run finds zero net-new companies, so frequent runs stay cheap and quiet. See
https://code.claude.com/docs/en/claude-code-on-the-web for trigger setup.

> Heads-up on cadence: trade publications and the ZoomInfo lookalike pool change slowly, so most
> 3-hourly runs will find nothing new (and still spend some ZoomInfo credits). Daily or weekly is
> usually plenty; 3-hourly is supported but largely redundant.

---

## Architecture: agent-orchestrated playbook

This project intentionally contains **no API keys and no standalone runtime**. Instead, an agent
(Claude, running in Claude Code on the web or desktop) executes the pipeline using the
integrations already connected to your workspace:

- **Salesforce (read-only MCP)** — dedup against existing accounts
- **ZoomInfo MCP** — firmographic enrichment
- **Web search / fetch** — scan publications for candidate companies
- **Gmail MCP + Google Drive MCP** — deliver the report

The repository is the **source of truth for *how* the pipeline runs**: the ICP definition, the
source list, the dedup rules, the scoring rubric, and the report format. To run it, you point an
agent at [`playbook/RUNBOOK.md`](playbook/RUNBOOK.md).

### How to run a search

In a Claude Code session connected to this repo (with Salesforce, ZoomInfo, Gmail, and Drive
MCP servers enabled), just say:

> **"Run the DOSS lead search."**

Claude reads [`CLAUDE.md`](CLAUDE.md) → follows [`playbook/RUNBOOK.md`](playbook/RUNBOOK.md)
step by step → produces the Google Sheet and email.

### How to schedule recurring runs

Claude Code on the web supports scheduled/triggered sessions. Create a trigger that opens a
session on this repo with the prompt *"Run the DOSS lead search and deliver the report."* A weekly
or bi-weekly cadence is recommended (see `config/settings.yaml`). See
https://code.claude.com/docs/en/claude-code-on-the-web for trigger setup.

---

## Repository layout

```
config/
  icp.yaml              # DOSS official ICP (revenue, size, geo, categories, disqualifiers)
  sources.yaml          # Publications + web searches + X to scan  <-- EDIT / EXTEND THIS
  focus.yaml            # Current "lookalike" campaign focus (anchors + profile)
  settings.yaml         # Delivery targets, run size, dedup keys
playbook/
  RUNBOOK.md            # The step-by-step pipeline the agent executes
  dedup.md              # Salesforce dedup rules + domain normalization + SOQL patterns
  scoring.md            # ICP fit scoring rubric
templates/
  report_sheet_schema.md    # Google Sheet column spec
  email_summary_template.md # Email subject + body template
docs/
  icp_source.md         # Transcription of the official "Best fit prospects" one-pager
  positioning.md        # DOSS positioning / PMF notes (for ICP screening judgment)
runs/
  README.md             # Where run logs are appended
```

---

## Configuration you control

- **`config/sources.yaml`** — the publications and web searches to scan. Seeded with recommended
  CPG sources; **replace/extend with DOSS's preferred publications and search terms.**
- **`config/icp.yaml`** — the ICP gate. Encoded from the official one-pager; adjust as the ICP evolves.
- **`config/settings.yaml`** — email recipient, Drive folder, target company count, dedup keys.
