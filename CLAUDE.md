# CLAUDE.md — India Trade Compliance SaaS

## What this is
An India-focused import/export compliance tool. Slice 1 answers one question:
"What does it legally take to import this product into India, and what will it cost?"
It gives: candidate HS/ITC-HS codes, duties and taxes, origin/FTA position, and
licences/registrations. Every result must show its reasoning and cite its source.

Full vision: `docs/vision.md` · Current spec: `docs/slice1-prd.md`

## Start of every session
1. Read `docs/state.md` (current objective, in progress, blocked).
2. Read the PRD section relevant to the task.
3. If the task isn't in the PRD or state file, stop and ask before building.

## End of every session
Update `docs/state.md` (completed / in progress / blocked / new risks), then commit.

## Stack (see docs/decisions/0001-tech-stack.md)
- Next.js (App Router) + TypeScript (strict)
- Supabase Postgres + Supabase Auth + Row Level Security
- Vercel for hosting
- Claude API for the AI layer (server-side only)
- Vitest for unit tests, Playwright for end-to-end tests

## Commands
<!-- Fill in once the project is scaffolded -->
- Dev: `npm run dev`
- Test: `npm test`
- Typecheck: `npm run typecheck`
- Lint: `npm run lint`
- DB migrations: `supabase migration new <name>` / `supabase db push`

## NON-NEGOTIABLES (compliance product — accuracy beats confidence)
1. NEVER invent or hardcode HS codes, tariff rates, cess/surcharge rates, IGST rates,
   licence requirements, FTA eligibility, or notification numbers. Not in code,
   not in seed files, not in test fixtures presented as real data.
2. Every regulatory value comes from the database and carries a `source_id`.
   If a value has no verified source, the system says "requires verification".
3. Regulatory data is TEMPORAL. Rules have `effective_from` / `effective_to`.
   Never UPDATE a rule in place — close it (set effective_to) and INSERT a new version.
   Every query takes an `as_of_date`.
4. Calculations are deterministic TypeScript pure functions in `lib/rules/`,
   with unit tests. The LLM never produces a final number, rate, or code decision
   that the user sees as authoritative.
5. The LLM is used for: understanding product descriptions, extracting attributes,
   asking clarifying questions, suggesting candidate HS codes WITH reasoning,
   explaining results. Its outputs are always labelled as suggestions.
6. Every analysis result stores a snapshot of: inputs, rules applied (with versions),
   sources, as_of_date, and model version — so we can answer
   "why did the system say this, and what was true on that date?"
7. Never assume shipment country = country of origin.
8. Never assume an FTA benefit because an agreement exists.
9. Tenant isolation: every org-scoped table has RLS. No service-role key in client code.

## Data entry rule
Claude may write loaders, parsers, schemas, and admin tools for regulatory data.
Claude must NOT populate real regulatory tables from its own memory.
Real data enters only from sources listed in `docs/sources.md` and reviewed by the founder.
Test fixtures use obviously fake values (e.g. HS "9999.99.99", rate 12.345%).

## Scope discipline
Only build what is in `docs/slice1-prd.md` and marked P0 in `docs/state.md`.
If something seems useful but isn't there, add it to BACKLOG in state.md — don't build it.
Anything from Slice 3+ (marketplace, suppliers, finance, freight) is out of scope.

## Code conventions
- Small, reviewable changes. One feature per session.
- Write or update tests before/with implementation.
- No new dependency without a one-line reason in the PR/commit message.
- Architecture decisions go in `docs/decisions/NNNN-title.md`.
- Result labels use: VERIFIED / ASSUMPTION / INFERENCE / REQUIRES VERIFICATION.

## Working with me
I'm the founder and final decision-maker. Challenge bad ideas directly.
When a request is ambiguous, ask the minimum question needed. When you finish,
tell me what I need to do manually (keys, accounts, data review) and the next step.
