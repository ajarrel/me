---
name: self-storage-reit-intel
description: >
  Public self-storage REIT earnings intelligence for owner-operators, investors,
  lenders, and market analysts. Trigger whenever the user asks about public storage
  company results, operator earnings, REIT earnings calls, sector benchmarks,
  competitor operator intelligence, forward guidance, or anything like "what are
  public operators saying" or "how did the storage REITs do." Also trigger
  proactively when the user is preparing for an investor presentation, ownership
  update, board package, lender discussion, market planning, or deal-underwriting
  context where public operator benchmarks would be useful. On first run, establish
  the user's organization profile, target markets, submarkets, and relevant peer
  set before doing data work.
---

# Self-Storage REIT Intelligence Skill

Produces an investor-grade, slide-deck-ready earnings brief covering the **most recent
reported quarters** of public self-storage REIT results, translated into the user's
configured market and operating context.

Default output: **interactive widget dashboard + markdown brief file**. Both render in one run.

---

## First-Run Market Setup

Before any earnings research, determine whether a reusable **Market Profile** exists in
the conversation, user-provided files, or prior run output. If no profile exists, pause
and ask the user to establish it. Do not assume a company, state, region, portfolio size,
submarket list, or peer set.

Ask for the minimum setup needed to make the brief useful:

1. **Organization label**: company name, portfolio name, or neutral label to use in callouts.
2. **Primary geography**: state, region, country, or national lens.
3. **Target markets and submarkets**: MSAs, counties, cities, trade areas, corridors, or
   named operating clusters the brief should map back to.
4. **Portfolio context**: approximate facility count, operating model, customer type, and
   any known strengths or constraints.
5. **Priority questions**: rate posture, occupancy pressure, supply pipeline, demand
   drivers, technology adoption, expense control, acquisitions, lending, or budget planning.
6. **Peer emphasis**: all public self-storage REITs by default, or a user-specified subset
   of operators/tickers.

After setup, restate the profile as JSON so it can be reused:

```json
{
  "organization_label": "",
  "primary_geography": "",
  "markets": [
    {
      "name": "",
      "submarkets": [],
      "notes": ""
    }
  ],
  "portfolio_context": "",
  "priority_questions": [],
  "peer_emphasis": "all current public self-storage REITs"
}
```

If the user says to proceed without setup, use a **national, investor-neutral lens** and
state that no market-specific read-through is available.

---

## Scope Parameters (set at start of run)

Fill these in explicitly at the top of the response before any data work. If the user
doesn't specify a value, use the default and state the assumption.

| Parameter | Default | Notes |
|---|---|---|
| `horizon` | 5 reported quarters | User can override, e.g. 8 quarters for longer-cycle view |
| `peer_set` | Current public self-storage REITs | Confirm current tickers at run time; user can supply a subset |
| `market_profile` | User-configured profile | If missing, run First-Run Market Setup before proceeding |
| `lens` | Market-profile lens | Alternatives: national, regional, investor-neutral, lender, board |
| `output` | Widget + `.md` brief | Alternatives: widget only, `.md` only, + `.pptx` |
| `audience` | Market-profile audience | Alternatives: ownership group, external investor, lender, board |

State the parameter block in the first line of the response. Example:

> **Scope:** 5 reported quarters, current public self-storage REIT peer set, configured market-profile lens, widget + .md, ownership audience.

---

## Covered Public Operators

Build the peer set dynamically at run time:

1. Identify all currently public self-storage REITs or public owner-operators that report
   comparable quarterly self-storage metrics.
2. Include any user-specified operators/tickers even if they are not REITs, but label them
   separately if comparability is limited.
3. Drop companies that have been acquired, delisted, or stopped reporting comparable
   quarterly metrics, and note the change in the scope block.
4. Add newly public operators once they have at least two quarters of public filings.

Do not hard-code a permanent peer set. Public operator availability changes over time.

---

## Workflow

### Step 1 - Determine Reporting Window

Today's date is known from the system prompt. Identify the most recently **reported**
quarters in the requested horizon.

Quarter reporting calendar:
- Q1 closes Mar 31; reports late Apr / early May
- Q2 closes Jun 30; reports early Aug
- Q3 closes Sep 30; reports early Nov
- Q4 closes Dec 31; reports mid-to-late Feb of the following year

Confirm timing and the currently active public peer set with current sources before
committing the window.

If the current date falls inside a reporting window, check whether all selected peers have
reported. If not, either ask whether the user wants to wait for completeness or mark the
missing peer's most recent quarter as "pending" and proceed with the available reports.

### Step 2 - Gather Data (Structured Collection Table)

For each selected public operator and each quarter in the horizon, fill the collection
table below **before rendering any output**. This forces completeness and prevents
under-sourced narrative.

Data collection priority order:
1. Company investor-relations releases and SEC filings
2. Earnings call transcripts
3. Reputable industry recaps and quarterly sector reports
4. Market and supply reports for the configured geographies

**Per-operator per-quarter data points:**

| Core metrics | Notes |
|---|---|
| Same-store ending occupancy (%) | Stated period-end figure; not average |
| Same-store revenue change YoY (%) | One decimal precision |
| Same-store opex change YoY (%) | One decimal precision |
| Same-store NOI change YoY (%) | One decimal precision |
| Core FFO per share or equivalent | Use company-defined metric and label it |
| Core FFO change YoY (%) | Compute if not stated and inputs are available |
| RevPAF or revenue per occupied square foot | State which measure is used |
| Achieved rate vs. asking rate | If disclosed; spread indicates concession pressure |
| Move-in rate change YoY (%) | Leading indicator for future revenue |
| ECRI contribution to revenue | If disclosed; label as percentage points or percent |

**Per-operator commentary capture:**
- New supply / construction pipeline commentary, national and market-specific
- Market callouts that overlap with the user's configured markets
- Technology and operational initiatives
- Pricing strategy language, including ECRI cadence, concessions, and move-in rates
- Demand driver framing, including housing turnover, migration, rates, and local economy

**For the most recent quarter only:**
- Full-year forward guidance where available
- Management tone indicators

**For the fifth-oldest quarter or first quarter in the horizon:**
- Initial full-year guidance from that period when available, for guidance accuracy tracking

### Step 3 - Compute Derived Metrics

After collecting raw data, compute:

1. **Sequential change (QoQ)** for every metric: direction arrow + magnitude.
   - Up = improved vs. prior quarter
   - Down = deteriorated vs. prior quarter
   - Flat = within +/-10 bps for occupancy or +/-0.2% for growth metrics
2. **Horizon trajectory**: Accelerating / Stabilizing / Decelerating / Choppy / Bottoming.
3. **Cross-peer dispersion per metric per quarter:**
   - Median
   - Range (high minus low)
   - Standard deviation
4. **Consensus call** for each quarter:
   - "Consensus" = at least 80% of peers moving in the same direction
   - "Split" = a visible majority but not 80%
   - "Divergent" = no clear majority

### Step 4 - Guidance Accuracy Tracking

For each peer, compare initial guidance to actual results:

| Score | Criterion |
|---|---|
| Beat | Full-year actual exceeded guidance midpoint issued at start of year |
| Met | Within +/-25 bps of midpoint |
| Missed (soft) | 25-75 bps below midpoint |
| Missed (hard) | More than 75 bps below midpoint |

Score both the **most recent completed year** and the **prior year** when available so the
brief can distinguish optimistic, conservative, accurate, and volatile forecasters.

Flag any operator that revised guidance mid-year, because that changes how heavily its
current guidance should be weighted.

### Step 5 - Market Relevance Filter

Apply the user's Market Profile to all commentary.

**Flag as market-relevant:**
- Direct mentions of configured states, regions, MSAs, counties, cities, submarkets, or
  trade areas
- Comparable adjacent regions or market types when direct mentions are unavailable
- Supply pipeline trends affecting the configured markets or similar growth profiles
- Demand drivers tied to the configured markets
- Technology rollouts applicable to the user's operating model
- Platform-agnostic operational practices such as expense control, insurance, property
  taxes, tenant protection, delinquency management, and staffing model
- Pricing strategy shifts that could affect local rate posture

**Note but do not spotlight:**
- Geography-specific regulation or supply dynamics outside the configured markets unless
  they signal a broader industry trend
- Capital markets events unless they affect guidance, acquisition appetite, or operating
  strategy relevant to the user

### Step 6 - Render Output

Render both deliverables in one run when tooling supports it:
1. Interactive widget dashboard
2. Markdown brief file saved to the available output directory and surfaced to the user

Offer, but do not auto-produce, a slide version if the user wants it for a presentation.

---

## Output Format

The brief has **seven sections**. Always produce all seven. Each section ends with a
single-sentence **"So What"** callout that synthesizes the section for the configured
organization and market profile.

---

### Section 0: Executive Summary (The 30-Second Read)

Three bullets, each two sentences max. Written for someone who will read this section only.

Format:
- **Sector state:** One sentence characterizing where the sector is in the cycle. One
  sentence on the implication.
- **Biggest surprise:** The data point most different from recent consensus expectations.
  One sentence on what it means.
- **Market read-through:** The most important takeaway for the configured markets.

---

### Section 1: Sector Snapshot Table

Comparative table across all quarters in the horizon, one sub-table per metric. Column
order is selected peers, then Median, then Range.

Repeat for each of these metrics:
- Ending occupancy (%)
- Same-store revenue change YoY (%)
- Same-store opex change YoY (%)
- Same-store NOI change YoY (%)
- Move-in rate change YoY (%)
- Core FFO/share or equivalent YoY (%)

Example structure:

| Quarter | Peer 1 | Peer 2 | Peer 3 | Median | Range |
|---|---|---|---|---|---|
| Q4'24 | | | | | |
| Q1'25 | | | | | |
| **Sequential change** | Up/Down/Flat | | | | |
| **Trajectory** | Stabilizing | Accelerating | | | |

Color coding convention in widget:
- Green: positive / improving
- Amber: flat / slightly negative
- Red: materially negative
- Sequential direction follows improvement/deterioration, not raw numeric sign when lower
  is better.

**So What:** [one sentence synthesizing the sector snapshot into a read for the configured markets]

---

### Section 2: Trend Visualizations (Widget)

Render all charts inside one widget with tab or scroll navigation.

**Chart A - Same-Store NOI Trajectory**
Line chart, one line per peer across the horizon. Y-axis: same-store NOI change YoY (%).
Include a median line. Zero line is solid gray.

**Chart B - Occupancy Trajectory**
Line chart, one line per peer across the horizon. Y-axis: ending occupancy (%). Add an
appropriate reference line only if it is meaningful for the selected peer set.

**Chart C - Revenue vs. Expense Growth**
Clustered bar for the most recent quarter: per peer, side-by-side bars for revenue change
and expense change. Annotate margin compression or expansion above each cluster.

**Chart D - Cross-Peer Dispersion Over Time**
Line chart showing range across peers for same-store revenue change, same-store NOI change,
and occupancy.

**Chart E - Move-In Rate vs. Revenue Growth Scatter**
One dot per peer for the most recent quarter. X-axis: move-in rate change YoY. Y-axis:
same-store revenue change YoY.

**Style directives:**
- Use a readable professional dashboard style
- Assign stable, high-contrast colors to peers dynamically
- Include labels, source notes, and quarter labels

**So What:** [one sentence on what the trajectory means for the configured rate and occupancy posture]

---

### Section 3: Cross-Peer Dispersion & Consensus

**3A. Direction Consensus Table**

For the most recent quarter, show per-metric:

| Metric | Peers positive | Peers negative | Call |
|---|---|---|---|
| Same-store revenue change YoY | | | Consensus up / Split / Consensus down |
| Same-store NOI change YoY | | | |
| Ending occupancy QoQ | | | |
| Move-in rate change YoY | | | |

**3B. Dispersion Commentary**

Two paragraphs:
1. Where the sector is converging and what sector-wide dynamics dominate.
2. Where the sector is diverging and which operator strategy, portfolio composition, or
   market exposure explains the outliers.

**3C. Outlier Spotlight**

One short paragraph on the most differentiated peer this quarter and the two most likely drivers.

**So What:** [one sentence on whether the configured markets should track consensus or outlier behavior]

---

### Section 4: Forward Guidance & Tone

**4A. Guidance Table (Most Recent Quarter)**

| Peer | SS Rev Guidance | SS Opex Guidance | SS NOI Guidance | FFO / Earnings Guidance | Tone |
|---|---|---|---|---|---|

Tone: Cautious / Neutral / Constructive / Bullish.

Show both the range and the midpoint whenever available.

**4B. Sector Consensus Outlook**

Two to three sentences on the implied full-year direction. Identify the most bullish and
most cautious peer and what each is betting on.

**4C. Pending Corporate Events**

Flag material events that affect guidance interpretation, such as pending M&A, leadership
changes, major capital raises, dispositions, or strategy shifts.

**So What:** [one sentence on what the guidance band implies for the configured budget or underwriting assumptions]

---

### Section 5: Guidance Accuracy Tracking

Table of full-year guidance vs. actuals for the most recent two completed years when available:

| Peer | Year-1 Guide Mid | Year-1 Actual | Year-1 Score | Year-2 Guide Mid | Year-2 Actual | Year-2 Score | Pattern |
|---|---|---|---|---|---|---|---|

Pattern values: Optimistic / Conservative / Accurate / Volatile / Insufficient history.

One paragraph below the table: whose guidance should be weighted most heavily for the
configured use case, and whose guidance should be discounted.

**So What:** [one sentence on which public guidance is most useful for the configured outlook]

---

### Section 6: Market Operator Intelligence Brief

This section translates public-operator commentary into the user's configured geography
and operating model. Write in plain prose with subheadings.

**6A. Configured Market Signals**
Direct market mentions from the quarter. If none exist, use clearly labeled analogs from
similar market types and explain the inference.

**6B. Supply Pipeline Watch**
New supply commentary with relevance to the configured markets. Extract any direct mentions
of the configured submarkets; otherwise compare against adjacent or structurally similar
markets with clear caveats.

Map explicitly to each configured market or submarket from the Market Profile.

**6C. Demand Drivers**
Housing market, migration, employment, household formation, and transaction-volume
commentary. Pull current local facts from sources at run time and cite them; do not rely on
stored demographic assumptions.

**6D. Technology & Operational Practices Worth Adopting**
What public operators are investing in that the configured operator should watch. For each
initiative:
- What it is
- Which peer(s) are doing it
- Relevance: Watch / Relevant / Act Now
- Applicability note for the configured operating model

Baseline categories to check every run: AI pricing, dynamic pricing cadence, digital
marketing attribution, centralized call centers, self-service kiosks, third-party management
platforms, revenue management software, lien/auction automation, tenant insurance or
protection plan penetration, delinquency management automation, and move-in workflow automation.

**6E. Rate & Pricing Intelligence**
Move-in rate trends, ECRI cadence, concession strategies, and discounting philosophy. Frame
as what pricing environment the configured operator is competing in.

**6F. Immaterial Flags**
One paragraph listing public-operator callouts that were considered but are not relevant to
the configured markets, with a short reason for exclusion.

**So What:** [two-sentence synthesis closing the section]

---

## Quality Standards

- All percentages presented with one decimal place precision when the source supports it
- Occupancy changes shown in basis points
- If a data point is unavailable after two search attempts, show "N/A" rather than estimating;
  make one more attempt via the company's most recent filing before giving up
- Never conflate full-year figures with single-quarter figures
- Guidance ranges must show both the range and the midpoint when available
- Distinguish clearly between same-store portfolio metrics and total-portfolio metrics
- Every specific statistic cited in the brief must have an inline source and quarter label
- Sequential arrows require a quarter-over-quarter comparison of the same metric
- Do not carry market-specific demographic facts from memory; pull fresh local data at run
  time and cite it
- Clearly label inferred market read-throughs when there is no direct mention of the
  configured markets

---

## Market Context (Configured at Run Time)

Use the Market Profile established on first run. Market context can include:
- Primary geography and operating regions
- MSAs, counties, cities, corridors, and named submarkets
- Portfolio size and operating model
- Known competitors and peer emphasis
- Priority strategic questions
- Known supply, demand, rate, or occupancy concerns

Re-verify any structural facts before citing them in output.

---

## Refresh Cadence

Designed to re-run quarterly after each earnings season:

| Run | Timing | Captures |
|---|---|---|
| Q1 results | Mid-May | Prior Q2, Q3, Q4 + current Q1 + prior year Q1 |
| Q2 results | Mid-August | Prior Q3, Q4, Q1 + current Q2 + prior year Q2 |
| Q3 results | Mid-November | Prior Q4, Q1, Q2 + current Q3 + prior year Q3 |
| Q4 / FY results | Late February | Prior Q1, Q2, Q3 + current Q4 + prior year Q4 |

When re-running, use prior output or the Market Profile to populate guidance accuracy
tracking and to flag what changed in sector narrative since the last run.

---

## Markdown Brief File

Save the full brief to the available output directory using a neutral filename such as
`reit-intel-brief-Q[N]-[YEAR].md`. File structure mirrors Sections 0-6 above. First line:
scope parameter block. Last line: `Prepared [date]. Sources: [primary sources used].`
