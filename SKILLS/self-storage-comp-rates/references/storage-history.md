# Rate History — Persistence Schema

This file defines the JSON format used to save and reload weekly comp rate snapshots.
The user saves this blob after each run and pastes it in at the start of the next run
to enable week-over-week comparison.

---

## Schema

```json
{
  "schema_version": "1.0",
  "run_date": "2026-04-06",
  "subject": {
    "name": "American Self Storage – Clayton East",
    "address": "123 Powhatan Rd, Clayton, NC 27520",
    "rates": {
      "CC": {
        "5x5":  null,
        "5x10": 109,
        "10x10": 157,
        "10x15": 185,
        "10x20": 225,
        "10x30": null
      },
      "CONV": {
        "5x5":  null,
        "5x10": 79,
        "10x10": 109,
        "10x15": 135,
        "10x20": 159,
        "10x30": 275
      },
      "PARK": {
        "uncovered": null,
        "covered": null,
        "rv_boat": null
      }
    },
    "promos": []
  },
  "competitors": [
    {
      "id": "exr-clayton-blvd",
      "name": "Extra Space – Clayton Blvd",
      "address": "12984 Clayton Blvd, Clayton, NC",
      "distance_miles": 3.5,
      "operator_type": "REIT",
      "source_url": "https://www.storagecafe.com/storage/nc/clayton/extra-space-storage-1234/",
      "rates": {
        "CC": {
          "5x5":  75,
          "5x10": 116,
          "10x10": 181,
          "10x15": 215,
          "10x20": 292,
          "10x30": 434
        },
        "CONV": {
          "5x5":  null,
          "5x10": null,
          "10x10": null,
          "10x15": null,
          "10x20": null,
          "10x30": null
        },
        "PARK": {
          "uncovered": null,
          "covered": null,
          "rv_boat": null
        }
      },
      "promos": [
        { "description": "First month free on select units", "applies_to": "CC 10x10" }
      ],
      "occupancy_signals": "Low availability on 10x10 CC — 2 units left"
    }
  ],
  "notes": "First run for this facility. Clayton East submarket."
}
```

---

## Field Reference

| Field | Type | Description |
|---|---|---|
| `schema_version` | string | Always `"1.0"` for this version |
| `run_date` | string | ISO date of this run (YYYY-MM-DD) |
| `subject.name` | string | Full facility name |
| `subject.address` | string | Full street address |
| `subject.rates` | object | Nested by category → canonical size → monthly rate (integer) or null |
| `subject.promos` | array | List of active promos on subject property |
| `competitors[].id` | string | Kebab-case stable identifier (used to match across runs) |
| `competitors[].operator_type` | enum | `"REIT"`, `"Reg"`, `"Ind"` |
| `competitors[].source_url` | string | Where rates were pulled from |
| `competitors[].rates` | object | Same nested structure as subject.rates |
| `competitors[].promos` | array | Active promos found on listing |
| `competitors[].occupancy_signals` | string | Free text — availability language observed on listing |

---

## WoW Diff Logic (how the skill uses this)

When a prior run JSON is loaded:
1. Match competitors by `id` field (stable across runs)
2. For each matched competitor × size cell, compute `current − prior` and `% change`
3. Competitors in current run but not prior = **new** (mark ★ new)
4. Competitors in prior but not current = **dropped** (note in change log)
5. Rate cells present in prior but null in current = **no longer listed** (note `—` with prior rate in parens)

---

## User Instructions (include at end of every report)

> **💾 Save this for next week:**
> Copy the JSON block below and paste it at the beginning of your next comp check request.
> This enables week-over-week rate change detection.
>
> Example prompt: *"Run my weekly comp check for [facility]. Here's last week's data: [paste JSON]"*

---

## Versioning

If the schema needs to change in a future version, increment `schema_version`. The skill
should gracefully handle v1.0 JSON even if later versions exist — just note any missing
fields and proceed with available data.