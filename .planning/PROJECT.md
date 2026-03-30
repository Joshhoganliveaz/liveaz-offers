# Offer Comparison Generator

## What This Is

A single-file HTML tool that lets the Live AZ Co team (Jacqui, Suzie) manually enter property details and offer terms, auto-calculate net-to-seller for each offer, and generate a branded, print-ready comparison page. The team prints to PDF for sharing with sellers. Josh can optionally publish the HTML to offers.liveazco.com.

## Core Value

The team can independently create professional offer comparison pages without developer involvement, using a tool they already know how to use (same pattern as the Tenet converter).

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Team can enter property details (address, beds/baths/sqft, list price, year built, features)
- [ ] Team can add multiple offers with key terms (price, down payment, loan type, earnest money, close date, concessions, buyer agent comp, pre-qual amount, inspection days)
- [ ] Team can add/remove offers dynamically
- [ ] Cost template is configurable (listing commission %, title/escrow, home warranty, TC fee, additional line items) and persists in localStorage
- [ ] Net-to-seller auto-calculates in real-time as fields change
- [ ] Offers auto-rank by net-to-seller, highest first, with dollar difference vs. top offer
- [ ] Generate button produces a branded comparison page matching the Dava Dr design
- [ ] Generated page opens in a new tab, optimized for print-to-PDF
- [ ] Generated page includes: property banner, ranked offer cards, side-by-side comparison table, financing strength section
- [ ] Strategic analysis text area for team's notes (included in generated page)
- [ ] Unrepresented buyer flag with adjustable commission handling
- [ ] Optional mortgage payoff field with interactive calculator on generated page

### Out of Scope

- PDF parsing / Claude API integration — team reads contracts and enters data manually
- Cloudflare Pages Functions / API endpoints — no backend needed
- One-click publish to GitHub — Josh handles publishing manually when needed
- Load Existing / round-tripping — each comparison is a fresh entry
- Authentication — local tool, not publicly accessible
- Integration with Lofty CRM — separate system

## Context

- **Existing reference:** `1960-e-dava-dr/index.html` is the design template for generated pages
- **Team:** Jacqui and Suzie coordinate with sellers daily and already read through every contract. Manual entry adds minimal overhead since they review every field anyway
- **Existing pattern:** The Tenet converter (`~/Projects/Tenet Equity/tenet-converter.html`) is a single-file HTML tool the team already uses. Same pattern here
- **Usage frequency:** 2-4 times per month
- **Output:** Team prints the generated page to PDF and shares with sellers. Josh can optionally push the HTML file to the repo for web publishing

## Constraints

- **Tech stack**: Single-file HTML, no build tools, no framework, no backend
- **Design**: Must match Dava Dr page design tokens (Slate, Cream, Canyon, Gold, Olive; Playfair Display + DM Sans)
- **Output**: Print-optimized layout that produces clean PDFs via browser print dialog
- **Persistence**: localStorage only (cost template defaults)

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Drop PDF parsing / Claude API | 2-4x/month frequency doesn't justify API infrastructure. Team reads contracts anyway | — Pending |
| Drop one-click publish | Josh can manually push HTML when needed. Eliminates GitHub API + Cloudflare Functions | — Pending |
| Single-file HTML (no framework) | Matches Tenet converter pattern, zero dependencies, team already knows this | — Pending |
| Print-to-PDF as primary output | No backend needed. Browser print dialog produces shareable PDFs instantly | — Pending |
| Manual entry over automation | Team reviews every field in every contract. Manual entry is the confirmation step | — Pending |

---
*Last updated: 2026-03-30 after strategic pivot to lightweight template*
