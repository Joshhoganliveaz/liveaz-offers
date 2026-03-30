# Requirements: Offer Comparison Generator

**Defined:** 2026-03-30
**Core Value:** Team can independently create professional offer comparison pages without developer involvement

## v1 Requirements

### Property Input

- [x] **PROP-01**: User can enter property address (street, city, state, zip)
- [x] **PROP-02**: User can enter property details (beds, baths, sqft, year built, lot size, pool, garage spaces)
- [x] **PROP-03**: User can enter list price
- [x] **PROP-04**: User can enter notable features as free text

### Offer Input

- [ ] **OFFR-01**: User can add an offer with buyer name(s), buyer agent name, and brokerage
- [ ] **OFFR-02**: User can enter financial terms: purchase price, down payment (amount + %), loan type, earnest money
- [ ] **OFFR-03**: User can enter timeline terms: close of escrow date, inspection period days, offer expiration
- [ ] **OFFR-04**: User can enter compensation terms: concessions amount, buyer agent comp (amount + %)
- [ ] **OFFR-05**: User can enter financing details: pre-qual amount, pre-qual lender
- [ ] **OFFR-06**: User can enter special terms as free text
- [ ] **OFFR-07**: User can flag an offer as unrepresented buyer (adjusts commission handling)
- [ ] **OFFR-08**: User can add multiple offers dynamically
- [ ] **OFFR-09**: User can remove an offer

### Cost Template

- [x] **COST-01**: User can configure listing commission percentage
- [x] **COST-02**: User can configure title/escrow estimate
- [x] **COST-03**: User can configure home warranty amount
- [x] **COST-04**: User can configure TC fee
- [x] **COST-05**: User can add/remove additional line items with label and amount
- [x] **COST-06**: Cost template persists in localStorage across sessions

### Calculations

- [ ] **CALC-01**: Net-to-seller calculates in real-time as any field changes
- [ ] **CALC-02**: Net formula: Purchase Price - Listing Commission - Buyer Agent Comp - Concessions - Title/Escrow - Home Warranty - TC Fee - Additional Items
- [ ] **CALC-03**: Offers auto-rank by net-to-seller, highest first
- [ ] **CALC-04**: Each offer shows dollar difference vs. top offer

### Analysis

- [x] **ANLS-01**: User can enter strategic analysis notes in a text area
- [x] **ANLS-02**: Analysis text appears in the generated comparison page

### Generated Page

- [ ] **PAGE-01**: Generate button produces a branded comparison page in a new tab
- [ ] **PAGE-02**: Page matches Dava Dr design (Slate/Cream/Canyon/Gold/Olive tokens, Playfair Display + DM Sans)
- [ ] **PAGE-03**: Page includes property banner with address, features, list price
- [ ] **PAGE-04**: Page includes ranked offer cards with key terms
- [ ] **PAGE-05**: Page includes side-by-side comparison table (best/worst values highlighted)
- [ ] **PAGE-06**: Page includes financing strength section (pre-qual amounts, headroom %, down payment comparison)
- [ ] **PAGE-07**: Page is print-optimized (clean PDF output via browser print dialog)
- [ ] **PAGE-08**: Page is responsive (mobile card stacking)
- [ ] **PAGE-09**: Page includes footer with Live AZ Co branding and generation timestamp

### Optional

- [ ] **OPT-01**: User can enter mortgage payoff amount; generated page includes interactive payoff calculator

## v2 Requirements

### PDF Parsing

- **PDF-01**: Upload ARMLS listing PDF for auto-extraction of property fields
- **PDF-02**: Upload AAR purchase contract PDF for auto-extraction of offer terms
- **PDF-03**: Addendum merging (counter offer terms override original)

### Publishing

- **PUB-01**: One-click publish to offers.liveazco.com via GitHub API
- **PUB-02**: Load existing published page for editing

## Out of Scope

| Feature | Reason |
|---------|--------|
| Claude API / PDF parsing | Frequency (2-4x/mo) doesn't justify API infrastructure; team reads contracts anyway |
| Backend / API endpoints | No server-side processing needed for manual entry + client-side generation |
| Authentication | Local tool, not publicly linked |
| Database / data persistence | Each comparison is standalone; cost template uses localStorage |
| CRM integration | Separate system, no overlap needed |
| Multi-user editing | Team of three, verbal coordination |
| Email/SMS notifications | Manual sharing is fine at current volume |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| PROP-01 | Phase 1 | Complete |
| PROP-02 | Phase 1 | Complete |
| PROP-03 | Phase 1 | Complete |
| PROP-04 | Phase 1 | Complete |
| OFFR-01 | Phase 1 | Pending |
| OFFR-02 | Phase 1 | Pending |
| OFFR-03 | Phase 1 | Pending |
| OFFR-04 | Phase 1 | Pending |
| OFFR-05 | Phase 1 | Pending |
| OFFR-06 | Phase 1 | Pending |
| OFFR-07 | Phase 1 | Pending |
| OFFR-08 | Phase 1 | Pending |
| OFFR-09 | Phase 1 | Pending |
| COST-01 | Phase 1 | Complete |
| COST-02 | Phase 1 | Complete |
| COST-03 | Phase 1 | Complete |
| COST-04 | Phase 1 | Complete |
| COST-05 | Phase 1 | Complete |
| COST-06 | Phase 1 | Complete |
| CALC-01 | Phase 2 | Pending |
| CALC-02 | Phase 2 | Pending |
| CALC-03 | Phase 2 | Pending |
| CALC-04 | Phase 2 | Pending |
| ANLS-01 | Phase 1 | Complete |
| ANLS-02 | Phase 1 | Complete |
| PAGE-01 | Phase 3 | Pending |
| PAGE-02 | Phase 3 | Pending |
| PAGE-03 | Phase 3 | Pending |
| PAGE-04 | Phase 3 | Pending |
| PAGE-05 | Phase 3 | Pending |
| PAGE-06 | Phase 3 | Pending |
| PAGE-07 | Phase 3 | Pending |
| PAGE-08 | Phase 3 | Pending |
| PAGE-09 | Phase 3 | Pending |
| OPT-01 | Phase 3 | Pending |

**Coverage:**
- v1 requirements: 35 total
- Mapped to phases: 35
- Unmapped: 0

---
*Requirements defined: 2026-03-30*
*Last updated: 2026-03-30 — phase traceability added*
