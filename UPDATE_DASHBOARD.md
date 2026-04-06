# Oil dashboard update instructions

You are updating `americas_oil_situation_dashboard.html` — an interactive dashboard tracking U.S. crude oil supply, stockpiles, imports, Strait of Hormuz status, and days-to-empty projections.

**All data lives in a single `DATA` object** at the top of the `<script>` block. You will only ever edit values inside that object. Do not touch the HTML structure, CSS, or the rendering logic below the `END DATA CONFIG` comment.

---

## How the dashboard works

The `DATA` object feeds a `renderDashboard()` function that propagates every value into the page via element IDs. When you change a number in `DATA`, the page automatically reflects it everywhere — metric cards, supply flow bars, chart projections, and computed fields like "Total available" and "Days to critical."

The DTE (days to empty) chart uses `DATA.commercialCrude` and `DATA.effectiveSpr` as starting inventory, then draws them down over 36 months using the deficit rate from each scenario. You do not need to manually compute DTE — just update the inventory numbers and scenario parameters and the math handles itself.

---

## Step-by-step update process

### 1. Gather the data

Search the web for each of the following. Sources are listed in priority order — use the first one that returns current data.

#### Stockpiles (updates weekly, usually Wednesday)

| Field | What to search | Primary source |
|---|---|---|
| `commercialCrude` | "EIA weekly crude oil inventories" | eia.gov/petroleum/supply/weekly |
| `commercialCrudeChange` | Same report — weekly change | Same |
| `gasolineStocks` | Same report — motor gasoline inventories | Same |
| `gasolineChange` | Same report — weekly change | Same |
| `distillateStocks` | Same report — distillate fuel inventories | Same |
| `cushingChange` | Same report — Cushing OK stocks change | Same |
| `eiaWeekEnding` | The "week ending" date on the report | Same |

Also check tradingeconomics.com/united-states/crude-oil-stocks-change as a backup — they publish the same EIA data in a more readable format.

#### SPR levels

| Field | What to search | Primary source |
|---|---|---|
| `sprPreRelease` | "US strategic petroleum reserve current level" | energy.gov/hgeo/opr/spr-quick-facts |
| `sprCommittedRelease` | "IEA coordinated oil release 2026" | Only update if the committed amount changes |
| `effectiveSpr` | Computed: `sprPreRelease - sprCommittedRelease` | You calculate this |

The SPR level changes slowly (weekly or less). If you can't find a newer number than what's already in the file, leave it.

#### Supply flows

| Field | What to search | Primary source |
|---|---|---|
| `domesticProduction` | "US crude oil production barrels per day" | EIA Short-Term Energy Outlook (eia.gov/outlooks/steo) |
| `canadaPipeline` | "US crude oil imports from Canada" | EIA Petroleum Supply Monthly or tradingeconomics |
| `mexicoMaritime` | "US crude oil imports from Mexico" | Same |
| `atlanticBasin` | Total non-Canada, non-Mexico, non-Gulf imports | Computed from EIA total imports minus known sources |
| `persianGulf` | "Strait of Hormuz tanker transits today" | hormuzstraitmonitor.com, UANI blog, or CNBC/Bloomberg reporting |

Production and pipeline data move slowly (monthly). Maritime and Hormuz data can shift daily.

#### Supply flow status indicators

Set each status field to one of three values based on your judgment:

- `"green"` — flow is operating normally, no disruption
- `"yellow"` — flow is reduced, at risk, or under policy pressure (tariffs, sanctions, rerouting)
- `"red"` — flow is effectively halted or critically impaired

#### Hormuz situation

| Field | What to search | Primary source |
|---|---|---|
| `hormuzCurrentTransits` | "Strait of Hormuz ship transits today" | hormuzstraitmonitor.com, Windward maritime intelligence, UANI |
| `strandedVLCCs` | "stranded tankers Persian Gulf" | Lloyd's List, CNBC shipping tracker |
| `attacksOnShips` | "attacks on ships Strait of Hormuz 2026" | Wikipedia (2026 Strait of Hormuz crisis), IMO reports, UANI |

If the Hormuz crisis has resolved (strait reopened, ceasefire, etc.), update the status fields and descriptive notes accordingly. The dashboard should reflect reality, not a frozen moment.

#### Prices

| Field | What to search | Primary source |
|---|---|---|
| `brentCurrent` | "Brent crude oil price today" | Any financial data source — tradingeconomics, investing.com, Bloomberg |
| `gasPrice` | "US average gas price today" | AAA gas prices (gasprices.aaa.com) or EIA |
| `eiaForecastPrice` | "EIA Brent crude oil price forecast" | EIA Short-Term Energy Outlook |
| `eiaForecastPeriod` | Which quarter the forecast targets | Same |
| `brentCurrentNote` | Compute: percentage change from `brentPreWar` | You calculate this |

#### Consumption

| Field | What to search | Primary source |
|---|---|---|
| `dailyConsumption` | "US petroleum consumption barrels per day" | EIA — usually ~20 mb/d, updates quarterly |

### 2. Update the DATA object

Open `americas_oil_situation_dashboard.html` and locate the `DATA` object between these two comment markers:

```
/* ============================================================
   DATA CONFIG — AGENT: UPDATE THIS BLOCK ONLY
```

and

```
/* ============================================================
   END DATA CONFIG — DO NOT EDIT BELOW THIS LINE
```

Update each field with the new values you found. Follow these formatting rules:

- **Numbers** are plain numbers, no quotes: `commercialCrude: 465.2`
- **Strings** use double quotes: `commercialCrudeChange: "+3.6M last week"`
- **The `updated` field** must reflect today's date: `updated: "April 7, 2026"`
- **The `eiaWeekEnding` field** must reflect the EIA report date, not today
- **`effectiveSpr`** must equal `sprPreRelease - sprCommittedRelease`

### 3. Update the scenario parameters

The three DTE scenarios need to stay internally consistent. Here's the logic:

**Base case** — All current domestic and pipeline supply remains intact. No Hormuz oil. Set `consumption` to the current EIA consumption figure. Set `supply` to `domesticProduction + canadaPipeline + mexicoMaritime + atlanticBasin`. Set `deficit` to `consumption - supply`. Update `label` with a human-readable estimate (e.g., "~3 years at current draw").

**Moderate** — Assume 15% reduction in Canadian pipeline flow and 30% drop in Atlantic Basin imports. Supply = `domesticProduction + (canadaPipeline * 0.85) + mexicoMaritime + (atlanticBasin * 0.7)`. Consumption drops slightly due to demand destruction. Recalculate deficit and label.

**Severe** — Assume 50% Canadian pipeline reduction and zero maritime imports. Supply = `domesticProduction + (canadaPipeline * 0.5)`. Consumption drops further due to emergency measures. Recalculate deficit and label.

For the label, compute: `(commercialCrude + effectiveSpr) / deficit` = days. Convert to months or years for readability.

### 4. Update the sources line

Update `DATA.sources` to reflect the actual reports you pulled from and their dates. Format: `"Source Name (date), Source Name (date), ..."`.

### 5. Pre-commit validation (you do this BEFORE saving)

The dashboard has a built-in validation system that runs automatically when the page loads. But you should catch problems *before* they get into the file. Run through this checklist after editing the DATA object:

#### Range checks

Every number must fall within these bounds. If a value you found is outside these ranges, **stop and flag it for the human** — don't just write it in.

| Field | Min | Max | Why |
|---|---|---|---|
| `commercialCrude` | 100 | 700 | US commercial crude has never been outside this range. A value like 46.1 instead of 461 is a decimal error. |
| `sprPreRelease` | 50 | 714 | 714M is physical capacity. The all-time low was ~347M in 2023. |
| `sprCommittedRelease` | 0 | 500 | Cannot release more than exists. |
| `effectiveSpr` | 0 | 714 | Cannot go negative or exceed capacity. |
| `gasolineStocks` | 50 | 400 | Historical range. |
| `distillateStocks` | 30 | 250 | Historical range. |
| `domesticProduction` | 8 | 18 | US has never produced outside this band. Current era is 12–14 mb/d. |
| `canadaPipeline` | 1 | 7 | Pipeline-constrained. |
| `mexicoMaritime` | 0 | 3 | Small relative to total. |
| `atlanticBasin` | 0 | 5 | Catch-all for non-Canada, non-Mexico, non-Gulf. |
| `persianGulf` | 0 | 5 | Zero during Hormuz closure, ~1-2 mb/d normally. |
| `brentCurrent` | 20 | 300 | Oil has never traded outside this range. |
| `dailyConsumption` | 14 | 25 | US consumption is structurally in this band. |
| `strandedVLCCs` | 0 | 800 | Global VLCC fleet is ~800 ships. |

#### Logical consistency checks

Run these mentally before saving:

1. **SPR math**: `effectiveSpr` must equal `sprPreRelease - sprCommittedRelease`. If it doesn't, something is wrong.

2. **Scenario deficit math**: For each scenario, `deficit` must equal `consumption - supply`. Recalculate by hand.

3. **Scenario ordering**: Deficit must increase from base → moderate → severe. Consumption should decrease or stay flat (demand destruction). If moderate is worse than severe, the scenarios are swapped.

4. **Supply vs consumption**: `domesticProduction + canadaPipeline + mexicoMaritime + atlanticBasin + persianGulf` should be close to `dailyConsumption` (within ~2 mb/d). If total supply wildly exceeds consumption, double-check import figures.

5. **Price direction**: During the Hormuz crisis, `brentCurrent` should be above `brentPreWar`. If it's dropped significantly below pre-war levels, either the crisis has de-escalated (update the notes to reflect this) or the number is wrong.

6. **Week-over-week changes**: `commercialCrudeChange`, `gasolineChange`, and `cushingChange` should be directionally consistent with the inventory numbers. If commercial crude went up by 5M but you wrote the change as negative, something's off.

7. **DTE labels**: After computing `(commercialCrude + effectiveSpr) / deficit` for each scenario, make sure the `label` string matches the math. Don't write "~3 years" if the math says 18 months.

#### Stop conditions

**Do not commit if any of these are true. Ask the human instead:**

- Any number is outside the range bounds above
- You couldn't find a source for a data point and had to guess
- A value changed by more than 30% week-over-week (possible data error or major event)
- The EIA report date hasn't changed since the last update (no new data to incorporate)
- The computed DTE for the base case is less than 90 days (this would be a national emergency — verify independently)
- The Hormuz crisis status has fundamentally changed (reopened, new closure, ceasefire) — the dashboard categories may need restructuring

### 6. Built-in dashboard validation

The dashboard runs its own `validateData()` function on page load. After saving your changes, open the HTML file in a browser. If you see a red or orange banner at the top of the page, the validation caught a problem. The banner will list specific errors (red) and warnings (orange) with field names and expected values.

- **Red errors** = hard failures. The data is logically broken. Do not commit until all red items are resolved.
- **Orange warnings** = soft checks. The data is technically valid but unusual. Review each one and confirm the value is genuinely correct before committing.

If the page loads with no banner, validation passed.

### 7. Commit

Only after steps 5 and 6 pass cleanly:

```bash
git add americas_oil_situation_dashboard.html
git commit -m "oil dashboard update: [today's date] — EIA week ending [date]"
git push
```

If you had to skip any data points because sources weren't available, note it in the commit message:

```bash
git commit -m "oil dashboard update: April 9, 2026 — EIA week ending April 3, 2026 — Hormuz transit count unverified, using prior value"
```

---

## What NOT to change

- Any HTML structure (tags, classes, IDs)
- Any CSS
- Any JavaScript below the `END DATA CONFIG` comment
- The Chart.js CDN link
- Color values in the flow bars (these are fixed by design)

If the Hormuz crisis ends or a major structural change happens (new supply route, new sanctions, SPR policy change), flag it for the human operator to decide whether the dashboard categories themselves need revision. Don't restructure the dashboard autonomously.

---

## Field reference

Here is every field in the DATA object with its type and unit:

```
updated                 string    "Month Day, Year"
eiaWeekEnding           string    "Month Day, Year"
sources                 string    comma-separated source list

commercialCrude         number    millions of barrels
commercialCrudeChange   string    e.g. "+5.5M last week"
sprPreRelease           number    millions of barrels
sprCapacity             number    millions of barrels (fixed at 714)
sprCommittedRelease     number    millions of barrels
sprCommittedNote        string    description of release type
effectiveSpr            number    millions of barrels (computed)
gasolineStocks          number    millions of barrels
gasolineChange          string    e.g. "-0.6M last week"
distillateStocks        number    millions of barrels
distillateNote          string    product description
cushingChange           string    e.g. "+520K"
cushingNote             string    unit note

domesticProduction      number    mb/d
canadaPipeline          number    mb/d
mexicoMaritime          number    mb/d
atlanticBasin           number    mb/d
persianGulf             number    mb/d
sprMaxDrawdown          number    mb/d (fixed at 4.4)

domesticStatus          string    "green" | "yellow" | "red"
canadaStatus            string    "green" | "yellow" | "red"
mexicoStatus            string    "green" | "yellow" | "red"
atlanticStatus          string    "green" | "yellow" | "red"
persianGulfStatus       string    "green" | "yellow" | "red"
sprDrawdownStatus       string    "green" | "yellow" | "red"

hormuzPreWarTransits    string    pre-crisis baseline
hormuzCurrentTransits   string    current daily count
hormuzTransitNote       string    context note
strandedVLCCs           number    count
strandedVLCCNote        string    context note
attacksOnShips          string    cumulative count
attacksSince            string    timeframe note

brentPreWar             number    $/barrel (fixed baseline)
brentPreWarDate         string    "Month Day, Year" (fixed)
brentCurrent            number    $/barrel
brentCurrentNote        string    change context
gasPrice                string    $/gallon (use "4+" style for approximations)
gasPriceNote            string    change context
eiaForecastPrice        number    $/barrel
eiaForecastNote         string    condition note
eiaForecastPeriod       string    "Q1" | "Q2" | "Q3" | "Q4"

dailyConsumption        number    mb/d

scenarios.{base|moderate|severe}:
  desc                  string    scenario description
  consumption           number    mb/d
  supply                number    mb/d
  deficit               number    mb/d (consumption - supply)
  label                 string    human-readable DTE estimate
```
