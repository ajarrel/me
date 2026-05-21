# Unit Taxonomy — Standardization Reference

This file defines how to classify and normalize any self-storage unit type/size into the
canonical matrix used across all comp rate reports.

---

## 4 Standard Categories

| Category Code | Display Name | What It Covers |
|---|---|---|
| CC | Climate-Controlled | Indoor, temperature/humidity controlled units |
| CONV | Conventional (Drive-Up) | Non-climate outdoor/drive-up access units |
| PARK | Parking / Vehicle | Uncovered parking, covered parking, RV/boat/auto |
| OTHER | Other | Lockers, wine storage, records storage, mixed |

### Category Classification Rules

**Climate-Controlled (CC):**
- Any unit labeled: "climate controlled," "climate," "heated/cooled," "interior," "temp controlled"
- Indoor corridor access generally = CC even if not labeled (use judgment + rate level as signal)
- "Heated only" = treat as CC (note in cell)

**Conventional (CONV):**
- Drive-up, outdoor access, non-climate, "standard," "unconditioned"
- Ground level or upper floor without climate control

**Parking / Vehicle (PARK):**
- Uncovered parking spaces
- Covered/enclosed vehicle storage
- RV / boat storage
- Auto storage bays
- Record these separately from unit storage — do not blend with CONV

**Other:**
- Wine storage, document storage, lockers under 5x5
- Unusual unit types that don't cleanly fit above
- Use sparingly — most units are CC or CONV

---

## Canonical Size Grid

When a competitor lists a non-standard size, map it to the nearest canonical size using the
rules below. Always note the actual size in the cell tooltip/notes.

| Canonical Size | Captures These Actual Sizes |
|---|---|
| 5x5 | 4x5, 5x5, 5x4, 4x4 |
| 5x10 | 5x8, 5x9, 5x10, 5x11, 6x10 |
| 10x10 | 9x10, 10x9, 10x10, 10x11, 10x12 |
| 10x15 | 10x13, 10x14, 10x15, 10x16 |
| 10x20 | 10x18, 10x19, 10x20, 10x21, 12x20 |
| 10x30 | 10x25, 10x28, 10x30, 10x35, 12x30 |
| Parking/Vehicle | Any size parking space, RV pad, covered stall |

If a competitor offers, e.g., a 10x25 but no 10x30, map it to 10x30 and note "(10x25)" in
the cell. Do not leave the 10x30 row empty when a close substitute exists.

---

## Matrix Format

### Column Headers
```
| Unit | ★ [Subject Name] | [Comp 1 Short] ([dist]·[Type]) | [Comp 2 Short] ... |
```
- Subject always first with ★ prefix
- Comp short name: abbreviate to ≤ 15 chars (e.g., "EXR Clayton Blvd", "Pub Stor Veterans")
- Distance: e.g., "2.1mi"
- Type: REIT / Reg / Ind

### Row Order (always in this sequence)
```
── CLIMATE-CONTROLLED ──
CC 5x5
CC 5x10
CC 10x10
CC 10x15
CC 10x20
CC 10x30
── CONVENTIONAL ──
Conv 5x5
Conv 5x10
Conv 10x10
Conv 10x15
Conv 10x20
Conv 10x30
── PARKING / VEHICLE ──
Parking (uncov)
Parking (covered)
RV / Boat / Auto
── OTHER ──
[Only include rows where at least 1 facility has this product]
```

Omit entire category sections if **no facility** in the comp set offers that type.

### Cell Format

Single rate: `$95`  
Street + promo: `$95 / $45★` (where ★ = promo, e.g. 50% off or $1 1st mo — explain in notes row)  
WoW up: `$95 ▲` (was $81)  
WoW down: `$65 ▼` (was $73)  
Not offered / not found: `—`  
New this week: `$95 ★new`

### Notes Row (per category section)
At the bottom of each category block, add a Notes row:
```
| Notes | [subject notes] | [comp1 promo/occupancy signal] | ... |
```
Keep each cell brief: "Full, no avail" / "$1 1st mo promo" / "Waitlist 10x10" / "—"

---

## Rate-Per-SF Supplement (optional)

If the user requests it or if there are notable size distortions (e.g., a comp's "10x10"
is actually 10x12), append a Rate/SF table beneath the main matrix using the same structure
but cells showing `$X.XX/sf` instead of monthly rates.

Formula: `monthly_rate / (width × depth)`