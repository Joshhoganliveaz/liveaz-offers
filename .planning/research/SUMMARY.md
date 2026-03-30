# Research Summary: LiveAZ Offers Admin Tool

**Domain:** Real estate offer comparison admin tool with PDF extraction and static page publishing
**Researched:** 2026-03-30
**Overall confidence:** HIGH

## Executive Summary

The LiveAZ Offers tool follows a well-established pattern: single-file HTML admin page with serverless API proxy functions. The team already uses this pattern with the Tenet converter, so there is zero learning curve on the client side. The backend uses Cloudflare Pages Functions (which are Workers under the hood) to proxy two external APIs: Anthropic's Claude API for PDF extraction and GitHub's Contents API for publishing.

The stack is deliberately minimal. No npm, no build tools, no framework, no database. The admin page is one HTML file with inline CSS and JavaScript. The backend is 5-6 TypeScript functions that each make one or two fetch() calls. External dependencies are zero beyond what Cloudflare and the browser provide natively.

The critical technical insight is that Claude's tool_use feature (GA, works on all models) is the right extraction approach, not the newer structured outputs beta (which requires Sonnet 4.5 or Opus 4.1, not the Sonnet 4 specified in PROJECT.md). Tool_use forces Claude to return JSON matching a defined schema, which maps directly to the admin form fields. Each extraction function defines a tool schema that serves as both the API contract and the data model.

The biggest non-obvious risk is the Cloudflare free plan's 10ms CPU time limit, which is incompatible with any function that calls an external API. The Workers Paid plan ($5/month) is required before any development begins.

## Key Findings

**Stack:** Zero-dependency architecture. Single-file HTML admin, Cloudflare Pages Functions (TypeScript), direct fetch() to Claude API and GitHub API. No SDKs, no build tools.

**Architecture:** Client generates HTML from template + confirmed data. Functions are thin API proxies. Generated pages carry embedded JSON for round-trip editing. GitHub repo is the "database."

**Critical pitfall:** Cloudflare free plan CPU limit (10ms) kills all API proxy calls. Must use Workers Paid ($5/mo) from day one.

## Implications for Roadmap

Based on research, suggested phase structure:

1. **Infrastructure & Extraction** - Set up Pages Functions, wrangler.toml, secrets. Build MLS listing extraction with tool_use. This validates the entire backend pattern before building UI.
   - Addresses: PDF upload, Claude extraction, admin key security
   - Avoids: Building UI before confirming the API proxy pattern works

2. **Admin UI & Offer Extraction** - Build the single-file admin page with form fields, confirmation checkboxes, and purchase contract extraction.
   - Addresses: Editable fields, confirmation gate, offer extraction
   - Avoids: Scope creep into addendum handling

3. **Calculations & Template** - Net-to-seller calculation engine, cost template with localStorage persistence, auto-ranking.
   - Addresses: Net-to-seller, cost template, auto-rank
   - Avoids: Premature publishing before calculations are solid

4. **Preview & Publish** - HTML template generation (based on Dava Dr design), preview in new tab, GitHub Contents API publishing.
   - Addresses: Preview, publish, responsive/print layout
   - Avoids: Publishing before all table-stakes features work

5. **Polish & Extensions** - Load existing pages, Claude strategic analysis draft, addendum merging.
   - Addresses: Differentiator features
   - Avoids: Scope creep into v1

**Phase ordering rationale:**
- Infrastructure first because the free plan CPU limit is a blocker that must be resolved immediately
- Extraction before UI because the tool_use schema defines what form fields exist
- Calculations before publishing because net-to-seller accuracy is the entire value proposition
- Publishing last among core features because it depends on everything else working
- Extensions deferred because they are not table stakes

**Research flags for phases:**
- Phase 1: Needs validation that Pages Functions can handle 10MB+ base64 payloads within memory limits
- Phase 2: May need iteration on tool_use schemas as real ARMLS/AAR PDFs reveal edge cases in Claude's extraction
- Phase 5: Addendum merging will need its own research spike on prompt engineering for selective field overlay

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | All technologies verified against official docs. Pattern proven by Tenet converter |
| Features | HIGH | Requirements clearly defined in PROJECT.md. Feature landscape is straightforward |
| Architecture | HIGH | Standard serverless proxy pattern. Embedded JSON for data persistence is battle-tested |
| Pitfalls | HIGH | CPU limit verified in Cloudflare docs. SHA requirement verified in GitHub docs. PDF extraction accuracy is a known challenge with documented mitigations |

## Gaps to Address

- **ARMLS PDF format variability:** Need to test extraction with 5-10 real ARMLS listing PDFs to refine the tool_use schema. Field names and positions may vary between listing types (residential, land, multi-family)
- **AAR contract versions:** Arizona Association of Realtors updates contract forms periodically. The extraction schema may need adjustments for different contract revisions
- **Cloudflare Pages deploy webhook:** Could potentially use a deploy hook to notify the admin when the page is live, avoiding the "wait 1-2 minutes" message. Needs investigation
- **Cost template defaults:** Need the team's standard closing cost breakdown (title fees, escrow, HOA prorations, commission splits) to set sensible defaults
