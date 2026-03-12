# Requirements: Homeowner Journey Map

**Defined:** 2026-03-11
**Core Value:** Show homeowners the full journey of their home -- where they've been, where they are, and where they could go -- so they feel empowered about their investment and keep Live AZ Co top-of-mind as their guide.

## v1 Requirements

### Data & Calculations

- [x] **DATA-01**: Page displays current home value estimate
- [x] **DATA-02**: Page shows equity gained since purchase (current value minus remaining mortgage)
- [ ] **DATA-03**: Page includes appreciation visualization (line or growth chart over time)
- [x] **DATA-04**: Page shows net proceeds estimate (what homeowner would walk away with after selling)
- [x] **DATA-05**: Page reframes equity as earnings per week/month ("Your home earned $847/week")
- [x] **DATA-06**: Page compares home appreciation vs S&P 500 performance over same period
- [x] **DATA-07**: Page compares cost of ownership vs renting the same home

### Future Scenarios

- [ ] **SCEN-01**: Hold & Rent projection -- when rental income would cover the mortgage
- [ ] **SCEN-02**: Move-Up projection -- net proceeds and what that buys as a down payment
- [ ] **SCEN-03**: Equity Play projection -- HELOC potential for investment property leverage

### Narrative & Design

- [x] **NARR-01**: Page uses scrolling narrative timeline (scrollytelling) that unfolds the home's story
- [x] **NARR-02**: All content follows SB7 framework -- homeowner is hero, Live AZ Co is guide
- [ ] **NARR-03**: Data reveals animate on scroll (counters count up, charts draw themselves)
- [ ] **NARR-04**: Narrative adapts by ownership duration (recent buyers get forward-looking; long-term get retrospective)
- [ ] **NARR-05**: Timeline includes milestone markers (purchase anniversary, equity milestones)
- [x] **NARR-06**: Live AZ Co branding throughout (Olive, Canyon, Gold, Cream, Charcoal palette; Source Serif 4 + DM Sans)
- [x] **NARR-07**: Mobile-first responsive design (primary viewing via text message on phone)
- [ ] **NARR-08**: Soft CTA to contact Josh ("Ready to explore your options? Let's talk.")

### Production

- [x] **PROD-01**: Admin input form creates a new page in under 5 minutes
- [x] **PROD-02**: Each homeowner page has a unique shareable URL

### Phase 02.1: Hardening & Edge Cases

- [ ] **FMT-01**: formatPercent uses whole numbers by default (24% not 24.3%)
- [ ] **FMT-02**: formatCurrency never shows cents (default 0 decimals)
- [ ] **FMT-03**: Negative currency displays with minus sign (-$23,000)
- [x] **SP5-01**: getSP500Price returns correct price for known month from static data
- [x] **SP5-02**: getSP500Price falls back to nearest available month
- [x] **SP5-03**: calcRealSP500Return computes actual growth from real data
- [x] **SP5-04**: calcRealSP500Return falls back to historical average when data unavailable
- [ ] **EDGE-01**: computeHomeownerMetrics handles negative appreciation correctly
- [ ] **EDGE-02**: computeHomeownerMetrics handles zero loan (cash purchase) correctly
- [ ] **VAL-01**: validateHomeownerInputs flags out-of-range values
- [ ] **VAL-02**: validateHomeownerInputs returns empty for valid inputs

## v2 Requirements

### Sharing & Polish

- **SHAR-01**: Dynamic OG social sharing card with branded preview when link is shared
- **SHAR-02**: Automated data refresh / revalidation for existing pages

### Data Integration

- **INTG-01**: Market data API integration (replace manual value entry)

## Out of Scope

| Feature | Reason |
|---------|--------|
| Interactive calculators / user-adjustable inputs | This is a curated experience, not a tool. Exploring scenarios = call Josh |
| User accounts or login for homeowners | Friction-free: tap link, see story. No signup needed |
| Automated email/text sending | Josh's personal touch is the differentiator |
| PDF export / printable report | CMA tool already handles print. This is digital-native |
| Neighborhood market reports | Homebot/Zillow already do this. This is about YOUR home, not the market |
| Homeowner self-service page creation | Value is that Josh creates each page personally |
| Complex dashboards or chart grids | This is a narrative, not a dashboard. Simple visualizations only |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| DATA-01 | Phase 2, 2.1 | Complete (hardened in 2.1) |
| DATA-02 | Phase 2, 2.1 | Complete (hardened in 2.1) |
| DATA-03 | Phase 2 | Pending |
| DATA-04 | Phase 2, 2.1 | Complete (hardened in 2.1) |
| DATA-05 | Phase 2, 2.1 | Complete (hardened in 2.1) |
| DATA-06 | Phase 2.1 | Complete |
| DATA-07 | Phase 2 | Complete |
| SCEN-01 | Phase 3 | Pending |
| SCEN-02 | Phase 3 | Pending |
| SCEN-03 | Phase 3 | Pending |
| NARR-01 | Phase 2 | Complete |
| NARR-02 | Phase 2 | Complete |
| NARR-03 | Phase 3 | Pending |
| NARR-04 | Phase 3 | Pending |
| NARR-05 | Phase 3 | Pending |
| NARR-06 | Phase 2 | Complete |
| NARR-07 | Phase 2 | Complete |
| NARR-08 | Phase 3 | Pending |
| PROD-01 | Phase 1, 2.1 | Complete (hardened in 2.1) |
| PROD-02 | Phase 1 | Complete |
| FMT-01 | Phase 2.1 | Pending |
| FMT-02 | Phase 2.1 | Pending |
| FMT-03 | Phase 2.1 | Pending |
| SP5-01 | Phase 2.1 | Complete |
| SP5-02 | Phase 2.1 | Complete |
| SP5-03 | Phase 2.1 | Complete |
| SP5-04 | Phase 2.1 | Complete |
| EDGE-01 | Phase 2.1 | Pending |
| EDGE-02 | Phase 2.1 | Pending |
| VAL-01 | Phase 2.1 | Pending |
| VAL-02 | Phase 2.1 | Pending |

**Coverage:**
- v1 requirements: 20 total
- Phase 2.1 hardening requirements: 11 total
- Mapped to phases: 31
- Unmapped: 0

---
*Requirements defined: 2026-03-11*
*Last updated: 2026-03-12 after Phase 02.1 planning*
