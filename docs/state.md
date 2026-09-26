# Project State
_Last updated: 2026-09-26 — update at the end of every session_

## CURRENT OBJECTIVE
Phase 0: lock Slice 1 scope and validate the problem with real importers
BEFORE writing product code.

## DECISIONS NEEDED FROM FOUNDER (blocking)
- [ ] D1: Product categories for Slice 1 (recommended: 1–2 categories, e.g. consumer
      electronics and/or toys — confirm based on who you can interview)
- [ ] D2: Origin countries for Slice 1 (recommended: 3–5, starting with the most
      common sources for the chosen categories)
- [ ] D3: Direction — imports into India only (recommended) vs. exports too
- [ ] D4: Confirm tech stack in docs/decisions/0001-tech-stack.md
- [ ] D5: Realistic Slice 1 timeline given part-time capacity

## COMPLETED
- Project instructions written
- Repo docs scaffolded (CLAUDE.md, vision, PRD draft, state, decisions, sources, validation plan)

## IN PROGRESS
- (none)

## P0 — NEXT UP
1. Founder decides D1–D3
2. Run 10–15 validation interviews (docs/validation-plan.md)
3. Produce 5 manual compliance reports by hand → these become golden test cases
4. Verify and record sources for the chosen categories in docs/sources.md
5. Then: scaffold repo + database schema (PRD §7)

## BLOCKED
- All product code is blocked on D1–D3 and on sources being verified.

## BACKLOG (not now)
- Export direction
- Additional categories / countries
- Document upload + extraction
- Slice 2 checklist workflow
- Payments

## RISKS
- R1 Data: regulatory data mostly in PDFs/notifications; curation effort underestimated
- R2 Accuracy: wrong classification or rate → user loss → liability
- R3 Liability: product must be positioned as decision support, not legal advice
- R4 Demand: importers may already rely on CHAs and not pay for software
- R5 Capacity: part-time founder; scope must stay narrow
- R6 Staleness: rates/notifications change (e.g. Budget) — need an update process

## OPEN QUESTIONS
- Who pays: small importers, CHAs/brokers, or sourcing agents?
- What do they pay for today and how much?
- How often does a typical user face a new product/import decision?
- Would a licensed customs broker review outputs as a trust layer?
