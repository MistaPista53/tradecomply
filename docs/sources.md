# Regulatory Sources Register

Rules:
- Nothing enters the rules tables without a row here (and a matching `sources` DB row).
- "Status" is UNVERIFIED until the founder has opened the official source, confirmed the
  content and date, and filled in "Verified on".
- Prefer official government/regulator publications. Secondary sources (blogs, CHA sites)
  may help you find things, but are never the cited source.
- Re-verify after every Union Budget and whenever a notification amends a rule.

## Source types needed for Slice 1
All entries below are starting points to research — none are verified yet.

| # | Need | Likely official source (to confirm) | Status | Verified on | Notes |
|---|---|---|---|---|---|
| S1 | Customs tariff (HS codes, standard BCD) | CBIC — Customs Tariff | UNVERIFIED | | Chapter + section notes also needed |
| S2 | Duty exemption / effective-rate notifications | CBIC — Customs notifications | UNVERIFIED | | Effective rate often differs from tariff rate |
| S3 | Surcharge / cess components | CBIC — notifications, Finance Act | UNVERIFIED | | Encode as duty_components data |
| S4 | IGST on imports + GST compensation cess | CBIC / GST notifications | UNVERIFIED | | |
| S5 | Import policy by ITC-HS code | DGFT — ITC(HS) import policy schedule | UNVERIFIED | | Free / restricted / prohibited + conditions |
| S6 | Product certification (e.g. quality control orders) | BIS — QCO list / registration schemes | UNVERIFIED | | Category-dependent |
| S7 | Wireless equipment approvals | DoT / WPC | UNVERIFIED | | Only if category includes wireless |
| S8 | Packaged-goods labelling for imports | Legal Metrology (Dept. of Consumer Affairs) | UNVERIFIED | | |
| S9 | FTA tariff concessions | CBIC — FTA notifications per agreement | UNVERIFIED | | Per origin country chosen in D2 |
| S10 | Rules of origin + proof of origin | CBIC / agreement texts (Dept. of Commerce) + origin rules under Customs Act | UNVERIFIED | | |
| S11 | Customs exchange rates | CBIC — exchange rate notifications | UNVERIFIED | | Periodic |
| S12 | Valuation method (assessable value) | Customs valuation rules | UNVERIFIED | | Needed before computeAssessableValue |
| S13 | Classification precedent (golden cases) | Customs advance rulings (published) | UNVERIFIED | | For eval set |
| S14 | Importer registration (IEC) | DGFT | UNVERIFIED | | Show as prerequisite |

Other regulators (FSSAI, CDSCO, plant/animal quarantine, etc.) only if the chosen categories need them.

## Verified source log
| Source id | Authority | Document ref | Published | Effective from | URL | Verified on | By |
|---|---|---|---|---|---|---|---|
| | | | | | | | |
