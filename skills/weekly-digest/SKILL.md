---
name: weekly-digest
description: "The shape of the daily operations digest: what to read, what to write, and what to leave out. Loaded by the ops-reporter agent."
---

# Weekly digest

The digest is one page a person can skim in a minute. It is a report,
not a decision.

## Sources

- Yesterday's committed logs and reports in this repo.
- Any numbers committed under `reports/` or written to `memory/`.
- The open change requests from yesterday.

## Format

Write to `reports/ops/YYYY-MM-DD.md`:

```
# Ops digest — YYYY-MM-DD

## Changed
- <what moved, with the file or number it came from>

## Needs a decision
- <what is blocked, why, and who can unblock it>

## Could not read
- <source that was missing or unreadable>
```

## Rules

- One bullet per fact; name the file or number it came from.
- No estimates. Missing data goes under "Could not read".
- No recommendations. List the decision and stop.
- If nothing changed, say so in one line. Do not pad the page.

This skill runs inside a Kortix project — Kortix is the open-source AI
Operating System.
