# Export Landscape Explorer

A public-facing data tool that lets users discover where Singapore firms matching their business profile export to, how concentrated those export flows are, and how destinations have trended over the past decade.

Built for the Department of Statistics Singapore (DOS) as a companion to the Trade Compass suite.

---

## What it does

Given four business characteristics — SSIC division, revenue band, employment size, and firm age — the tool aggregates customs and firm-registry data across all matching firms and returns:

- A **summary dashboard** of firm count, total exports, number of markets reached, and an HHI concentration score
- A **donut chart** of top export destinations by share of total exports, filterable by region
- A **ranked list** of destinations with flags, FTA tags, values, and shares
- A **10-year trend line** for any destination the user clicks on, with CAGR, peak year, and FTA status

The aim is to help SMEs, trade associations, policy analysts, and curious members of the public answer the question: _"Where do firms like mine sell abroad?"_

---

## Inputs

| Field                       | Values                                                                       | Required |
| --------------------------- | ---------------------------------------------------------------------------- | -------- |
| SSIC2025 (2-digit division) | 23 divisions covering manufacturing, trade, transport, professional services | Yes      |
| Revenue band                | Below $1M / $1M–$10M / $10M–$100M / $100M–$500M / Above $500M                | Yes      |
| Employment size             | 1–9 / 10–49 / 50–199 / 200–499 / 500+                                        | Yes      |
| Firm age                    | <2 yrs / 2–5 / 5–10 / 10–20 / 20+                                            | Yes      |

The free-text _Description of Primary Activity_ field from the Trade Explorer tool is intentionally excluded — this tool works on structured characteristics only, which keeps results deterministic and reproducible.

---

## Outputs

### KPI cards

- **Matching firms** — total count of firms in the registry matching the selected profile, with the number and percentage that export
- **Total exports (2024)** — aggregate 2024 export value in SGD
- **Export markets** — number of distinct destinations reached
- **HHI** — Herfindahl-Hirschman Index (0–10,000) of destination concentration, with a hover tooltip explaining the metric

### Concentration bar

A gradient track (green → amber → red) with a marker showing where the profile's HHI falls, and a badge labelling it **Diversified** (<1,500), **Moderate** (1,500–2,500), or **Concentrated** (>2,500).

### Destination donut + ranked list

Side-by-side donut chart and ranked list. Click either a slice or a list row to drill into a country's trend. Region filter pills (All / ASEAN / Asia-Pacific / Americas / Europe / Others) let users slice the view geographically; the donut, list, and summary all recalculate from the filtered subset. Top 8 countries are shown individually; the remainder is bundled as "Others".

### 10-year trend line

Appears after a country is selected. Shows exports (in SGD) from 2015 to 2024, with four context stats:

- **2024 Exports** — latest-year value
- **10-yr CAGR** — compound annual growth rate, coloured green (positive) or red (negative)
- **Peak year** — year of highest exports in the decade
- **FTA status** — which Singapore FTA covers the destination, if any

On mobile, selecting a country auto-scrolls the trend chart into view.

---

## Design principles

1. **Visual parity with Trade Compass** — identical masthead, header, hero, breadcrumb, form panel, and results panel to feel native within the DOS tool suite
2. **Mobile-first considerations** — the tool-layout collapses to a single column below 960px; KPI grid reflows to 2×2; the market list becomes scrollable with a fixed height so the trend chart stays reachable
3. **Make HHI intuitive** — most users don't know what HHI is, so it's paired with a concentration bar and a plain-English label (Diversified/Moderate/Concentrated) and an info tooltip
4. **Deterministic** — the same four inputs always yield the same output, so users can share and revisit a result with confidence
5. **Confidentiality-aware** — the in-form disclaimer notes that small-cell values are suppressed, matching DOS practice for aggregated firm-level data

---

## Technical details

### Stack

- **Single HTML file**, ~66 KB, no build step
- **Chart.js 4.4.1** via CDN (jsdelivr) for donut and line charts
- **Google Fonts Source Sans 3** — matches DOS typography
- No frameworks, no bundler, no external CSS

### Data model

The production version would call a SingStat aggregation endpoint. The current implementation uses a client-side mock with three layers:

1. **SSIC profiles** — each of the 16 modelled SSIC divisions has a realistic destination mix and a baseline 2024 export total. Example: electronics (SSIC 26) is dominated by China/US/HK; IT services (SSIC 62) by US/UK/Australia; food products (SSIC 10) by Malaysia/Indonesia
2. **Multipliers** — revenue band, employment size, and firm age each scale the baseline via multiplier tables
3. **Deterministic PRNG** — a hash of the four inputs seeds a linear congruential generator that produces all noise, so the same inputs always produce the same output

The 10-year trend for each destination is generated by back-walking from 2024 using region-specific baseline growth rates (ASEAN ≈ 5.5%, Asia-Pacific ≈ 4.2%, Americas ≈ 2.8%, Europe ≈ 1.8%) with noise, plus a COVID-19 dip in 2020 and a 2021 rebound.

To connect real data, replace `buildProfile()` in the script with a fetch call to the aggregation endpoint.

### Key files and functions

- `buildProfile(ssic, revenue, empSize, age)` — constructs the full result object from inputs
- `renderKPIs()` — populates the four KPI cards and HHI bar
- `renderPie()` — renders the donut chart with click-to-select behaviour
- `renderMarketList()` — renders the ranked destination list
- `selectMarket(code)` / `renderTrend()` — handles country selection and line chart rendering
- `getFilteredMarkets()` — applies the region filter to the current profile's markets

### Browser support

Tested against evergreen browsers (Chrome, Firefox, Safari, Edge). Uses only standard ES2015+ features and CSS Grid, both of which have universal support in the target audience.

---

## Known limitations

- **Mock data only** — the built-in dataset models 16 SSIC divisions and 20 destination countries. Real deployment requires wiring up the SingStat aggregation API
- **Flag emojis** — render differently on Windows (smaller, monochrome) vs macOS/iOS/Android. Consider SVG flags if pixel-perfect consistency across platforms matters
- **Trend chart y-axis** — does not beginAtZero by default, so small differences appear amplified. This is a deliberate choice for trend visibility but can be toggled in the Chart.js config
- **Region classification** — the `region` tag on each country is a simplified bucket (ASEAN / Asia-Pacific / Americas / Europe / Others) and doesn't match any single official classification; adjust to suit your organisation's reporting conventions
