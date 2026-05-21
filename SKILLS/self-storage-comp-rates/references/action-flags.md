# Action Item Flags — Decision Logic

This file defines the rules for generating the Action Items section of a comp rate report.
Flags are designed to be opinionated but brief — the goal is 3–5 high-signal items, not a
comprehensive audit.

---

## Priority Levels

| Level | Symbol | When to Use |
|---|---|---|
| HIGH | 🔴 | Clear pricing gap or urgent market move; revenue impact likely |
| MEDIUM | 🟡 | Emerging signal, directionally important but warrants monitoring |
| HOLD | 🟢 | Subject is well-positioned; no near-term action needed |
| INFO | ℹ️ | Relevant market intelligence; no rate action implied |

Limit output to **top 5 items max**. Prioritize HIGH first, then MED, then HOLD/INFO only if
there's useful context.

---

## Flag Generation Rules

### 🔴 HIGH — Raise Rates (Subject is Below Market)

Trigger when:
- Subject rate for a size/type is **>10% below** the comp median for that same cell
- Applies to CC and CONV unit sizes with ≥ 2 comp data points

Recommended action wording:
> "You are $X below market median ($Y vs. $Z avg comps). Recommend raising to $[target],
> which keeps you slightly below the highest comp to minimize churn risk."

Target pricing rule:
- Set target at **comp median − 5%** (stay competitive but capture upside)
- If REIT comps are the only ones above, weight indie comps more heavily to avoid chasing
  algorithmic REIT pricing

### 🔴 HIGH — Match a Key Comp Move (Comp Raised Significantly)

Trigger when:
- A key competitor (REIT or closest comp) **raised rates ≥ 8%** WoW on a major unit size
  (5x10, 10x10, 10x20)
- Subject is at or below the old comp rate

Recommended action wording:
> "[Comp name] raised [size] CC from $X → $Y (+Z%). This opens room to raise your rate
> to $[target] without being the market high. Recommend matching within 2 weeks."

### 🟡 MEDIUM — Hold and Monitor (Comp Cut Rates)

Trigger when:
- 1–2 comps **cut rates or added a promo** on a size the subject also offers
- Subject is currently within 5% of those comps

Recommended action wording:
> "[N] comps cut [size] rates this week (avg −$X). Recommend holding your rate for 2 more
> weeks to assess demand impact before matching. If occupancy softens, revisit."

Do NOT recommend the subject immediately cut rates to match — give it time.

### 🟡 MEDIUM — Promo Gap (Comp Running Promo, Subject Is Not)

Trigger when:
- A major comp is running a notable promo ($1 first month, 50% off first 3 months) and
  the subject is not running any promo on that unit type

Recommended action wording:
> "[Comp] is running a [promo description] on [size]. If your occupancy on that unit type
> is below 85%, consider matching with a 1-month promo. If occupancy is strong, hold."

### 🟢 HOLD — Well-Positioned

Trigger when:
- Subject rate is within **±5% of comp median** for a major size
- No notable comp movement this week

Wording:
> "[Size] [type]: Priced within 5% of market median. No action needed."

Generate one consolidated HOLD note covering all well-positioned sizes rather than listing
each individually.

### ℹ️ INFO — Market Intelligence Only

Use for:
- Availability signals (occupancy highs/lows at a specific comp)
- A comp that opened or closed
- A comp that joined/left aggregators (data availability change)
- Broad market narrative (all REITs softening, independents holding firm, etc.)

---

## Formatting Template

```
⚡ ACTION ITEMS FOR [FACILITY NAME — SHORT MARKET LABEL]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔴 HIGH   | [Unit Size + Type]: [1-line issue statement]
           [Comp context: comps and their rates]
           Recommend: [Specific action + target rate or timing]

🟡 MED    | [Unit Size + Type]: [1-line issue statement]
           Recommend: [Specific action or watch directive]

🟢 HOLD   | [Sizes covered]: [Why no action needed]

ℹ️  INFO   | [Market observation — 1–2 lines max]
```

---

## Subject Rates Not Provided

If the user did not provide the subject facility's own current rates:
- Skip HIGH/HOLD flags that require subject rate comparison
- Still generate comp-movement flags (comp raised/cut independently of subject positioning)
- Note at top of Action Items: *"Add your current rates to the Subject column to unlock
  full positioning analysis."*

---

## Rate-Change Thresholds Reference

| Change | Description |
|---|---|
| ≥ +10% | Significant raise — HIGH flag if subject hasn't moved |
| +5–9% | Moderate raise — MED flag |
| < ±5% | Noise / minor adjustment — HOLD or INFO only |
| −5 to −9% | Moderate cut — MED "monitor" flag |
| ≤ −10% | Significant cut — MED/HIGH depending on subject positioning |
| New promo added | Always ℹ️ INFO; escalate to 🟡 MED if subject may need to respond |