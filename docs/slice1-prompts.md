# Slice 1 — Claude Code Prompt Pack

How to use:
- Run the prompts in order. One prompt per session.
- Start each session fresh (`/clear`), paste the prompt, approve the plan, and let it build.
- End each session with `/wrap`.
- Don't move to the next prompt until the current phase's "Done when" is true.
- Anything in [brackets] is for you to fill in.

---

## P0 — Kickoff (decisions, no code)

```
Read CLAUDE.md, docs/state.md, docs/vision.md, docs/slice1-prd.md, docs/sources.md,
docs/validation-plan.md and docs/decisions/*. Don't write any app code.

1. In 5–8 lines, confirm what Slice 1 is, what's out of scope, and the non-negotiables.
2. Walk me through decisions D1–D5 in docs/state.md one at a time. Recommend, explain briefly,
   wait for my answer, and push back if a choice widens scope.
3. Once decided: update state.md, replace the [D1]/[D2]/[D3] placeholders in the PRD, set ADR 0001
   to ACCEPTED (or revise it), and trim docs/sources.md to only what my categories and countries need.
4. Create these templates:
   - docs/report-template.md: the manual compliance report in PRD §6 format, with
     VERIFIED/ASSUMPTION/INFERENCE/REQUIRES VERIFICATION tags and a sources table
   - tests/golden/_template.md: golden case format (inputs, expected HS code, expected policy
     status, expected requirements, expected duty lines with base/rate/amount, as_of_date,
     source ids, who verified it)
5. Tell me exactly what I must do manually before the next session.
```
Done when: D1–D5 are decided, the templates exist, and you have started interviews.

---

## P0.5 — Between sessions (you, not Claude Code)

Do the interviews. Write 5+ manual reports using docs/report-template.md (use the claude.ai
Project with web search for research). Verify every source you used in docs/sources.md.
Save each report as a golden case in tests/golden/.

---

## P1 — Scaffold

```
/feature Phase 1 scaffold

Scope: Next.js (App Router) + TypeScript strict, Tailwind, Supabase (local dev via CLI + auth),
Vitest, Playwright, ESLint, a typecheck script, and GitHub Actions CI running lint + typecheck +
unit tests. Add .env.example with every variable documented, and no secrets committed.

Folder structure:
app/ (routes + api), lib/rules/ (pure calculation code, no I/O), lib/data/ (DB access),
lib/ai/ (+ prompts/ with versioned prompt files), lib/validation/ (Zod schemas),
supabase/migrations/, tests/unit/, tests/golden/, tests/e2e/.

Add one trivial passing test in each test type to prove the pipeline works.
No product features. Afterwards, fill in the Commands section of CLAUDE.md.
```
Done when: `npm run dev`, tests, lint, typecheck and CI all pass on a fresh clone.

---

## P2 — Database schema + provenance

```
/feature Phase 2 schema

Implement the minimum schema in PRD §7 as Supabase migrations: sources, countries, authorities,
hs_codes, classification_notes, category_attributes, duty_components, tariff_rates, policy_rules,
regulatory_requirements, trade_agreements, origin_rules, exchange_rates, orgs, org_members,
analyses, audit_log.

Requirements:
- Every regulatory table has source_id (NOT NULL, FK), effective_from (NOT NULL), and nullable
  effective_to; versioned tables also have supersedes_id.
- A check constraint ensures effective_to > effective_from.
- Prevent overlapping active versions for the same key (exclusion constraint or trigger;
  explain which you chose).
- Regulatory tables are append-only: block UPDATE of rate/condition columns with a trigger.
  The only allowed update is setting effective_to.
- The analyses table is immutable after creation.
- RLS is enabled on all org-scoped tables, with policies by org_id. Regulatory tables are readable
  by authenticated users and writable only by the service role.
- Enable pgvector on classification_notes (the embedding column can stay empty for now).
- Generate TypeScript types from the schema.

Tests: migration applies cleanly; an overlapping version is rejected; an in-place rate update is
rejected; RLS blocks cross-org reads (test with two users).
Use only obviously fake fixture data (HS 9999.99.99, rate 12.345%).
```
Done when: all constraint and RLS tests pass.

---

## P3 — Rules engine (the core — take your time here)

```
/feature Phase 3 rules engine

Build lib/rules/ as pure TypeScript functions per PRD §8:
- resolveRules(input, ruleSet, asOfDate): selects rules valid on asOfDate, where effective_from
  <= date < effective_to or effective_to is null; the most specific match wins (full code over
  prefix, origin-specific over "any"). Document the precedence order in code comments and an ADR.
- evaluateConditions(conditions, attributes): a small JSON condition language
  (eq, neq, in, lt, lte, gt, gte, exists, and, or, not). No eval(). Unknown operators throw.
  Missing attributes return "UNKNOWN", not false.
- computeDuties(assessableValue, components, rates): iterates components by calc_order; each
  component's base comes from its base_expression data, never hardcoded. Supports ad valorem,
  specific (per unit) and mixed rates. Uses decimal-safe arithmetic (a decimal library, not JS
  floats). Rounding rules are configurable data and marked REQUIRES VERIFICATION.
- computeAssessableValue: stub that throws "valuation method not verified" until the
  S12 source in docs/sources.md is VERIFIED.
- Every function returns a CalculationTrace: steps with inputs, rule id, source id, formula
  string, and result. The final output must be fully reconstructable from the trace.

Separately, lib/data/loadRuleSet.ts fetches everything for (hsCode, origin, direction,
asOfDate) in as few queries as possible, then hands the data to the pure functions.

Tests (fake data only): date boundaries (the day before, on, and after effective dates);
precedence; UNKNOWN propagation; specific and mixed rates; currency conversion using a dated
exchange rate; a missing rate producing "REQUIRES VERIFICATION" instead of zero; and a trace
that sums exactly to the total. Aim for 100% branch coverage on lib/rules/.
```
Done when: coverage is 100% on lib/rules/ and I've reviewed the trace output by hand.

---

## P4 — Verified data loading (you supply data, Claude builds the pipe)

```
/feature Phase 4 data import

Build an admin-only import flow for regulatory data:
- CSV templates in data/templates/ for each regulatory table, with a README explaining each
  column.
- A CLI script (npm run import:rules -- <file> --dry-run) and an admin page (founder-only,
  checked server-side) that:
  1. validates every row with Zod
  2. rejects any row whose source_id doesn't exist or whose source isn't marked verified
  3. shows a diff: new rows, rows being closed (effective_to set), and conflicts
  4. applies only after confirmation, inside a single transaction
  5. writes to audit_log
- A script that imports tests/golden/*.md into structured test fixtures.

IMPORTANT: Do not fill any CSV with real values. I will fill them from verified sources.
Put only fake example rows in the templates.
```
Then you: fill the CSVs for your in-scope categories and countries from verified sources and import them.

```
Run every golden case in tests/golden/ through the rules engine against the imported data.
Report each mismatch with the trace, and say whether you think the error is in the data, the
engine, or the golden case. Don't change data or golden cases yourself. Ask me.
```
Done when: 100% of golden duty calculations match exactly.

---

## P5 — Intake + clarifying questions (first AI feature)

```
/feature F1 product intake

- POST /api/analyses: takes a free-text description plus origin and value, and returns
  extracted attributes and the next questions.
- lib/ai/extractAttributes: calls the Claude API server-side with structured JSON output.
  Validate with a Zod schema per category (built from category_attributes). Drop unknown keys.
  Low-confidence fields count as missing.
- Questions come from category_attributes (DB), not the model. The model may only rephrase a
  question for clarity. Ask a maximum of 8 questions, ordered by importance. Each question
  shows why_text.
- POST /api/analyses/:id/answers: supports "don't know", which is recorded as missing info.
- If the description is out of scope (another category, export, or non-standard transaction),
  return a clear "not covered yet" message. Never guess.
- Log every AI call to a table: prompt version, model, input hash, output, tokens, latency.
- UI: a start screen with one input box and example prompts, then one question card at a
  time with a progress bar.

Tests: mock the Claude API in unit tests. Add a small eval script (npm run eval:extract) that
runs the golden-case descriptions against the real API and reports attribute accuracy.
```
Done when: extraction accuracy on golden descriptions is good enough that you'd trust it, the flow works end to end, and out-of-scope inputs are refused cleanly.

---

## P6 — HS classification assistance

```
/feature F2 HS candidates

- Embed hs_codes descriptions and classification_notes (the in-scope set only) into pgvector.
  Add a script to re-embed when data changes.
- lib/ai/classify: retrieve the top N headings and notes for the confirmed attributes, then
  have Claude rank up to 3 candidates. For each: code, reasoning, relevant notes cited by id,
  and "what would change this answer". Output is JSON validated by Zod.
- Hard rule: any code not in the retrieved set is discarded (test this with a mocked model
  that returns an invented code).
- Confidence is computed deterministically from signals (retrieval score gap, missing required
  attributes, model agreement across 2 runs), not self-reported by the model. Low confidence
  sets human_review_recommended.
- POST /api/analyses/:id/confirm-code: the user confirms a code or picks "not sure".
- UI: candidate cards with reasoning and sources, confirm button, and a "not sure" path that
  recommends CHA review.
- npm run eval:classify reports top-1 and top-3 accuracy on golden cases, per prompt version,
  and saves the results to evals/results/.
```
Done when: top-3 accuracy is at least 90% on in-scope golden cases, and invented codes are always discarded.

---

## P7 — Full result + report + PDF

```
/feature F3 F4 F5 F6 result report

After code confirmation:
- Run loadRuleSet plus the rules engine to get policy status, regulatory requirements, the duty
  calculation (with trace), and FTA options for the origin. Show FTA as "potentially eligible
  IF…" with conditions and proof required. Never make the preferential rate the headline.
- Save an immutable analyses row: inputs, attributes, confirmed code, result_json, rule ids,
  source ids, as_of_date, and model and prompt versions.
- lib/ai/explain writes the SUMMARY and NEXT ACTION text from the finished result. Numbers are
  injected from the result and never generated. Add a test that fails if any number in the
  generated text isn't in the result.
- Result page in PRD §6 order: summary at the top, then collapsible sections, the calculation
  table (inputs → rule → formula → amount), a sources panel (authority, reference, effective
  date, verified date), a VERIFIED/ASSUMPTION/INFERENCE/REQUIRES VERIFICATION tag on every
  claim, and a confidence rating with its reasons.
- Missing rule data shows "not verified — consult a licensed CHA", never "none required".
- GET /api/analyses/:id/pdf: the same content, with the disclaimer and as_of_date on every page.
- Playwright e2e: 3 full journeys from the golden cases.
```
Done when: the 3 e2e journeys pass, and a golden case's PDF matches your manual report.

---

## P8 — Pilot readiness

```
/feature Phase 8 pilot readiness

Prepare the app for 5–10 pilot users:
- Supabase email login with single-user orgs; saved analyses list (F7, P1).
- Rate limiting on the AI endpoints; per-user daily analysis cap (configurable).
- Security pass: RLS review, check that the service-role key is server-only, input validation on
  every route, error handling that never leaks stack traces, and review of dependencies.
- Product analytics events (analysis_started, question_answered, code_confirmed, report_viewed,
  pdf_downloaded, analysis_abandoned + step), stored in our own table. No third-party tracker yet.
- Feedback: a "Was this correct?" control on each report section, and a "report an error" field.
- A staleness banner if any source used hasn't been verified within [N] days.
- Deploy to Vercel with a production Supabase project. Write docs/runbook.md covering deploy,
  rollback, backup/restore, and the process for updating rules after a notification or Budget.

Finish with a pre-launch checklist in docs/state.md listing what I must verify manually.
```
Done when: 5+ pilot users are running real analyses and you're reading their feedback weekly.

---

## Use anytime

- `/next` — when you're unsure what to do
- `/triage [idea]` — before adding anything not in the PRD
- `/wrap` — at the end of every session

After a Budget or a new notification:
```
A rule changed: [authority, notification ref, effective date, what changed]. I've verified it
and added it to docs/sources.md as [source id]. Prepare the import CSV that closes the old rows
and opens the new ones, run it as a dry run, show me the diff, and list every golden case whose
expected result changes.
```

When something looks wrong in a result:
```
Analysis [id] looks wrong: [what]. Using its stored snapshot and trace, find where the result
came from (data, rule selection, calculation, or AI) and explain before changing anything.
```
