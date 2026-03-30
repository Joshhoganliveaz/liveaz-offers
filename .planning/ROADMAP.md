# Roadmap: Offer Comparison Generator

## Overview

Build a single-file HTML offer comparison tool in three phases: first the complete data entry form (property, offers, costs, analysis), then the financial calculation engine that computes net-to-seller and ranks offers in real-time, and finally the branded comparison page output matching the Dava Dr design. Each phase delivers a testable layer of the tool.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

- [ ] **Phase 1: Form & Data Entry** - Complete input UI for property details, multiple offers, configurable cost template, and analysis notes
- [ ] **Phase 2: Calculations & Ranking** - Real-time net-to-seller engine with auto-ranking and difference display
- [ ] **Phase 3: Branded Output** - Generated comparison page matching Dava Dr design, print-optimized for PDF sharing

## Phase Details

### Phase 1: Form & Data Entry
**Goal**: Team can enter all property and offer data into a working form with persistent cost defaults
**Depends on**: Nothing (first phase)
**Requirements**: PROP-01, PROP-02, PROP-03, PROP-04, OFFR-01, OFFR-02, OFFR-03, OFFR-04, OFFR-05, OFFR-06, OFFR-07, OFFR-08, OFFR-09, COST-01, COST-02, COST-03, COST-04, COST-05, COST-06, ANLS-01, ANLS-02
**Success Criteria** (what must be TRUE):
  1. User can fill out property address, details (beds/baths/sqft/year built/lot/pool/garage), list price, and notable features
  2. User can add multiple offers with all term fields (buyer info, financial terms, timeline, compensation, financing, special terms, unrepresented flag)
  3. User can add and remove offers dynamically without losing data in other offers
  4. User can configure cost template line items (commission %, title/escrow, warranty, TC fee, custom items) and those defaults persist across browser sessions
  5. User can enter strategic analysis notes in a text area
**Plans:** 2 plans

Plans:
- [ ] 01-01-PLAN.md — HTML structure, CSS, property details, cost template with localStorage, analysis notes
- [ ] 01-02-PLAN.md — Dynamic offer card system (add/remove/renumber) with all field groups

### Phase 2: Calculations & Ranking
**Goal**: Net-to-seller calculates instantly as any field changes, and offers auto-sort by best net
**Depends on**: Phase 1
**Requirements**: CALC-01, CALC-02, CALC-03, CALC-04
**Success Criteria** (what must be TRUE):
  1. Changing any offer or cost field immediately updates the net-to-seller display without clicking a button
  2. Net-to-seller formula correctly applies: Purchase Price minus Listing Commission minus Buyer Agent Comp minus Concessions minus Title/Escrow minus Home Warranty minus TC Fee minus Additional Items
  3. Offers automatically reorder with highest net-to-seller first
  4. Each offer displays the dollar difference compared to the top-ranked offer
**Plans**: TBD

Plans:
- [ ] 02-01: TBD

### Phase 3: Branded Output
**Goal**: Team can generate a professional, print-ready comparison page that matches the Dava Dr design and produce clean PDFs
**Depends on**: Phase 2
**Requirements**: PAGE-01, PAGE-02, PAGE-03, PAGE-04, PAGE-05, PAGE-06, PAGE-07, PAGE-08, PAGE-09, OPT-01
**Success Criteria** (what must be TRUE):
  1. Clicking Generate opens a new tab with a branded comparison page using Dava Dr design tokens (Slate/Cream/Canyon/Gold/Olive, Playfair Display + DM Sans)
  2. Generated page includes property banner, ranked offer cards, side-by-side comparison table with best/worst highlighting, and financing strength section
  3. Printing the generated page to PDF produces a clean, professional document with no cut-off content or broken layouts
  4. Generated page displays strategic analysis notes and Live AZ Co footer with generation timestamp
  5. Optional mortgage payoff field, when filled, adds an interactive payoff calculator to the generated page

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Form & Data Entry | 0/2 | Planning complete | - |
| 2. Calculations & Ranking | 0/? | Not started | - |
| 3. Branded Output | 0/? | Not started | - |
