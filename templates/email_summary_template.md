# Email Summary Template

Sent via the Gmail MCP to `settings.delivery.email.to` after the Sheet is created.

**Subject:** `{subject_prefix} {N} net-new ICP companies — {RUN_DATE}`

**Body (HTML or plain):**

```
Hi Max,

Here's this run's DOSS lead search — companies in our ICP that are NOT yet in Salesforce,
enriched with ZoomInfo.

  • Sources scanned: {sources_count} publications + {searches_count} web searches
  • Candidates found: {candidates_count}
  • Already in Salesforce (excluded): {known_count}
  • Net-new, ICP-fit companies reported: {reported_count}

Full results (Google Sheet): {sheet_url}

Top {top_n} by ICP fit:
  1. {company} — {category}, {revenue}, {hq}. {one_line_rationale}
  2. ...
  3. ...
  4. ...
  5. ...

Notes:
  - {any caveats — e.g. "3 companies had no ZoomInfo match; firmographics partial."}

Run details and the audit trail of excluded (already-known) companies are in runs/log.md.

— Automated by DOSS Lead Search V2
```

## Notes
- Populate every `{placeholder}`. If a value is unknown, write "unknown" rather than leaving it blank.
- Keep the top list to 5 unless the run is small.
- If configured for review instead of send, create this as a **draft** and tell Max it's waiting.
