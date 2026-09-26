# Vision & Principles (reference — not the current build spec)

## Long-term vision
An operating layer for Indian businesses doing international trade: classification,
regulatory requirements, duties, origin/FTA, licences, documents, shipment prep,
customs brokers, verified suppliers, trade finance, logistics, AI assistance.

We do NOT build this all at once.

## Roadmap
| Slice | Window (revisit — see state.md) | What |
|---|---|---|
| 1 | now | Compliance lookup + calculator (classification, duty/tax, origin/FTA, licences) |
| 2 | after Slice 1 validated | First-shipment workflow: checklists, documents, readiness status |
| 3 | later | Customs broker / service-provider connections |
| 4 | later | Verified supplier network |
| Year 2 | later | Trade finance, freight, insurance, logistics |

Slices 3+ are strategic context only. Do not build toward them.

## Operating principles
- BUILD LESS. LEARN FASTER. Manual MVP → validation → automation.
- Every feature has a reason. Every regulatory claim has evidence.
  Every calculation is explainable. Every architecture decision is written down.
- Feature triage: BUILD NOW / VALIDATE FIRST / DEFER / REJECT.
  Ask: "What is the smallest version that gives real user value and teaches us something?"
- Priorities: P0 must-have for current MVP · P1 high value / validates a key assumption ·
  P2 later · P3 distraction.

## Development loop (per substantial feature)
Problem → user → workflow → requirements → acceptance criteria → data → business logic →
regulatory dependencies → architecture → implement → test → review → document.

## Regulatory data model principle
Not "HS X = duty Y", but:
RULE → jurisdiction → HS code → origin → destination → transaction type →
effective_from → effective_to → rate → conditions → exceptions → source → source_version.
The system must answer: "What rule applied to this transaction on this date?"

## AI vs deterministic
AI: language understanding, attribute extraction, clarifying questions,
classification assistance, explanation, summaries, missing-info detection.
Deterministic: calculations, tariff application, date validity, thresholds,
country relationships, rule evaluation, known restrictions, eligibility conditions.

## Result format (every analysis)
Summary · Classification · Duties & taxes · Eligibility · Regulatory requirements ·
Origin/FTA · Documents · Risks · Missing information · Sources · Confidence · Next action

## Success sequence
Idea → validated problem → MVP → first users → repeat usage → first revenue → PMF → scale.
