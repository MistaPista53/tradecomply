# Phase 0 — Validation Plan (before product code)

## Goal
Find out, cheaply, whether importers have this pain often enough to pay — and build a
golden test set while doing it.

## Who to talk to (target 10–15 conversations)
- 6–8 small/medium importers in the chosen categories
- 2–3 CHAs / customs brokers
- 2–3 first-time importers (e-commerce / D2C sellers sourcing abroad)
Channels: personal network, trade associations, LinkedIn, import-focused WhatsApp/Telegram
groups, IndiaMART/TradeIndia buyer listings, CHA offices near ports/ICDs.

## Interview script (20 min — ask about the past, don't pitch)
1. Tell me about the last time you imported a new product. What happened step by step?
2. How did you find the HS code, duty and required approvals?
3. What went wrong or took longest? What did it cost you (money, days, stock)?
4. Who did you rely on? What do you pay them? Were you satisfied?
5. How many new products / new origins do you evaluate per year?
6. Have you ever had a shipment held, re-classified or penalised? What happened?
7. Before placing an order, what would you want to know that you didn't?
8. If a tool gave you a sourced report in 10 minutes, what would it need to show for you to trust it?
9. Would you pay for that? Per report or monthly? (Then show a price and watch the reaction.)
10. Can I prepare one report for a product you're considering now? (← the real test)

## Concierge MVP
For every "yes" to Q10: produce the report by hand (with Claude + official sources), in the
format of PRD §6, within 48 hours. Record:
- time taken to produce · sources used · where you got stuck
- did they read it · did they act on it · did they ask for another · would they pay (and how much)

Each finished report → `tests/golden/<slug>.md` (inputs, expected code, duty lines, sources).

## Decision criteria after Phase 0
| Signal | Proceed | Rethink |
|---|---|---|
| Users with real, recent pain | ≥ 6 of 10 | < 4 |
| Asked for a second report | ≥ 3 | 0–1 |
| Willing to pay (stated price) | ≥ 3 | none |
| Buyer clarity | Clear who pays | Still unclear → more interviews |

## Log
| # | Date | Who (type) | Category | Key pain | Pays today | Report? | Paid/would pay |
|---|---|---|---|---|---|---|---|
| 1 | | | | | | | |
