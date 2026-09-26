# 0003 — AI suggests, rules decide
Status: ACCEPTED

## Decision
The LLM handles language: extraction, questions, candidate HS codes with reasoning, explanation.
Deterministic TypeScript handles all numbers, rates, dates, eligibility conditions and policy status.
LLM output never becomes a user-facing number or authoritative decision.

## Why
Compliance errors are costly; calculations must be exact, reproducible, and testable.
LLMs are strong at understanding messy product descriptions and weak at guaranteed correctness.

## Consequences
- HS candidates restricted to codes retrieved from our curated table.
- Report text generated AFTER calculation; numbers injected, not written by the model.
- Every AI call logged with prompt version for evaluation.
