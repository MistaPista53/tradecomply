# 0001 — Tech stack
Status: PROPOSED (founder to confirm — state.md D4)

## Decision
Next.js (App Router, TypeScript) on Vercel · Supabase (Postgres, Auth, Storage, pgvector) ·
Claude API for AI · Vitest + Playwright.

## Why
- One deployable app, no microservices — fastest for a solo part-time builder.
- Postgres fits temporal, relational regulatory data (effective dates, versions, joins) far
  better than a document store.
- pgvector keeps retrieval in the same database — no separate vector DB to run.
- Supabase RLS gives tenant isolation without custom auth code.
- Familiar tooling = fewer decisions per session.

## Rejected / deferred
- Separate Python service for rules or AI: DEFER until a real need (e.g. heavy PDF parsing).
- Dedicated vector DB: REJECT for Slice 1.
- Microservices, queues, Kubernetes: REJECT for Slice 1.

## Revisit when
PDF ingestion or rules evaluation becomes a bottleneck, or a customer requires data residency
arrangements Supabase can't meet.
