# Oil dashboard update instructions

You are updating `index.html` (the dashboard page) — an interactive dashboard tracking U.S. crude oil supply, stockpiles, imports, Strait of Hormuz status, and days-to-empty projections.

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

#### Shortage propagation (`shortageFlow` array)

The "Shortage propagation" section shows a four-stage chain: Gulf-dependent buyers → Atlantic Basin bidding → European refiners → U.S. impact. Update each stage's `status` (`"green"`/`"yellow"`/`"red"`), `metric` (short headline number), and `note` (one-sentence plain-language summary).

| Stage (typical) | What to search | Primary sources |
|---|---|---|
| Gulf-dependent Asian refiners | "Japan refinery run cuts", "South Korea crude imports", "India crude waivers" | Reuters energy, Energy Intelligence, S&P Platts |
| Atlantic Basin bidding war | "Asian refiners outbidding Europe for WTI", "non-Middle East crude competition" | Reuters, Kpler, Argus, IEA OMR |
| European refiners squeezed | "ARA oil stocks weekly", "Northwest Europe refining margins", "gasoil crack" | Argus Media (Insights Global), Oxford Energy, IEA OMR |
| U.S. impact window | "US Atlantic Basin crude imports", "Gulf Coast refinery crude", AAA gas prices | EIA PSM, Reuters, AAA |

Keep `note` under ~35 words per stage. The chain renders from top-to-bottom in the order listed in the array — stage 1 is the furthest from the U.S., stage 4 is the closest. When the crisis eases, downgrade statuses from red → yellow → green from the top down (the pressure releases from the source first).

#### Shortage map (`shortageMap` object)

The world-map companion to the shortage-propagation chain. It shades countries by pre-crisis Persian Gulf / Hormuz reliance and draws a simple arc from the Gulf to each country's centroid. Keep the data list small and curated (10–15 countries) — the signal dies if you try to cover everyone.

> **⚠️ DO NOT FABRICATE VESSEL-LEVEL DETAIL.** Free open-source news rarely resolves a named tanker arriving at a named port on a named date. That is fine. The default for every `lastCargo` entry is `status: "unknown"` with `loadDate: null` and `arrivalDate: null`. Only set `status` to `"landed"` / `"in-transit"` / `"diverted"` when a named, dated public-press article confirms the cargo — and cite that article in `lastCargo.source` (publisher + ISO date, e.g. `"Reuters, 2026-03-19"`). A credibility-free map is worse than a map with honest gaps. The page validator will flag any non-`unknown` cargo that lacks a plausible citation.
>
> **Concrete rules:**
> - Never invent a vessel name, IMO number, load date, or arrival date that is not in a named public source.
> - Do not copy schema examples from this doc or from the data block as if they were real shipments. If you cannot find a real cited cargo, leave `status: "unknown"` and dates `null`.
> - `lastCargo.source` must read like a citation ("Reuters, 2026-03-19", "CNBC, 2026-04-17"), not like a routing description ("typical AG routing"). Routing descriptions are only acceptable when `status: "unknown"`.
> - If in doubt, downgrade to `"unknown"`. That is always a safe edit.

**Only use free, open data.** No Kpler / Vortexa / ClipperData / paid Platts assessments. Primary sources, all free:

| Field | What to search | Primary sources |
|---|---|---|
| `hormuzSharePct` | "\<country\> crude oil imports by origin", "Middle East share of \<country\> oil imports" | JODI-Oil (joint-org CSVs), EIA International Energy Data / Country Analysis Briefs, Energy Institute Statistical Review, national regulators (METI, KNOC, PPAC, Philippines DOE, DESNZ) |
| `daysOfCover` | "\<country\> strategic petroleum reserve days", "\<country\> crude stocks cover" | IEA Monthly Oil Data Service (OECD), national stockpile reporters, Reuters explainer pieces. When no hard number exists, pick a conservative middle estimate and note the uncertainty in `notes`. |
| `lastCargo` | Reuters / CNBC / Argus / Al Jazeera vessel-level reporting | If free sources don't resolve a named vessel, set `status: "unknown"` and leave dates null. **Do not fabricate** load/arrival dates or vessel names. Typical loadings: Ras Tanura (SA), Juaymah (SA), Jubail (SA), Mina Al Ahmadi (KW), Basrah (IQ), Kharg (IR). |
| `notes` | — | ≤30 words, measured tone matching `shortageFlow` style. Call out uncertainty explicitly when it's material. |
| `sources` | — | 2–4 short citations (e.g. "Reuters — Japan extra reserve release (2026-04-10)"). |

**ISO codes:** `iso3` must be a real ISO 3166-1 alpha-3 code. It's the join key to the world-atlas TopoJSON. Also update `SHORTAGE_MAP_ISO_NUM_TO_ALPHA3` in `index.html` if you add a country not already mapped there (look up the ISO 3166-1 numeric code — e.g. Japan is `392`). Very small countries that don't appear at the 110m world-atlas resolution (like Singapore) also need an entry in `SHORTAGE_MAP_FALLBACK_CENTROIDS` so they still get an arc and a dot.

**Cargo status vocabulary:**
- `"landed"` — last Gulf-origin cargo has been confirmed discharged at the destination port in free press.
- `"in-transit"` — a named cargo is at sea, widely reported, not yet discharged.
- `"diverted"` — the cargo was rerouted to a different buyer or port.
- `"unknown"` — free sources don't resolve it. This is the **honest default** and the most common state.

**Pattern for when the picture changes:** when a new "final shipment landed" story runs (e.g. Reuters confirms the last Hormuz VLCC discharged at Batangas), flip that country's `lastCargo.status` to `"landed"` and backfill `arrivalDate`. When things ease, gradually downgrade `hormuzSharePct` exposure color only if there's real evidence of re-routing; otherwise keep the structural number and move the story in the chain section above.

#### Paper vs. physical price (`priceDisparity` object)

The "Paper vs. physical oil price" section compares screen futures to actual cargo prices and shows the backwardation as a tightness gauge.

| Field | What to search | Primary source |
|---|---|---|
| `paperBrent` | "Brent crude oil price today" (front-month futures) | ICE / tradingeconomics / Bloomberg |
| `physicalEstimate` | "Dated Brent price", "Dated Brent vs futures spread" | Platts assessments via S&P Global, Argus, CNBC/Reuters reporting (usually behind paywall — rely on press quoting the Platts number) |
| `spread` | Computed: `physicalEstimate - paperBrent` | You calculate this |
| `m1m3Spread` | "Brent M1 M3 spread", "Brent front-month backwardation" | ICE margin rate PDFs, CME Group insights, tradingview, IEA OMR |
| `backwardationStatus` | Judgment based on `m1m3Spread` magnitude | Typical: $0–$2 green, $2–$10 yellow, $10+ red |

If Dated Brent is not quoted anywhere free, mark `physicalNote` with a clear "estimate from press reporting, date X" so readers know it's not a live number.

#### Estimate transparency (`estimatedFields` + `estimates`)

Any value on the dashboard that is not a direct read of a primary source should be flagged as an estimate. The page shows a small `est.` pill next to flagged values and an expandable "How these … are estimated" block in each affected section.

**`estimatedFields`** — which rendered cells get the pill:

```js
estimatedFields: {
  values: ["v-hormuz-current", "v-vlccs", "v-attacks",
           "v-disp-physical", "v-disp-spread", "v-disp-m1m3"],
  flows:  ["flow-atlantic", "flow-gulf", "flow-spr"]
}
```

- `values` — DOM IDs of `<div class="value">` elements (on metric cards).
- `flows` — DOM IDs of `<div class="flow-row">` rows (pill goes on the `.flow-val` span).

If you introduce a new estimated value, add the ID here and add a matching item under `estimates`.

**`estimates`** — methodology blocks, keyed by section:

```js
estimates: {
  supplyFlows:    { title, items: [{ label, note }, ...] },
  hormuz:         { title, items: [...] },
  priceDisparity: { title, items: [...] }
}
```

- `title` — section heading shown in the collapsed `<details>` summary (e.g. "How these flows are estimated").
- `items[].label` — short descriptor including the rendered value (e.g. "Physical (Dated Brent) ~$130").
- `items[].note` — one or two sentences explaining the derivation, naming the source and date where possible. Readers should be able to answer "how do we know this number and how recent is it?" from the note alone.

**Rules of thumb for what qualifies as an estimate:**

- Computed residuals (e.g. `atlanticBasin` = EIA total minus named flows).
- Statutory capabilities surfaced as if they were flows (e.g. `sprMaxDrawdown`).
- Values derived from press reporting of paywalled assessments (e.g. Dated Brent).
- Tallies compiled from news outlets without an official cumulative authority (e.g. attacks on ships).
- Ranges instead of point values (e.g. `~5–10` transits).

Hard numbers direct from EIA, DOE, AAA, ICE, or equivalent primary sources do **not** need the marker.

### 2. Update the DATA object

Open `index.html` and locate the `DATA` object between these two comment markers:

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

### 2b. Metric history (click-to-chart)

The file **`metric-history.json`** (next to `index.html`) stores **time series** for dashboard metrics. When readers click a highlighted card or flow row, the page loads this file and draws a **Chart.js** line of past values.

**On each data refresh**, append a new point to **every series you changed** (same keys as `data-metric` in `index.html`). Each point looks like:

```json
{ "t": "2026-04-03", "v": 464.7, "compile": "April 8, 2026", "eiaWeekEnding": "April 3, 2026" }
```

- **`t`** — ISO date; prefer the **EIA week-ending date** for weekly stocks.
- **`v`** — number in the same units as `DATA` (e.g. commercial crude in **M bbl**, Cushing change in **thousands of barrels** for `cushingChangeKbbl`, gas in **`gasPriceUsd`** as dollars per gallon).
- **`compile`** / **`eiaWeekEnding`** — optional; shown in the chart tooltip.

If a series is missing or empty, the modal explains that no history is recorded yet. **Serving over `http://`/`https://`** is required so the browser can `fetch('metric-history.json')`; opening `index.html` alone as `file://` will not load the file.

### 3. Update the scenario parameters

The three DTE scenarios need to stay internally consistent. Here's the logic:

**Base case** — All current domestic and pipeline supply remains intact. No Hormuz oil. Set `consumption` to the current EIA consumption figure. Set `supply` to `domesticProduction + canadaPipeline + mexicoMaritime + atlanticBasin`. Set `deficit` to `consumption - supply`. Update `label` with a human-readable estimate (e.g., "~3 years at current draw").

**Moderate** — Assume 15% reduction in Canadian pipeline flow and 30% drop in Atlantic Basin imports. Supply = `domesticProduction + (canadaPipeline * 0.85) + mexicoMaritime + (atlanticBasin * 0.7)`. Consumption drops slightly due to demand destruction. Recalculate deficit and label.

**Severe** — Assume 50% Canadian pipeline reduction and zero maritime imports. Supply = `domesticProduction + (canadaPipeline * 0.5)`. Consumption drops further due to emergency measures. Recalculate deficit and label.

For the label, compute: `(commercialCrude + effectiveSpr) / deficit` = days. Convert to months or years for readability.

### 4. Sources footnote (`sourcesIntro` + `sourceLinks`)

**`DATA.sourcesIntro`** — Short paragraph (one to three sentences) naming **which report dates / compile** this dashboard snapshot uses (e.g. EIA WPSR release date, STEO vintage). No comma-separated wall of names; that detail lives in the list below.

**`DATA.sourceLinks`** — Ordered array of `{ label, url, description }`:

- **`label`** — Short name of the outlet or report (shown as the link text).
- **`url`** — Must start with `https://` or `http://`.
- **`description`** — One or two sentences on **what this source offers** and how the dashboard uses it (e.g. “Official weekly U.S. crude and product stocks…”).

These render as **one unified list** under **Sources** in `#dash-sources`: intro paragraph, then each item = **clickable title** + description beneath. Official sources first (EIA, DOE, IEA); then trackers and press.

When you add a new outlet to your research set, append a `sourceLinks` row with a real description—do not add a bare link with no explanation.

### 4c. Update Log (data narrative)

The **Update Log** is the slide-out panel from the title bar (`Update Log` button). It is **not** for technical or deploy notes (Netlify, file renames, bugfixes). It is for a **short reporter-style summary** of **substantive changes to the dashboard’s information** after you update `DATA`—so readers can see “what moved” in plain language.

**Who maintains it:** Anyone changing `DATA` (human or AI) should add a log entry as part of the same update.

**Where to edit:** In [index.html](index.html), find `<aside class="update-log-panel" id="update-log-panel">`. Inside `.update-log-panel-inner`, add a **new** `<div class="update-log-entry">` block at the **top** (newest first), above older entries. Keep at least one older entry if you want history, or trim very old blocks so the panel stays readable.

**Entry format (copy this structure):**

```html
<div class="update-log-entry">
  <div class="date">[Month Day, Year] — data refresh</div>
  <ul>
    <li><strong>EIA (week ending [date]):</strong> One sentence: biggest commercial crude / products / Cushing moves vs last week.</li>
    <li><strong>Balances / scenarios (if changed):</strong> One sentence on supply vs consumption or scenario assumptions.</li>
    <li><strong>Hormuz / maritime (if changed):</strong> One sentence; say if figures are estimates.</li>
    <li><strong>Prices (if changed):</strong> Brent / gas direction vs prior snapshot.</li>
  </ul>
</div>
```

**Reporter style (keep it small):**

- **2–4 bullets** per update, not paragraphs. Lead with the **EIA week** when stock data changed.
- Use **active voice** and **numbers** where they appear in `DATA` (e.g. “Commercial crude +3.2M bbl to 465M”).
- Do **not** paste the whole `sourcesIntro` or source list; say “per EIA WPSR” if needed.
- If only one section changed (e.g. Hormuz only), one focused bullet is enough.

**AI / agent checklist when updating `DATA`:**

1. After all `DATA` fields and `sourceLinks` / `sourcesIntro` are updated, **open the Update Log HTML** and **prepend** a new `update-log-entry` as above.
2. Use `updated` and `eiaWeekEnding` from `DATA` in the entry so the log matches the file.
3. Summarize **only** what changed in this pass; skip “no change” sections or say “unchanged from prior week” in one clause.
4. Do **not** log implementation details (validation fixes, CSS, hosting).

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
| `priceDisparity.paperBrent` | 20 | 300 | Front-month Brent futures — same bounds as `brentCurrent`. |
| `priceDisparity.physicalEstimate` | 20 | 400 | Dated Brent can spike well above futures in a crisis (peaked $144 in Apr 2026). |
| `priceDisparity.spread` | -20 | 120 | Physical premium over paper. Negative = contango (physical below futures). |
| `priceDisparity.m1m3Spread` | -20 | 60 | Brent M1–M3 futures spread; positive = backwardation. |

#### Logical consistency checks

Run these mentally before saving:

1. **SPR math**: `effectiveSpr` must equal `sprPreRelease - sprCommittedRelease`. If it doesn't, something is wrong.

   **Paper/physical spread math**: `priceDisparity.spread` must equal `physicalEstimate - paperBrent` (within ±$2/bbl tolerance). If it doesn't, recompute.

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

Only after steps 5 and 6 pass cleanly (and §4c Update Log entry is added if this was a substantive data refresh):

```bash
git add index.html metric-history.json
git commit -m "oil dashboard update: [today's date] — EIA week ending [date]"
git push
```

If you had to skip any data points because sources weren't available, note it in the commit message:

```bash
git commit -m "oil dashboard update: April 9, 2026 — EIA week ending April 3, 2026 — Hormuz transit count unverified, using prior value"
```

---

## What NOT to change

- Any HTML structure (tags, classes, IDs)—**except** appending or editing **Update Log** entries inside `#update-log-panel` as described in §4c when refreshing data.
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
sourcesIntro            string    short paragraph: report dates / compile context for this snapshot
sourceLinks             array     [{ label, url, description }, ...]  unified source list with what each offers

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

shortageFlow            array     [{ stage, status, metric, note }, ...]
  stage                 string    short name of the propagation stage
  status                string    "green" | "yellow" | "red"
  metric                string    short headline number or phrase (e.g. "Japan at 67.8% run")
  note                  string    one-sentence plain-language explainer (~35 words)

shortageMap             object    world-map view of country-level Hormuz exposure
  asOf                  string    snapshot date shown in the sub-header (e.g. "April 21, 2026")
  title                 string    section heading (optional; falls back to the static HTML)
  description           string    one-sentence caption (optional)
  countries             array     [{ iso3, name, hormuzSharePct, daysOfCover, lastCargo, notes, sources }, ...]
    iso3                string    ISO 3166-1 alpha-3 (3 letters, joins to the world-atlas TopoJSON)
    name                string    human-readable country name
    hormuzSharePct      number    0–100, pre-crisis % of crude imports routed via the Strait of Hormuz
    daysOfCover         number    0–400, est. days before physical shortage bites (stocks + non-Gulf swaps)
    lastCargo           object    last known Gulf-origin shipment
      loadPort          string    Gulf loading port (Ras Tanura / Juaymah / Jubail / Basrah / …)
      dischargePort     string    destination port
      loadDate          string?   ISO date or null if free sources don't resolve it
      arrivalDate       string?   ISO date or null
      status            string    "landed" | "in-transit" | "diverted" | "unknown"
      barrels           number    cargo size (typical VLCC ~2,000,000; Suezmax ~1,000,000)
      source            string    short citation
    notes               string    ≤30-word plain-English summary of exposure + current posture
    sources             string[]  2–4 short open-source citations

priceDisparity          object
  paperBrent            number    $/bbl — ICE Brent front-month futures
  paperNote             string    which contract / when
  physicalEstimate      number    $/bbl — Dated Brent or press-reported physical cargo assessment
  physicalNote          string    source and date of the assessment
  spread                number    $/bbl — physicalEstimate - paperBrent (positive = physical premium)
  spreadNote            string    context (peak, recent range)
  m1m3Spread            number    $/bbl — Brent front month minus 3-months-out (positive = backwardation)
  m1m3Note              string    short context
  backwardationStatus   string    "green" | "yellow" | "red" (judgment)

estimatedFields         object    which rendered cells get the "est." pill
  values                string[]  DOM IDs of .metric .value elements
  flows                 string[]  DOM IDs of .flow-row elements (pill goes on .flow-val)

estimates               object    per-section methodology blocks
  <section>             object    keyed by supplyFlows | hormuz | priceDisparity | ...
    title               string    summary shown when <details> is collapsed
    items               array     [{ label, note }, ...]
      label             string    short descriptor naming the estimated value
      note              string    one or two sentences: source, date, derivation, confidence
```
