---
name: self-storage-comp-rates
description: >
  Competitive rate intelligence for self-storage owner-operators. Trigger whenever the user
  asks about competitor pricing, comp analysis, market rates, street rates, rate benchmarking,
  or week-over-week rate changes. Also trigger for: "what are my comps charging," "pull comp
  rates," "rate survey," "competitive set," "run my weekly rate check," "did comps move rates,"
  "rate tracker," or any request to research storage pricing near a facility or market. Handles
  the full workflow: identifying competitors within a radius, fetching live rates from
  StorageCafe, SpareFoot, RentCafe, and facility websites; producing an apples-to-apples matrix
  by unit category (CC, Conventional, Parking, Other) and size (5x5–10x30); detecting
  week-over-week changes; and generating Match / Hold / Beat action flags. Use proactively
  whenever the user says "run my comps" or mentions tracking rates over time. On first run,
  establish the user's organization profile, target markets, saved facilities, and desired
  competitor-set rules before doing market research.
---

# Self-Storage Comp Rate Tracker

Produces a competitive rate matrix for a self-storage facility with **week-over-week change
detection** and **action-item flags**. Designed to run weekly to track how the competitive
set moves pricing and promos over time. The skill is company- and market-agnostic: do not
assume a default operator, state, city, submarket, or saved comp list.

See `references/unit-taxonomy.md` for the canonical unit type/size standardization rules.
See `references/storage-history.md` for guidance on persisting rate history across runs.
See `references/action-flags.md` for the logic governing Match / Hold / Beat recommendations.

---

## First-Run Market Setup

Before running a comp survey, determine whether a reusable **Market Profile** exists in the
conversation, user-provided files, prior run JSON, or reference material. If no profile
exists, pause and ask the user to establish it. Do not assume a company, market, state,
portfolio, or competitor list.

Ask for the minimum setup needed to make future runs repeatable:

1. **Organization label**: company name, portfolio name, or neutral label for reports.
2. **Saved facilities**: facility names, addresses, and optional internal IDs.
3. **Markets and submarkets**: cities, MSAs, counties, trade areas, corridors, or named
   operating clusters.
4. **Default radius rules**: default radius plus any urban, suburban, rural, or dense-market
   overrides.
5. **Competitor rules**: include/exclude aggregators, REITs, regional operators,
   independents, managed stores, parking-only sites, portable storage, or new developments.
6. **Subject rate source**: user-provided rate sheet, website, PMS export, or "leave subject
   rates blank until provided."
7. **History preference**: whether the user will paste prior-run JSON or maintain a saved
   baseline file.

After setup, restate the reusable profile as JSON so it can be pasted into future runs:

```json
{
  "organization_label": "",
  "facilities": [
    {
      "id": "",
      "name": "",
      "address": "",
      "market": "",
      "submarket": "",
      "notes": ""
    }
  ],
  "market_rules": {
    "default_radius_miles": 5,
    "urban_radius_miles": 3,
    "rural_radius_miles": 8,
    "competitor_inclusion_rules": [],
    "competitor_exclusion_rules": []
  },
  "subject_rate_source": "",
  "history_preference": ""
}
```

If the user says to proceed without setup, use a neutral one-facility run: identify comps
fresh from the provided facility address, default to a 5-mile radius, and state that no
saved market profile is available.

---

## Workflow Overview

```
1. Confirm profile/inputs → 2. Load prior run (if exists) → 3. Identify comp set →
4. Fetch live rates → 5. Standardize into matrix → 6. Diff vs. prior run →
7. Output: matrix + change log + action items → 8. Save this run as new baseline
```

---

## Step 1 — Clarify Inputs

Confirm before proceeding:
- **Subject facility name + address** (required; needs to be geocodable)
- **Market Profile** — use saved profile if available; otherwise run First-Run Market Setup
- **Radius** — default from Market Profile, or 5 miles if none; adjust for market density
- **Saved comp set?** — ask "Is there a saved competitor list for this facility, or should I
  identify them fresh?" If the user has a prior run saved, ask for it or look for it in the
  conversation context (it will be in the standard JSON format, see `references/storage-history.md`).
- **Subject rates** — ask whether the user's rates are supplied, should be fetched from a
  public source, or should be left blank for the user to fill in.
- **Proceed with defaults?** — if the user says "run my weekly comps for [facility]", skip
  the interrogation and proceed with Market Profile defaults + prior data if provided.

---

## Step 2 — Load Prior Run Data

If the user provides or pastes a prior run JSON (see `references/storage-history.md` for schema),
parse it and hold it in context. The prior run contains:
- Run date
- Competitor list with their then-current rates
- Subject property rates at time of prior run

If no prior run exists, this is a **first run** — proceed normally; note that WoW comparison
will not be available and flag that.

---

## Step 3 — Identify Competitors

Use `web_search` with queries like:
- `self storage near [address/cross streets] [city] [state/region]`
- `self storage [city] [state/region] units prices`
- `site:storagecafe.com self storage [city] [state/region]`
- `site:sparefoot.com self storage [city] [state/region]`

Target **4–8 direct competitors** within the radius. Prioritize:
1. Proximity — closest first
2. Same trade area / overlapping customer draw
3. Fit with Market Profile inclusion/exclusion rules
4. Flag operator type: Public operator / REIT, regional operator, independent, managed
   facility, parking-specialist, or other

If a saved comp set exists from a prior run, **use that list as the starting point** and only
add/remove competitors if there's a clear reason (new facility opened, closure, etc.).

If user-provided submarket reference lists exist, read them when the subject matches one of
the configured markets to save research time. Treat bundled reference examples as optional
starting material only when the user's Market Profile explicitly matches them; otherwise
identify the comp set fresh.

---

## Step 4 — Fetch Live Rates

For each competitor, retrieve current street rates using this priority order:

1. **StorageCafe facility page** — most reliable structured rate data
   - URL pattern varies by country, state, province, and city; use facility-specific pages
   - Use `web_fetch` on the facility-specific page (not city index) for full unit list

2. **SpareFoot** — good for promotional/web rates and availability signals
   - Search: `sparefoot.com [city] [state/region] self storage [facility name]`

3. **Facility's own website** — for operators that don't list on aggregators

4. **RentCafe** — secondary cross-check

**Rate types to capture:**
- Street rate (standard monthly rate as listed)
- Web/promo rate (first-month discounts, % off, $1 first month) — capture separately
- If only a range is listed, use the low end for the smallest unit of that size
- Also note: availability signals ("X left," waitlist language, heavy promo = soft occupancy)

**Do not fabricate rates.** If a rate for a size/type is not found, use `—`.

---

## Step 5 — Standardize Rates Into Matrix

**Read `references/unit-taxonomy.md` before building the matrix.**

The unit taxonomy defines:
- How to classify any unit type a competitor lists into one of the 4 standard categories
- Which sizes are canonical vs. approximate (e.g., a competitor's 10x12 maps to 10x10 row)
- How to handle parking/vehicle and outdoor storage

Build the matrix as specified in `references/unit-taxonomy.md` with:
- Rows = canonical unit sizes, grouped by category (CC first, then Conv, then Parking, then Other)
- Columns = Subject property + each competitor (with distance and operator type in header)
- Subject column always first after row labels

---

## Step 6 — WoW Diff (if prior run exists)

For each competitor × unit cell where both current and prior values exist:

1. Calculate delta: `current_rate − prior_rate`
2. Calculate % change: `delta / prior_rate × 100`
3. Classify direction: ▲ Up / ▼ Down / → Unchanged / ★ New (no prior) / ✗ Dropped (no current)

Build a **Change Log** — a separate summary listing only competitors/sizes where something moved:

```
Change Log — [current date] vs. [prior date]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
▲ Public Operator A: 10x10 CC  $181 → $195  (+$14, +7.7%)
▼ Independent Operator B: 5x10 non-CC  $73 → $65  (−$8, −11.0%)  [added promo: $1 1st mo]
→ [2 competitors unchanged across all tracked sizes]
```

If **no prior run exists**, skip this section and note: *"No prior run available — change
comparison will begin next week."*

---

## Step 7 — Output Structure

Deliver results in this order:

### A. Run Header
```
Subject: [Facility Name] — [Address]
Market Profile: [Organization Label or "None"] | [Market/Submarket or "Unconfigured"]
Run Date: [Today]  |  Prior Run: [Date or "None"]
Comp Set: [N] facilities within [X] miles
```

### B. Rate Matrix

See `references/unit-taxonomy.md` for the exact table format. Key rules:
- Subject column is highlighted with ★ prefix in header
- Cells with rate increases vs. prior run get ▲ appended
- Cells with rate decreases get ▼ appended
- New comps or new unit offerings get ★ appended
- Include a Notes row at the bottom of each category section

### C. Change Log (WoW Summary)

As specified in Step 6. If first run, replace with: *"First run — baseline established."*

### D. Market Position Summary

4–6 sentence narrative:
- Where subject property sits vs. comp range for the 3–4 most strategic sizes
- Any notable occupancy signals across the comp set
- Whether comps are broadly tightening or loosening pricing
- REIT vs. independent spread

### E. Action Items

**Read `references/action-flags.md` before generating this section.**

Present as a prioritized list:
```
ACTION ITEMS FOR [FACILITY NAME]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
HIGH  | 10x10 CC: You are $24 below market median ($157 vs. $181 avg).
          Recommend: Raise to $175 (+$18). Comps: Public Operator A $195 ▲,
          Independent Operator B $168.

MED   | 5x10 non-CC: 2 of 4 comps cut rates this week (avg −$9).
          Recommend: Hold for 2 more weeks; watch for demand softness before matching.

HOLD  | 10x20 CC: You are within 5% of market median. No action needed.

INFO  | A public operator added "$1 first month" promo on 5x10 non-CC.
          Likely occupancy softness — monitor but no rate change recommended.
```

Limit to the **top 5 most important action items** to keep it actionable, not overwhelming.

### F. Saved Run JSON

At the very end of the output, provide the JSON blob for this run so the user can save it
for next week's comparison. Format is defined in `references/storage-history.md`.

Also include or update the reusable Market Profile JSON when it changed during the run.

---

## Step 8 — Housekeeping Notes

- Tell the user to save the JSON block at the end of the output as their new baseline for the
  next weekly run.
- Suggest: "Paste this JSON at the start of your next comp check to enable WoW comparison."
- If a Market Profile was created or updated, tell the user that future runs should start
  with that profile or a prior saved-run JSON to avoid re-establishing markets.
- If rates for fewer than 3 competitors were successfully retrieved, flag data quality and
  suggest widening search or checking the aggregator sites manually.

---

## Error Handling

- If a facility's page returns no rates (JS-rendered), fall back to SpareFoot → RentCafe →
  facility website
- If fewer than 3 comps have data, note the limitation and widen radius
- If a prior run JSON fails to parse, proceed as a first run and notify the user
- If the subject facility's own rates aren't provided by the user, note "Subject rates not
  provided — add your own rates to the Subject column for full comparison"
