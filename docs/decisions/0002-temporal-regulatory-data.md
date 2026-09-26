# 0002 — Regulatory data is temporal and versioned
Status: ACCEPTED

## Decision
Every regulatory row has effective_from, effective_to (nullable), source_id, and supersedes_id.
Rows are never updated in place. A change = close the old row + insert a new row.
Every rules query takes an explicit as_of_date.

## Why
Rates and requirements change (budgets, notifications, amendments). Users and auditors must be
able to ask "what applied on the date of this shipment?" and "why did the system say this?"

## Consequences
- Queries always filter by date range.
- Analyses store rule_ids_applied and source_ids — results remain reproducible.
- Admin import shows a diff: which rows close, which open.
- Tests cover date boundaries.
