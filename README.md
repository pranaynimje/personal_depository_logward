# D&D Intelligence Hub — Dashboard Documentation

> Detention & Demurrage monitoring dashboard for FCL container portfolios.
> Built with React + Recharts. Mock data — representative of a live Logward portfolio.

---

## Overview

The D&D Intelligence Hub gives logistics and operations teams a single view of their container detention and demurrage exposure — across carriers, lanes, and cost categories. It is structured around six tabs, each answering a specific operational question.

| Tab | Name | Question it answers |
|---|---|---|
| 1 | Command Center | What is my total D&D exposure right now? |
| 2 | Cost Overview | Where exactly is that cost coming from? |
| 3 | Carrier Intel | Which carriers are driving the problem? |
| 4 | Cost Optimizer | Which containers should I act on today? |
| 5 | Historical | Is the situation getting better or worse? |
| 6 | Surcharges | Which lanes have the worst surcharge terms? |

---

## Tab 1 — Command Center

**Purpose:** High-level portfolio health at a glance.

- **Grand Total KPI** — total D&D exposure across all containers ($184,370 in current dataset)
- **Origin vs Destination split** — 94% of cost sits at origin (pre-shipment), 6% at destination
- **Free Time Health** — containers grouped into Red (35, breaching today), Yellow (24, expiring soon), Green (205, within free period), Expired (3,780 past free period)
- **Container Journey Visual** — milestone flow from Gate In POL → Load POL → Ocean Transit → Discharge POD → Gate Out POD → Empty Return, showing average dwell at each stage
- **Cost Breakdown Table** — four charge categories (Detention, Demurrage, Storage, Combined D&D) × Origin/Destination with total cost, container count, and avg free period
- **Top Risk Containers** — highest-risk individual containers with daily burn rate and recommended action

---

## Tab 2 — Cost Overview

**Purpose:** Full breakdown of where cost is concentrated.

- **Origin vs Destination bar chart** — all charge categories compared side by side
- **Cost Distribution pie chart** — proportional share of each category
- **Full Cost Matrix** — all 7 surcharge categories (see below) × Origin/Dest/Total/Containers with cost/avg free period
- **Contributing Factors** — worst carrier, worst lane group, avg days beyond free period

### Charge Categories

| Category | Description | Origin Free Period | Dest Free Period |
|---|---|---|---|
| Detention | Equipment (container) held beyond allowed pickup/return window | 5.1 days | 6.0 days |
| Demurrage | Container left at port beyond free storage window | 3.1 days | 3.0 days |
| Storage | Warehouse/yard storage charge | 3.1 days | 3.0 days |
| Combined D&D | Single combined detention + demurrage rate (carrier-negotiated) | 9.9 days | 12.0 days |
| Demurrage + Storage | Combined port + warehouse charge | 4.2 days | 4.0 days |
| Detention + Demurrage | Combined equipment + port charge | 7.8 days | 8.5 days |
| Det. + Dem. + Storage | All three combined | 11.2 days | 13.0 days |

> **Note:** Combined rates (D&D, Det+Dem+Storage) have a single pooled free period shared across charge types. Whether a combined or separate rate is cheaper depends on the lane's actual dwell pattern — see Tab 6 Surcharges for per-lane rate recommendation.

---

## Tab 3 — Carrier Intel

**Purpose:** Identify which carriers contribute most to D&D cost.

### View Switcher Dropdown

The tab has four selectable views. Switching the dropdown updates both the chart and the scorecard table.

#### 1. Avg Dwell Time
Shows average dwell days per carrier split by origin (left bar) and destination (right bar).
- Colors: Detention (amber), Demurrage (purple), Combined D&D (red)
- Reference line at 5.1d = origin detention free period
- Bars above the line = containers in paid tiers on average

**Scorecard columns:** Carrier · Volume · O.Det · O.Dem · Total Origin · Beyond FP · D.Dem · D.Det · Total Dest · Missing Milestones · Carrier Score

#### 2. Containers Exceeding Free Days
Shows count of containers estimated past their free period vs within it.
- Red = past free period, Green = within free period
- Use this view to prioritise which carrier's fleet needs release action first

**Scorecard columns:** Carrier · Total Vol · Past FP (count) · % Past FP · Within FP · Avg Beyond (days) · Risk Level

#### 3. Cost Exposure by Carrier
Estimated total D&D cost per carrier, sorted highest to lowest.
- Formula: `(pastFPCount × daysOverdue × $120) + (containers × avgDDem × $40) + (containers × avgDDet × $80)`
- Directional estimate — not billing data

**Scorecard columns:** Carrier · Vol · Est. Cost · Cost/Container · Past FP Count · Avg Beyond · Score

#### 4. Free Period Tier Distribution
Shows how containers are distributed across cost tiers for each carrier.

| Tier | Meaning | Est. Daily Rate |
|---|---|---|
| Free (No Charge) | Within free period | $0/day |
| 1–3 Days Overdue | Just past free period, low tier | ~$50/day |
| 4–7 Days Overdue | Mid tier, costs escalating | ~$100/day |
| 7+ Days Overdue | Highest tier, urgent action needed | ~$200/day |

**Scorecard columns:** Carrier · Vol · Free · 1–3d Over · 4–7d Over · 7d+ Over · % Overdue · Score (Deep / Moderate)

### Carrier Score Formula

The **Carrier Score** (0–100, higher = worse D&D profile) is:

```
min(100,
  (avgODet + avgODem) × 8
  + (avgDDem + avgDDet) × 5
  + (missingMilestones / containers) × 2
)
```

- Origin dwell weighted more heavily (×8) — origin detention is the dominant cost driver (94% of portfolio)
- Missing milestones penalised — indicates data gaps that hide true dwell
- Score > 70 = Red, 40–70 = Amber, < 40 = Green

> Scores are for internal prioritisation only. Business relationships and contractual terms should inform decisions alongside these scores.

### Container Risk Score (drill-down)

When you click a carrier row, top-risk containers appear. Each container has an individual **Container Risk Score** (distinct from the carrier aggregate):

```
Based on: days in origin detention beyond free period + missing milestone count
Range: 0–100. Score ≥ 75 = High (red). 50–74 = Medium (amber).
```

---

## Tab 4 — Cost Optimizer

**Purpose:** Prioritise which containers to act on given a target date.

- **Date-Based Prediction** — select a future date; dashboard estimates total cost if no containers are cleared by then, broken down by charge category
- **Container Prioritization Planner** — split-screen A/B comparison of container batches with shared and per-group filters
- **Avoidable Cost** — cost that can be eliminated by clearing a container today; `+3d` and `+7d` columns show cost if delayed

### Daily Rate Tiers (used in prediction)

| Days beyond free period | Daily rate applied |
|---|---|
| 1–3 days | $50/day |
| 4–7 days | $100/day |
| 7+ days | $200/day |

### Container Risk Level (filter in Planner)

| Level | Score range |
|---|---|
| High | ≥ 75 |
| Medium | 50–74 |
| Low | < 50 |

---

## Tab 5 — Historical

**Purpose:** Month-over-month trend to assess whether actions are working.

- **Monthly D&D Cost Trend** — stacked area chart: Detention, Demurrage, Storage, Combined D&D
- **Filter:** All / Origin only / Destination only
- **Stage Performance Table** — each journey stage with avg dwell, free period, breach days, cost, and status (Over / Near / OK)
- **Port Dwell** — avg dwell at each POL (origin) and POD (destination), ranked by score
- **Lane Performance** — freight vs D&D surcharge split per lane; lanes with surcharge ratio > 35% flagged

### Port Score Formula

```
POL score = (avgODet × 0.6) + (avgODem × 0.4)
POD score = (avgDDem × 0.5) + (avgDDet × 0.5)
```

---

## Tab 6 — Surcharges

**Purpose:** Lane-level surcharge terms and negotiation intelligence.

- **Lane selector** — choose any lane to see its full surcharge breakdown
- **Rate Recommendation** — dashboard compares combined D&D rate vs separate detention + demurrage rates and recommends which is cheaper for that lane's actual dwell profile
- **Negotiation Script** — auto-generated talking points for carrier renewal discussions based on actual dwell data
- **Free Period Ask** — calculated additional free days to request, based on how many days containers exceed the current free period on average

---

## Data Reference

### Portfolio Snapshot (current mock data)

| Metric | Value |
|---|---|
| Total D&D Exposure | $184,370 |
| Origin Exposure | $174,162 (94%) |
| Destination Exposure | $10,208 (6%) |
| Total Carriers | 8 |
| Avg Origin Detention Dwell | 4.8 days |
| Avg Destination Detention Dwell | 5.51 days |
| Ocean Transit Avg | 44.87 days |
| Containers Breaching Free Period | 35 (red) |
| Containers Expiring Soon | 24 (yellow) |

### Carrier Data

| Carrier | Containers | Avg O.Det | Avg O.Dem | Avg D.Dem | Avg D.Det | Missing Milestones |
|---|---|---|---|---|---|---|
| OOLU | 288 | 9.86d | 0.97d | 2.78d | 5.81d | 813 |
| ONEY | 905 | 2.37d | 0.87d | 2.66d | 5.48d | 3,106 |
| MSCU | 227 | 5.98d | 1.01d | 2.55d | 5.59d | 642 |
| MAEU | 229 | 7.77d | 1.00d | 3.22d | 5.86d | 744 |
| HLCU | 427 | 5.62d | 0.74d | 2.39d | 5.23d | 1,301 |
| EGLV | 139 | 2.09d | 0.43d | 1.38d | 4.82d | 375 |
| COSU | 141 | 0.73d | 0.82d | 2.82d | 5.74d | 451 |
| CMDU | 279 | 6.35d | 1.00d | 2.79d | 5.81d | 815 |

---

## Glossary

| Term | Meaning |
|---|---|
| D&D | Detention & Demurrage |
| Detention | Charge for holding a shipping line's container beyond the agreed free return window |
| Demurrage | Charge for leaving a container at the port/terminal beyond the free storage window |
| Storage | Warehouse or yard storage charge after port pickup |
| Free Period (FP) | Number of days included at no charge before detention/demurrage begins |
| Beyond FP | Days a container has exceeded its free period — every day incurs a daily charge |
| POL | Port of Loading (origin port) |
| POD | Port of Discharge (destination port) |
| O.Det | Origin Detention (days) |
| O.Dem | Origin Demurrage (days) |
| D.Det | Destination Detention (days) |
| D.Dem | Destination Demurrage (days) |
| Missing Milestones | Tracking events not received — reduces visibility into actual dwell |
| SCAC | Standard Carrier Alpha Code (2–4 letter carrier identifier) |
