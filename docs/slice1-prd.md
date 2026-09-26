# Slice 1 PRD — Compliance Lookup + Calculator
_Status: DRAFT — scope values marked [D1]/[D2]/[D3] wait on founder decisions in state.md_

---

## 1. Problem
A small Indian importer considering a new product can't easily answer:
- What is the HS / ITC-HS code?
- Is it free to import, restricted, or prohibited? What licences/registrations/certifications apply?
- What duty and tax will I pay, and what's my landed cost?
- Does the origin country qualify for a lower FTA rate — and what proof is needed?

Today they ask a CHA, search scattered government PDFs, or guess.
Guessing wrong means detained shipments, penalties, or a bad sourcing decision.

## 2. Scope (deliberately narrow)
| Dimension | Slice 1 | Later |
|---|---|---|
| Direction | Import into India only [D3] | Export |
| Categories | 1–2 categories [D1] | More |
| Origin countries | 3–5 [D2] | More |
| Transaction type | Standard commercial import (B2B, home consumption) | Samples, SEZ, EOU, re-import, gifts, courier |
| Output | On-screen structured report + PDF export | Slice 2 checklists/workflow |

Out-of-scope inputs get a clear "not covered yet" message — never a guessed answer.

## 3. Target user (hypothesis — to validate)
Primary: small/medium Indian importer or trading business, 1–20 imports/year,
no in-house compliance person, relies on a CHA.
Secondary: first-time importers (e-commerce sellers, D2C brands sourcing abroad).
Possible buyer instead: CHAs/customs brokers wanting faster pre-shipment answers. Validate.

## 4. Core user journey
1. User types a plain description ("Bluetooth speaker with rechargeable battery from China").
2. System extracts attributes and asks ONLY the questions needed (progressive, 3–8 questions):
   material, function, power source, battery, wireless, age group, origin, value, quantity.
3. System shows candidate HS codes with reasoning; user confirms one (or flags "unsure").
4. System runs deterministic checks for the confirmed code + origin + date:
   import policy, regulatory requirements, duty/tax calculation, FTA options.
5. User sees the structured report (§6), can export PDF, and is shown next actions.
6. Analysis is saved with a full snapshot (inputs, rules, sources, date, model version).

## 5. Features (P0 unless marked)

### F1. Product intake + clarifying questions
- Free-text input → AI extracts attributes into a typed schema.
- Question set driven by category-specific attribute requirements (stored in DB, not prompts).
- AC: never more than 8 questions; every question explains why it's asked;
  user can skip with "don't know" (lowers confidence, recorded as missing info).

### F2. HS classification assistance
- AI proposes up to 3 candidate codes from the curated in-scope code set only
  (retrieval over tariff headings + notes + advance rulings, then LLM ranks/explains).
- Each candidate: code, description, reasoning, relevant notes, what would change the answer.
- AC: codes outside the curated set can't be returned; output labelled as a suggestion;
  user must confirm; low-confidence → "human review recommended" flag.

### F3. Import policy + regulatory check (deterministic)
- For confirmed code + origin + as_of_date: import policy status (free/restricted/prohibited)
  and applicable requirements (e.g. certification, registration, labelling) from rules tables.
- AC: every requirement shows authority, source, effective date, verification date;
  if no rule data exists for the code → "not verified — consult CHA", never "none required".

### F4. Duty + tax calculator (deterministic)
- Inputs: confirmed code, origin, transaction value, currency, freight, insurance, quantity, date.
- Output: step-by-step table: INPUTS → RULES APPLIED → CALCULATION → OUTPUT.
- AC: every line shows rate, base, formula, source; exchange rate source and date shown;
  results match golden test cases exactly.

### F5. Origin / FTA analysis
- Show agreements that could apply for the origin country, standard vs preferential rate,
  duty difference, rule-of-origin summary, required proof of origin.
- AC: shown as "potentially eligible IF …" with conditions; never auto-applies the preferential
  rate as the headline number.

### F6. Report + export
- Section order per §6. PDF export with disclaimer and as_of_date. (P0)
- Saved analyses list per user. (P1)

### F7. Accounts (P1 — can be deferred until first pilot users)
- Email login, single-user orgs. No teams/roles in Slice 1.

## 6. Result format
SUMMARY · CLASSIFICATION · DUTIES & TAXES · ELIGIBILITY · REGULATORY REQUIREMENTS ·
ORIGIN/FTA · DOCUMENTS (list only — generation is Slice 2) · RISKS · MISSING INFORMATION ·
SOURCES · CONFIDENCE (High/Medium/Low + why) · NEXT ACTION

Every claim is tagged: VERIFIED / ASSUMPTION / INFERENCE / REQUIRES VERIFICATION.
Footer disclaimer: decision support, not legal/customs advice; confirm with a licensed CHA.

## 7. Data model (minimum)

### Reference & provenance
- `sources` — id, authority, title, document_ref (e.g. notification no.), url, published_on,
  effective_from, retrieved_on, verified_on, verified_by, file_hash, notes
- `countries` — iso2, name
- `authorities` — id, name, short_code

### Classification
- `hs_codes` — code, level (chapter/heading/subheading/tariff item), description, parent_code,
  unit, effective_from, effective_to, source_id
- `classification_notes` — hs_code_prefix, note_type (section/chapter/explanatory/ruling),
  text, source_id  ← embedded for retrieval
- `category_attributes` — category, attribute_key, question_text, why_text, required, order

### Temporal rules (never updated in place — versioned)
- `duty_components` — id, key (e.g. BCD / surcharge / IGST / cess), name, calc_order,
  base_expression (which prior components form the base), description, source_id
- `tariff_rates` — hs_code, component_key, origin_country (null = any), agreement_id (null = standard),
  rate_type (ad_valorem / specific / mixed), rate_value, specific_amount, specific_unit,
  conditions_json, effective_from, effective_to, source_id, supersedes_id
- `policy_rules` — hs_code_prefix, direction, status (free/restricted/prohibited/STE),
  conditions_json, effective_from, effective_to, source_id
- `regulatory_requirements` — hs_code_prefix, authority_id, requirement_type
  (licence/registration/certification/labelling/NOC), description, conditions_json,
  effective_from, effective_to, source_id
- `trade_agreements` — id, name, partner_countries, effective_from, source_id
- `origin_rules` — agreement_id, hs_code_prefix, criterion_summary, proof_required,
  effective_from, effective_to, source_id
- `exchange_rates` — currency, rate, rate_type, effective_from, effective_to, source_id

### Users & analyses
- `orgs`, `users` (Supabase Auth), `org_members`
- `analyses` — id, org_id, created_by, as_of_date, input_json, attributes_json,
  confirmed_hs_code, result_json, rule_ids_applied[], source_ids[], model_version,
  confidence, created_at   ← immutable snapshot
- `audit_log` — who, what, when, entity, before/after

All org-scoped tables: RLS by org_id.

## 8. Rules engine (lib/rules/)
Pure functions, no I/O inside the calculators — data is loaded first, then passed in.

- `resolveRules(hsCode, origin, direction, asOfDate)` → applicable rows from each rules table
  where effective_from <= asOfDate < effective_to (or effective_to null).
- `computeAssessableValue(inputs, rules)` — valuation method REQUIRES VERIFICATION against the
  Customs Valuation Rules before implementation; record the source in docs/sources.md.
- `computeDuties(assessableValue, components, rates)` — iterate components in calc_order;
  each component's base is defined by data (base_expression), not hardcoded.
  The commonly described Indian import chain (basic duty → surcharge on duty →
  IGST on value + duties → compensation cess where applicable) is REQUIRES VERIFICATION
  and must be encoded as data in duty_components, with sources.
- `evaluateConditions(conditions_json, attributes)` — small, explicit condition language
  (eq, in, lt, gt, exists, and/or). No eval().
- Output: a `CalculationTrace` — every step with inputs, rule id, source id, formula, result.

## 9. AI layer (lib/ai/)
| Task | Approach | Guardrail |
|---|---|---|
| Attribute extraction | Claude with JSON schema output | Validate with Zod; reject unknown keys |
| Clarifying questions | Pick from category_attributes; AI only phrases them | Max 8 |
| HS candidates | Retrieve from hs_codes + classification_notes (pgvector) → Claude ranks & explains | Only codes from retrieved set; confidence + reasoning required |
| Report explanation | Claude writes summary from the deterministic result | Cannot change numbers; numbers injected, not generated |

Log every AI call: prompt version, model, inputs hash, output, latency, cost.
Prompts live in `lib/ai/prompts/` with version numbers.

## 10. API (Next.js route handlers)
- `POST /api/analyses` — create from description → returns attributes + questions
- `POST /api/analyses/:id/answers` — submit answers → returns HS candidates
- `POST /api/analyses/:id/confirm-code` — confirm HS → runs rules → returns full result
- `GET  /api/analyses/:id` — result + trace
- `GET  /api/analyses/:id/pdf` — export
- `GET  /api/analyses` — list (P1)
Admin (founder only): `POST /api/admin/sources`, `POST /api/admin/rules/import` (CSV with source_id).

## 11. UI screens
1. Landing / start: single input box + example prompts
2. Questions: one card at a time, progress indicator, "why we ask", skip
3. Classification: candidate cards, reasoning, confirm / "not sure"
4. Result: summary on top, collapsible sections per §6, calculation table, sources panel
5. Saved analyses (P1)
6. Admin: sources + rules import with preview and diff (founder only)

Mobile-friendly. Plain language first, technical detail expandable.

## 12. Security
Supabase Auth; RLS on all org data; service role only in server code; secrets in Vercel env;
rate limiting on AI endpoints; uploaded files (later) in private buckets with signed URLs;
audit_log for admin data changes; daily backups (Supabase).

## 13. Testing & evaluation
- Unit: every rules-engine function, including date boundaries (day before/after effective dates)
  and missing-data paths.
- Golden cases (`tests/golden/`): the manual reports from validation + published advance rulings
  for the chosen categories. Each case: inputs, expected code, expected duty lines, sources.
- Classification eval: top-1 and top-3 accuracy on golden cases; track per prompt version.
- Launch bar (proposal): 100% of golden duty calcs exact; HS top-3 ≥ 90% on in-scope cases;
  zero outputs containing codes/rates without a source.
- E2E (Playwright): full journey on 3 representative products.

## 14. Build phases
0. Validation + manual reports + source verification (no code)
1. Scaffold: Next.js + Supabase + auth + CI + tests running
2. Schema + migrations + admin import for sources and rules
3. Rules engine + calculation trace + golden tests
4. Load verified data for in-scope categories/countries
5. AI: attribute extraction + questions
6. AI: HS candidate retrieval + ranking + eval harness
7. Result page + PDF
8. Pilot with 5–10 users from validation interviews

## 15. Slice 1 is DONE when
- In-scope products produce complete reports with sources for every claim
- Launch bar in §13 met
- 5+ pilot users have each run 3+ real analyses, and we know whether they'd pay
