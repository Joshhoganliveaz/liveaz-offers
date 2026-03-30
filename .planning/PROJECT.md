# Offer Comparison Admin Tool

## What This Is

A self-service admin tool at `offers.liveazco.com/_admin/` that lets the Live AZ Co team (Jacqui, Suzie) upload MLS listing and purchase contract PDFs, review Claude-extracted data with confirmation checkboxes, and publish branded offer comparison pages with one click. Removes Josh as the bottleneck for creating offer comparison pages that sellers use to evaluate competing offers on their property.

## Core Value

Sellers receive accurate, professionally branded offer comparison pages that the team can create and update independently, without developer involvement.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Team can upload an ARMLS MLS listing PDF and get structured property data extracted by Claude
- [ ] Team can upload one or more AAR purchase contract PDFs and get structured offer data extracted
- [ ] Addendums/counter offers can be attached to existing offer cards, with Claude merging terms
- [ ] Every extracted field is editable with a confirmation checkbox gate before publishing
- [ ] Net-to-seller auto-calculates in real-time using cost template + offer terms
- [ ] Offers auto-rank by net-to-seller, highest first
- [ ] Cost template is configurable and persists in localStorage
- [ ] Claude drafts strategic analysis comparing all offers; team can edit before publishing
- [ ] Preview opens the generated page in a new tab
- [ ] Publish commits HTML to GitHub repo via Pages Function; Cloudflare auto-deploys
- [ ] Load Existing dropdown lets team re-open and update previously published pages
- [ ] Generated pages match the Dava Dr design (Slate/Cream/Canyon/Gold/Olive tokens, Playfair Display + DM Sans)
- [ ] Generated pages are fully responsive with print-optimized layout
- [ ] All API endpoints secured with X-Admin-Key header validation
- [ ] CORS restricted to offers.liveazco.com

### Out of Scope

- Authentication/password protection on admin page — discoverability is sufficient for team of three
- Multi-user simultaneous editing — team coordinates verbally
- Historical tracking of offer changes — not a current need
- Email/SMS notifications on publish — manual sharing is fine
- Automatic offer ranking commentary beyond Claude's initial draft — team edits manually
- Integration with Lofty CRM — separate system, no overlap needed for v1

## Context

- **Existing reference:** `1960-e-dava-dr/index.html` is the design template for generated pages
- **Team:** Jacqui and Suzie coordinate with sellers daily but cannot create HTML pages. Josh currently hand-builds each one with Claude Code
- **Domain:** Arizona real estate. ARMLS MLS listing PDFs (1-3 pages) and AAR purchase contracts (10-30 pages)
- **Existing pattern:** The Tenet converter (`~/Projects/Tenet Equity/tenet-converter.html`) is a single-file HTML admin tool the team already uses. Same architectural pattern here
- **Deployment:** Cloudflare Pages with Pages Functions. Repo auto-deploys on push

## Constraints

- **Tech stack**: Single-file HTML admin page, no build tools, no framework — must match team's existing Tenet converter pattern
- **Infrastructure**: Cloudflare Pages Functions only (no standalone Workers, no database)
- **API**: Claude Sonnet (claude-sonnet-4-20250514) for PDF extraction via tool_use structured output
- **Security**: X-Admin-Key header on all API endpoints + CORS restricted to offers.liveazco.com
- **File size**: Client-side validation rejects PDFs over 10MB
- **Publishing**: GitHub Contents API to create/update files in the repo

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Single-file HTML admin (no framework) | Matches Tenet converter pattern, no build tools, any developer can maintain | — Pending |
| Pages Functions over standalone Worker | Same repo, same deployment, zero additional infrastructure | — Pending |
| Checkbox confirmation gate | Claude may misread PDF numbers; sellers make six-figure decisions on these pages | — Pending |
| Embedded JSON for round-tripping | No database to maintain; generated page carries its own data | — Pending |
| No auth on admin page (v1) | `_admin` path not publicly linked; team of three | — Pending |

---
*Last updated: 2026-03-30 after initialization*
