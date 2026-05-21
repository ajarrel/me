---
name: self-storage-reit-intel
description: >
  Public REIT earnings intelligence for self-storage owner-operators. Trigger whenever
  the user asks about REIT results, public storage company performance, what the big
  operators reported, how REITs are doing, REIT earnings calls, sector benchmarks,
  competitor operator intelligence, or anything like "what are the REITs saying" or
  "how did Extra Space / Public Storage / CubeSmart / NSA / SmartStop do." Also trigger
  proactively when the user is preparing for an investor presentation, speaking
  engagement, ownership update, or deal-underwriting context where REIT sector
  benchmarks would be relevant. Produces an investor-grade brief covering the most
  recent 5 quarters of REIT results with sequential + YoY trend deltas, cross-REIT
  dispersion stats, NC-flagged commentary, forward guidance, and an NC operator
  intelligence section on practices and technologies worth adopting.
---
 
# Self-Storage REIT Intelligence Skill
 
Produces an investor-grade, slide-deck-ready earnings brief covering the **most recent 5 quarters**
of public self-storage REIT results, with a **North Carolina regional operator lens** for ASSI.
 
Default output: **interactive widget dashboard + markdown brief file**. Both render in one run.
 
---
 
## Scope Parameters (set at start of run)
 
Fill these in explicitly at the top of the response before any data work. If the user didn't
specify a value, use the default and state the assumption.
 
| Parameter | Default | Notes |
|---|---|---|
| `horizon` | 5 quarters (current + 4 trailing) | User can override (e.g. 8 quarters for a longer-cycle view) |
| `reit_set` | Core 5: PSA, EXR, CUBE, NSA, SMA | Drop NSA once PSA acquisition closes |
| `lens` | NC / Southeast | Alternatives: "Sun Belt broad", "national", "investor-neutral" |
| `output` | Widget + `.md` brief | Alternatives: widget only, `.md` only, + `.pptx` |
| `audience` | ASSI ownership group | Alternatives: "external investor", "lender", "board" |
 
State the parameter block in the first line of the response. Example:
 
> **Scope:** 5 quarters (Q4'24 → Q4'25), PSA/EXR/CUBE/NSA/SMA, NC-Southeast lens, widget + .md, ASSI ownership audience.
 
---
 
## Covered REITs (Core Set)
 
| Ticker | Company | Notes |
|--------|---------|-------|
| PSA | Public Storage | Largest by unit count; bellwether for sector commentary |
| EXR | Extra Space Storage | Largest by facility count post-LSI merger; heavy 3rd-party mgmt |
| CUBE | CubeSmart | Most NYC/Boston-concentrated; most urban portfolio |
| NSA | National Storage Affiliates Trust | **Pending acquisition by PSA** (announced early 2026 at ~$41.68/share); guidance interpretation should flag this |
| SMA | SmartStop Self Storage | IPO'd April 2025; smallest of the five but tech-forward; explicit AI/automation positioning |
 
**Drop criteria:** If a REIT is acquired and stops filing quarterly, drop from core set and note in scope block. If a new self-storage REIT IPOs and has 2+ quarters of public filings, add to core set.
 
---
 
## Workflow
 
### Step 1 — Determine Reporting Window
 
Today's date is known from the system prompt. Identify the 5 most recently **reported** quarters.
 
Quarter reporting calendar:
- Q1 closes Mar 31; reports late Apr / early May
- Q2 closes Jun 30; reports early Aug
- Q3 closes Sep 30; reports early Nov
- Q4 closes Dec 31; reports mid-to-late Feb (next year)
Confirm timing with one search before committing the window:
 
```
web_search: "self storage REIT earnings most recent quarter results [year]"
```
 
If the current date falls inside a reporting window (e.g. mid-Feb, early May), check whether all five REITs have reported yet. If not all five have reported, either (a) wait for completeness if user can delay, or (b) mark the missing REIT's most recent quarter as "pending" and proceed with the other four.
 
### Step 2 — Gather Data (Structured Collection Table)
 
For each of the 5 REITs × 5 quarters, fill the collection table below **before rendering any output**. This forces completeness and prevents the "I'll just run a search and see what comes back" failure mode.
 
Data collection priority order:
1. Company IR press releases / 8-K filings (SEC EDGAR direct)
2. InsideSelfStorage.com quarterly recap articles
3. SkyView Advisors quarterly industry reports
4. Earnings call transcripts (Motley Fool, Seeking Alpha, Fintool)
5. Yardi Matrix / StorTrack sector reports (for regional context)
**Per-REIT per-quarter data points:**
 
| Core metrics | Notes |
|---|---|
| Same-store ending occupancy (%) | Stated period-end figure; NOT average |
| Same-store revenue Δ YoY (%) | One decimal precision |
| Same-store opex Δ YoY (%) | One decimal precision |
| Same-store NOI Δ YoY (%) | One decimal precision |
| Core FFO per share ($) | As-reported |
| Core FFO Δ YoY (%) | Computed if not stated |
| **RevPAF or Rev/Occ sq ft** | Revenue per available square foot OR per occupied sq ft — state which |
| **Achieved rate vs. asking rate** | If disclosed; spread indicates discount/concession pressure |
| **Move-in rate Δ YoY (%)** | Leading indicator for next-quarter revenue |
| **ECRI contribution to revenue** | % points of revenue growth from existing-customer rate increases |
 
**Per-REIT commentary capture:**
- New supply / construction pipeline commentary (national + any regional detail)
- Market callouts (flag NC/Southeast/Sun Belt separately from CA/NYC/etc.)
- Technology / operational initiatives mentioned
- Pricing strategy language (ECRI cadence, concession philosophy, move-in rate direction)
- Demand driver framing (housing turnover, migration, rates)
**For the most recent quarter only:**
- Full-year forward guidance (SS revenue, expense, NOI ranges + midpoints, Core FFO range + midpoint)
- Management tone indicators (phrases like "remain cautious," "encouraged by," "stabilization," "bottoming")
**For the fifth-oldest quarter in the horizon:**
- What they guided for the full year at that time (needed for Step 4 guidance tracking)
### Step 3 — Compute Derived Metrics
 
After collecting raw data, compute:
 
1. **Sequential Δ (QoQ)** for every metric — direction arrow + magnitude
   - ↑ = improved vs prior quarter
   - ↓ = deteriorated vs prior quarter
   - → = flat (within ±10 bps for occupancy, ±0.2% for growth metrics)
2. **5-quarter trajectory** — one of: Accelerating / Stabilizing / Decelerating / Choppy / Bottoming
   - Look at the direction of change across all 5 quarters, not just endpoints
3. **Cross-REIT dispersion per metric per quarter:**
   - Median across the 5 REITs
   - Range (high − low)
   - Standard deviation (useful for "is the sector moving in unison?")
4. **Consensus call** for each quarter:
   - "Consensus" = 4 of 5 REITs moving in the same direction
   - "Split" = 3-2 or 2-2-1 direction mix
   - "Divergent" = no clear majority
### Step 4 — Guidance Accuracy Tracking (Persistent)
 
For each REIT, compare:
 
| Score | Criterion |
|---|---|
| ✅ Beat | Full-year actual exceeded guidance midpoint issued at start of year |
| ➖ Met | Within ±25 bps of midpoint |
| ⚠️ Missed (soft) | 25–75 bps below midpoint |
| ❌ Missed (hard) | More than 75 bps below midpoint |
 
Score both the **most recent completed year** and the **prior year** so the brief can show who is chronically optimistic vs. conservative. A single data point is anecdote; two is pattern.
 
Flag any REIT that has revised guidance mid-year (typically after Q2 or Q3 prints) — this is a meaningful signal.
 
### Step 5 — NC Relevance Filter
 
Apply to all commentary:
 
**Flag as NC-relevant (include in NC Intel section):**
- Direct mentions: Sunbelt, Southeast, Mid-Atlantic, Carolinas, NC, Raleigh, Charlotte, Research Triangle, Triad, Wilmington
- MSA-level: Any Top-50 MSA named that applies regionally (Charlotte-Concord MSA, Raleigh-Cary MSA, Durham-Chapel Hill MSA, Greensboro-High Point MSA, Winston-Salem MSA, Fayetteville MSA, Wilmington MSA)
- Supply pipeline trends affecting Sun Belt / Southeast
- Demand drivers tied to Southeast migration / housing turnover
- Technology rollouts applicable to regional operators (AI pricing, digital marketing, automation)
- Platform-agnostic operational practices (expense control, insurance, property taxes, tenant protection plan penetration)
- Pricing strategy shifts (ECRI cadence, concession philosophy)
**Note but don't spotlight (Immaterial Flags):**
- CA rent control / LA state-of-emergency rent restrictions
- NYC / Boston / DC supply-constrained premium market commentary
- West Coast or Northeast-specific market dynamics
- REIT-specific capital structure (bond issuances, JV structures) unless it signals industry trend
### Step 6 — Render Output
 
Render **both** deliverables in one run:
1. Interactive widget dashboard via `show_widget`
2. Markdown brief file saved to `/mnt/user-data/outputs/` and surfaced via `present_files`
Offer (do not auto-produce) a `.pptx` slide version if the user wants it for an upcoming presentation.
 
---
 
## Output Format
 
The brief has **seven sections**. Always produce all seven. Each section ends with a **"So What for ASSI"** single-sentence callout that synthesizes the section for a 20-facility NC operator.
 
---
 
### Section 0: Executive Summary (The 30-Second Read)
 
Three bullets, each two sentences max. Written for someone who will read this section only.
 
Format:
- **Sector state:** One sentence characterizing where the sector is in the cycle (e.g. "Stabilizing with divergence — PSA and EXR decelerating while SMA and CUBE show sequential improvement"). One sentence on the implication.
- **Biggest surprise:** The one data point most different from 6-month-ago consensus expectations. One sentence on what it means.
- **NC read-through:** The single most important takeaway for a Raleigh/Charlotte-region operator.
---
 
### Section 1: Sector Snapshot Table (5-Quarter Matrix)
 
Comparative table across all 5 quarters, one sub-table per metric. Column order always:
**PSA | EXR | CUBE | NSA | SMA | Median | Range**
 
Repeat for each of these metrics:
- Ending Occupancy (%)
- SS Revenue Δ YoY (%)
- SS Opex Δ YoY (%)
- SS NOI Δ YoY (%)
- Move-in Rate Δ YoY (%)
- Core FFO/sh YoY (%)
Example structure (for one metric):
 
| Quarter | PSA | EXR | CUBE | NSA | SMA | Median | Range (H−L) |
|---|---|---|---|---|---|---|---|
| Q4'24 | | | | | | | |
| Q1'25 | | | | | | | |
| Q2'25 | | | | | | | |
| Q3'25 | | | | | | | |
| Q4'25 | | | | | | | |
| **Sequential Δ** | ↑/↓/→ | ↑/↓/→ | ↑/↓/→ | ↑/↓/→ | ↑/↓/→ | | |
| **5Q trajectory** | Stabilizing | Accelerating | … | | | | |
 
Color coding convention (use in widget):
- Green: positive / improving
- Amber: flat / slightly negative (-0.1% to -1.5%)
- Red: materially negative (worse than -1.5%)
- Sequential arrow color follows direction: ↑ green, ↓ red, → gray
**So What for ASSI:** [one sentence synthesizing the sector snapshot into a read for the Triangle region]
 
---
 
### Section 2: Trend Visualizations (Widget)
 
Render via `show_widget` (HTML/React with Recharts). Reuse the widget across all charts; do not call `show_widget` multiple times — put multiple charts inside one widget with tab or scroll navigation.
 
**Chart A — 5-Quarter Same-Store NOI Trajectory**
Line chart, one line per REIT across all 5 quarters. Y-axis: SS NOI Δ YoY (%). Include a median line (dashed, white) overlaid. Zero line is solid gray.
 
**Chart B — 5-Quarter Occupancy Trajectory**
Line chart, one line per REIT across all 5 quarters. Y-axis: ending occupancy (%). Dashed horizontal reference line at 90%.
 
**Chart C — Revenue vs. Expense Growth (Most Recent Quarter)**
Clustered bar: per REIT, side-by-side bars for revenue Δ and expense Δ. Annotate the gap (margin compression/expansion) above each cluster.
 
**Chart D — Cross-REIT Dispersion Over Time**
Line chart showing (high − low) range across REITs for SS Revenue Δ, SS NOI Δ, and Occupancy, one line each, across all 5 quarters. Rising lines = sector diverging; falling lines = sector converging.
 
**Chart E — Move-In Rate vs. Revenue Growth Scatter**
One dot per REIT for most recent quarter. X-axis: move-in rate Δ YoY. Y-axis: SS revenue Δ YoY. Shows whether revenue is being driven by rate, occupancy, or ECRI.
 
**Style directives:**
- Dark background (`#0f1117`), IBM Plex Mono for labels, IBM Plex Sans for titles
- Palette: PSA=`#4e9af1`, EXR=`#f5a623`, CUBE=`#7ed321`, NSA=`#9b59b6`, SMA=`#e74c3c`, Median=`#ffffff`
- Load `read_me` modules `chart` and `data_viz` before building
**So What for ASSI:** [one sentence on what the trajectory means for ASSI's 2026 rate/occupancy posture]
 
---
 
### Section 3: Cross-REIT Dispersion & Consensus
 
New section — the heart of the "sector vs. individual story" read.
 
**3A. Direction Consensus Table**
 
For the most recent quarter, show per-metric:
 
| Metric | REITs positive | REITs negative | Call |
|---|---|---|---|
| SS Revenue Δ YoY | | | Consensus ↑ / Split / Consensus ↓ |
| SS NOI Δ YoY | | | |
| Ending Occupancy QoQ | | | |
| Move-in Rate Δ YoY | | | |
 
**3B. Dispersion Commentary**
 
Two paragraphs:
1. Where is the sector *converging*? (Metrics where all 5 REITs are moving in the same direction with small range.) This is where sector-wide dynamics dominate.
2. Where is the sector *diverging*? (Metrics with wide range.) This is where individual operator strategy / portfolio composition is the story. Identify the outlier(s) and state why.
**3C. Outlier Spotlight**
 
One short paragraph on the REIT most differentiated this quarter (best or worst) and the two most likely drivers.
 
**So What for ASSI:** [one sentence on whether ASSI's NC submarkets should track the consensus or the outlier]
 
---
 
### Section 4: Forward Guidance & Tone
 
**4A. Guidance Table (Most Recent Quarter)**
 
| REIT | SS Rev Guidance | SS Opex Guidance | SS NOI Guidance | Core FFO Guidance | Tone |
|------|-----------------|------------------|-----------------|-------------------|------|
| PSA  | range (mid)     | range (mid)      | range (mid)     | range (mid)       | |
| EXR  |                 |                  |                 |                   | |
| CUBE |                 |                  |                 |                   | |
| NSA  |                 |                  |                 |                   | |
| SMA  |                 |                  |                 |                   | |
 
Tone: Cautious / Neutral / Constructive / Bullish (single word).
 
Show both the range AND the midpoint (e.g. "-1.0% to +0.5% (midpoint -0.25%)").
 
**4B. Sector Consensus Outlook**
 
2-3 sentences on the implied full-year direction. Identify the most bullish and most cautious REIT and what each is betting on.
 
**4C. Pending Corporate Events**
 
Flag material events that affect guidance interpretation (pending M&A, leadership changes, major capital raises/dispositions). Example: NSA's pending acquisition by PSA makes NSA guidance less forward-useful.
 
**So What for ASSI:** [one sentence on what the guidance band implies for ASSI's 2026 budget assumptions]
 
---
 
### Section 5: Guidance Accuracy Tracking (Two-Year Lookback)
 
Table of full-year guidance vs. actuals for the most recent *two* completed years:
 
| REIT | Year-1 Guide Mid | Year-1 Actual | Year-1 Score | Year-2 Guide Mid | Year-2 Actual | Year-2 Score | Pattern |
|------|------------------|---------------|--------------|------------------|---------------|--------------|---------|
 
Pattern values: "Chronically optimistic," "Chronically conservative," "Accurate," "Volatile."
 
One paragraph below the table: which REITs' guidance to trust more when setting ASSI's budget assumptions, and which to discount.
 
**So What for ASSI:** [one sentence on which REIT's guidance should be weighted most heavily when anchoring ASSI's outlook]
 
---
 
### Section 6: NC Operator Intelligence Brief
 
This section is the most valuable for ASSI. Write in plain prose (not bullets) with subheadings.
 
**6A. NC / Southeast Market Signals**
Any direct market mentions from the quarter. If none, note what commentary implies for Sun Belt markets and specifically the Research Triangle. Flag South-region performance from Yardi Matrix if available.
 
**6B. Supply Pipeline Watch**
New supply commentary with NC relevance. What % of REIT portfolios face competitive supply pressure entering the upcoming year? Is that improving or worsening? Extract any REIT-specific commentary on Charlotte, Raleigh, Greensboro, or Research Triangle submarkets.
 
Map to ASSI submarkets explicitly:
- **Johnston County (Clayton, Angier):** How does commentary apply?
- **Wake County suburbs (Fuquay-Varina, Knightdale):**
- **Chatham County (Pittsboro):**
- **Cabarrus County (Concord):**
- **Triad / High Point:** (Previously flagged as highest-risk for oversupply — does REIT commentary confirm or refute?)
**6C. Demand Drivers**
Housing market and migration commentary. Any direct NC figures (housing turnover, migration) should be pulled from current Census/NAR data, not memorized. Frame: what is the leading indicator for NC demand over the next 12 months?
 
**6D. Technology & Operational Practices Worth Adopting**
What are the REITs investing in that a 20-facility regional operator should watch? For each initiative:
- What it is
- Which REIT(s) are doing it
- NC Relevance: 🔴 Watch / 🟡 Relevant / 🟢 Act Now
- Applicability note for ASSI (reference ASSI's existing custom tech where relevant: gate monitoring, protection plan administration, tenant auditing, deal underwriting)
Baseline categories to check every run: AI pricing agents, dynamic pricing cadence, digital marketing attribution, centralized call centers, self-service kiosks, third-party management platforms, revenue management software, lien/auction automation, tenant insurance/protection plan penetration, delinquency management automation, OCR'd move-in workflows.
 
**6E. Rate & Pricing Intelligence**
REIT move-in rate trends (YoY). ECRI cadence and philosophy commentary. Discount/concession strategies. Frame as: what pricing environment is ASSI competing in, and what are the largest operators doing that applies pressure on or supports regional operator rate-holding?
 
**6F. Immaterial Flags**
One paragraph listing callouts from REIT earnings NOT relevant to NC operations (CA rent control, NYC/Boston supply dynamics, etc.) so the reader can confirm they were considered and consciously excluded.
 
**So What for ASSI (overall):** [two-sentence synthesis closing the section]
 
---
 
## Quality Standards
 
- All % figures presented with one decimal place precision (e.g., -1.1%, not "about -1%")
- Occupancy changes shown in basis points (e.g., "down 70 bps"), not decimal percent
- If a data point is unavailable after two search attempts, show "N/A" rather than estimating — but make one more attempt via the company's most recent 10-Q / 8-K on SEC EDGAR before giving up
- Never conflate full-year figures with single-quarter figures; always label clearly
- Guidance ranges must show both the range AND the midpoint
- Distinguish clearly between same-store portfolio metrics and total-portfolio metrics
- Every specific statistic cited in the brief must have an inline source and quarter label
- Sequential arrows require the comparison to be quarter-over-quarter of the same metric; do not mix YoY and QoQ in the same arrow
- Do **not** carry NC-specific demographic facts (migration figures, housing turnover) from memory — pull fresh from Census/NAR/Yardi at run time and cite
---
 
## NC Market Context (for framing, not citation)
 
ASSI's primary submarkets: Research Triangle (Raleigh, Durham, RTP), Johnston County (Clayton, Angier), Chatham County (Pittsboro), Wake County suburbs (Fuquay-Varina, Knightdale), Cabarrus County (Concord), High Point / Greensboro corridor.
 
Structural facts (re-verify if citing):
- NC consistently ranks top-3 nationally for net domestic migration (behind FL, often tied with TX/SC)
- Charlotte and Raleigh MSAs are Sun Belt population-growth leaders
- Research Triangle historically attracts new self-storage supply along major corridors (US-1, I-40, I-540)
- NC has no statewide rent control regulation — CA-specific REIT commentary is not applicable
- Triad/Greensboro flagged internally by ASSI as highest-risk for oversupply
---
 
## Refresh Cadence
 
Designed to re-run quarterly after each earnings season:
 
| Run | Timing | Captures |
|---|---|---|
| Q1 results | Mid-May | Prior Q2, Q3, Q4 + current Q1 + prior year Q1 |
| Q2 results | Mid-August | Prior Q3, Q4, Q1 + current Q2 + prior year Q2 |
| Q3 results | Mid-November | Prior Q4, Q1, Q2 + current Q3 + prior year Q3 |
| Q4 / FY results | Late February | Prior Q1, Q2, Q3 + current Q4 + prior year Q4 |
 
When re-running, the prior run's output can be provided as context — use it to populate the guidance accuracy tracking (Section 5) and to flag what has changed in sector narrative since last run.
 
---
 
## Markdown Brief File
 
Save the full brief to `/mnt/user-data/outputs/reit-intel-brief-Q[N]-[YEAR].md` and present via `present_files`. File structure mirrors Sections 0-6 above. First line: scope parameter block. Last line: "Prepared [date] · Sources: [primary sources used]."