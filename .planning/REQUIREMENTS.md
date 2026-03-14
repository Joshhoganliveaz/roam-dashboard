# Requirements: Homeowner Journey Map

**Defined:** 2026-03-11
**Core Value:** Show homeowners the full journey of their home -- where they've been, where they are, and where they could go -- so they feel empowered about their investment and keep Live AZ Co top-of-mind as their guide.

## v1 Requirements

### Data & Calculations

- [x] **DATA-01**: Page displays current home value estimate
- [x] **DATA-02**: Page shows equity gained since purchase (current value minus remaining mortgage)
- [x] **DATA-03**: Page includes appreciation visualization (line or growth chart over time)
- [x] **DATA-04**: Page shows net proceeds estimate (what homeowner would walk away with after selling)
- [x] **DATA-05**: Page reframes equity as earnings per week/month ("Your home earned $847/week")
- [x] **DATA-06**: Page compares home appreciation vs S&P 500 performance over same period
- [x] **DATA-07**: Page compares cost of ownership vs renting the same home

### Future Scenarios

- [x] **SCEN-01**: Hold & Rent projection -- when rental income would cover the mortgage
- [x] **SCEN-02**: Move-Up projection -- net proceeds and what that buys as a down payment
- [x] **SCEN-03**: Equity Play projection -- HELOC potential for investment property leverage

### Narrative & Design

- [x] **NARR-01**: Page uses scrolling narrative timeline (scrollytelling) that unfolds the home's story
- [x] **NARR-02**: All content follows SB7 framework -- homeowner is hero, Live AZ Co is guide
- [x] **NARR-03**: Data reveals animate on scroll (counters count up, charts draw themselves)
- [x] **NARR-04**: Narrative adapts by ownership duration (recent buyers get forward-looking; long-term get retrospective)
- [x] **NARR-05**: Timeline includes milestone markers (purchase anniversary, equity milestones)
- [x] **NARR-06**: Live AZ Co branding throughout (Olive, Canyon, Gold, Cream, Charcoal palette; Source Serif 4 + DM Sans)
- [x] **NARR-07**: Mobile-first responsive design (primary viewing via text message on phone)
- [x] **NARR-08**: Soft CTA to contact Josh ("Ready to explore your options? Let's talk.")

### Production

- [x] **PROD-01**: Admin input form creates a new page in under 5 minutes
- [x] **PROD-02**: Each homeowner page has a unique shareable URL

### Phase 02.1: Hardening & Edge Cases

- [x] **FMT-01**: formatPercent uses whole numbers by default (24% not 24.3%)
- [x] **FMT-02**: formatCurrency never shows cents (default 0 decimals)
- [x] **FMT-03**: Negative currency displays with minus sign (-$23,000)
- [x] **SP5-01**: getSP500Price returns correct price for known month from static data
- [x] **SP5-02**: getSP500Price falls back to nearest available month
- [x] **SP5-03**: calcRealSP500Return computes actual growth from real data
- [x] **SP5-04**: calcRealSP500Return falls back to historical average when data unavailable
- [x] **EDGE-01**: computeHomeownerMetrics handles negative appreciation correctly
- [x] **EDGE-02**: computeHomeownerMetrics handles zero loan (cash purchase) correctly
- [x] **VAL-01**: validateHomeownerInputs flags out-of-range values
- [x] **VAL-02**: validateHomeownerInputs returns empty for valid inputs

## v1.1 Requirements

### Value Range

- [x] **VALU-01**: Josh can enter low comp value and high comp value in admin form (replaces single currentValue)
- [x] **VALU-02**: Homeowner sees three condition choices (Mostly original / Made some updates / Completely remodeled) that map to positions within the comp range
- [x] **VALU-03**: Selecting a condition recalculates all downstream numbers (equity, scenarios, comparisons, net proceeds) using the condition-mapped value
- [x] **VALU-04**: Condition change triggers cascading ripple animation flowing down the page section by section

### Cromford Chart

- [x] **CROM-01**: Josh can upload a Cromford Report screenshot in the admin form
- [x] **CROM-02**: System extracts price-per-sqft time-series data points from the uploaded screenshot via vision/OCR
- [x] **CROM-03**: Appreciation chart displays real market data as a branded, animated Recharts chart (replaces straight-line estimate)
- [x] **CROM-04**: Condition selector shifts the entire appreciation curve proportionally (price-per-sqft × home sqft × condition factor)

### Investment Comparison

- [x] **INVS-01**: Single "Investment Comparison" section replaces separate S&P 500 and Rent vs Own sections
- [x] **INVS-02**: Section shows three key numbers: home ROI with leverage, S&P alternative return on down payment, and rent drag subtracted from S&P path
- [x] **INVS-03**: Personalized leverage callout using their specific numbers (e.g., "$25K down controlling a $500K asset = 165% return")
- [x] **INVS-04**: Rate lock advantage callout quantifying annual savings vs current market rates

### Future Scenarios

- [ ] **SCEN-04**: All three scenarios show 5-year, 10-year, and 15-year projection timelines
- [ ] **SCEN-05**: One market rate slider affects all scenarios (refi rate for Hold & Rent, purchase rate for Move-Up, HELOC rate for Equity Play)
- [ ] **SCEN-06**: Rate lock advantage narrative — if homeowner's rate is below market, highlight competitive advantage instead of refi option
- [ ] **SCEN-07**: 4th combo scenario card: "Keep, rent, and buy another" chaining Hold & Rent + Equity Play + Move-Up math
- [ ] **SCEN-08**: Enhanced Hold & Rent: projected cash flow and appreciation at 5/10/15yr, rent growth vs fixed mortgage gap widening
- [ ] **SCEN-09**: Enhanced Move-Up: education on sell-to-buy paths (sell first, bridge loan, rent-back, simultaneous close)
- [ ] **SCEN-10**: Enhanced Equity Play: HELOC potential with rate-adjusted costs from market rate slider
- [x] **SCEN-11**: Information cliffs removed — full information so homeowners feel empowered, conversations happen when ready

## v2 Requirements

### Sharing & Polish

- **SHAR-01**: Dynamic OG social sharing card with branded preview when link is shared
- **SHAR-02**: Automated data refresh / revalidation for existing pages

### Data Integration

- **INTG-01**: Market data API integration (replace manual value entry)

### Cromford Automation

- **CROM-05**: Automated Tableau Public scraping (deferred — reverse-engineered API too fragile for v1.1)

## Out of Scope

| Feature | Reason |
|---------|--------|
| User accounts or login for homeowners | Friction-free: tap link, see story. No signup needed |
| Automated email/text sending | Josh's personal touch is the differentiator |
| PDF export / printable report | CMA tool already handles print. This is digital-native |
| Neighborhood market reports | Homebot/Zillow already do this. This is about YOUR home, not the market |
| Homeowner self-service page creation | Value is that Josh creates each page personally |
| Automated Tableau scraping | Reverse-engineered API too fragile; screenshot OCR is more reliable |
| Appraisal-grade chart accuracy | Appreciation chart is narrative, not legal — directional accuracy sufficient |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| DATA-01 | Phase 2, 2.1 | Complete (hardened in 2.1) |
| DATA-02 | Phase 2, 2.1 | Complete (hardened in 2.1) |
| DATA-03 | Phase 2 | Complete |
| DATA-04 | Phase 2, 2.1 | Complete (hardened in 2.1) |
| DATA-05 | Phase 2, 2.1 | Complete (hardened in 2.1) |
| DATA-06 | Phase 2.1 | Complete |
| DATA-07 | Phase 2 | Complete |
| SCEN-01 | Phase 3 | Complete |
| SCEN-02 | Phase 3 | Complete |
| SCEN-03 | Phase 3 | Complete |
| NARR-01 | Phase 2 | Complete |
| NARR-02 | Phase 2 | Complete |
| NARR-03 | Phase 3 | Complete |
| NARR-04 | Phase 3 | Complete |
| NARR-05 | Phase 3 | Complete |
| NARR-06 | Phase 2 | Complete |
| NARR-07 | Phase 2 | Complete |
| NARR-08 | Phase 3 | Complete |
| PROD-01 | Phase 1, 2.1 | Complete (hardened in 2.1) |
| PROD-02 | Phase 1 | Complete |
| FMT-01 | Phase 2.1 | Complete |
| FMT-02 | Phase 2.1 | Complete |
| FMT-03 | Phase 2.1 | Complete |
| SP5-01 | Phase 2.1 | Complete |
| SP5-02 | Phase 2.1 | Complete |
| SP5-03 | Phase 2.1 | Complete |
| SP5-04 | Phase 2.1 | Complete |
| EDGE-01 | Phase 2.1 | Complete |
| EDGE-02 | Phase 2.1 | Complete |
| VAL-01 | Phase 2.1 | Complete |
| VAL-02 | Phase 2.1 | Complete |
| VALU-01 | Phase 5 | Complete |
| VALU-02 | Phase 5 | Complete |
| VALU-03 | Phase 5 | Complete |
| VALU-04 | Phase 5 | Complete |
| CROM-01 | Phase 5 | Complete |
| CROM-02 | Phase 5 | Complete |
| CROM-03 | Phase 5 | Complete |
| CROM-04 | Phase 5 | Complete |
| INVS-01 | Phase 6 | Complete |
| INVS-02 | Phase 6 | Complete |
| INVS-03 | Phase 6 | Complete |
| INVS-04 | Phase 6 | Complete |
| SCEN-11 | Phase 6 | Complete |
| SCEN-04 | Phase 7 | Pending |
| SCEN-05 | Phase 7 | Pending |
| SCEN-06 | Phase 7 | Pending |
| SCEN-07 | Phase 7 | Pending |
| SCEN-08 | Phase 7 | Pending |
| SCEN-09 | Phase 7 | Pending |
| SCEN-10 | Phase 7 | Pending |

**v1.0 Coverage:**
- v1.0 requirements: 31 total
- Mapped to phases: 31
- Unmapped: 0 ✓

**v1.1 Coverage:**
- v1.1 requirements: 20 total
- Mapped to phases: 20
- Unmapped: 0 ✓

---
*Requirements defined: 2026-03-11*
*Last updated: 2026-03-13 after v1.1 roadmap creation*
