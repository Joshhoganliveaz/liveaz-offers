# Offer Comparison Admin Tool

**Date:** 2026-03-30
**Status:** Approved
**Repo:** `~/Projects/_deliverables/liveaz-offers/`
**Live URL:** offers.liveazco.com

## Problem

Creating offer comparison pages for sellers requires Josh, Claude Code, and raw HTML skills. Jacqui and Suzie cannot create or update these pages independently, even though they coordinate with sellers daily. This tool removes that bottleneck.

## Solution

A self-service admin tool at `offers.liveazco.com/_admin/` where the team uploads PDFs (MLS listing + purchase contracts), reviews parsed data, and publishes branded offer comparison pages with one click.

## User Flow

1. Open `offers.liveazco.com/_admin/`
2. (Optional) Load an existing page to edit via "Load Existing" dropdown
3. Upload the **ARMLS MLS listing PDF**. Claude extracts: address, beds/baths/sqft, list price, year built, subdivision, lot size, pool, garage, notable features. Fields populate a property summary card, each editable with a confirmation checkbox.
4. Upload **purchase contract PDFs** (one or more). Each PDF creates an offer card with extracted fields. Additional addendums can be attached to an existing offer card via "Add Addendum" and Claude merges the terms (counter offer overrides original).
5. **Review with checkboxes.** Each key field per offer (purchase price, down payment, concessions, buyer agent comp, earnest money, close date, pre-qual amount) has an editable input + a checkbox. All checkboxes must be checked before publish unlocks.
6. **Net-to-seller calculations** auto-compute in real-time using the cost template + each offer's terms. Offers rank by net, highest first.
7. **Strategic analysis** text area pre-filled by Claude's draft (a separate `/api/analyze` call made after all offers are parsed and confirmed, sending the structured offer data for comparative analysis). Team edits before publishing.
8. **Preview** opens the generated page in a new tab.
9. **Publish** (button disabled until all checkboxes confirmed). Commits HTML to GitHub repo via Pages Function, Cloudflare auto-deploys. Shows the live URL on success.

## Architecture

### Admin Page (`_admin/index.html`)

Single self-contained HTML file. No build tools, no framework. Embedded CSS + JS. Matches the Tenet converter pattern.

**Layout (top to bottom):**

- Header with Live AZ Co branding
- Cost Template Panel (collapsible, saved in localStorage)
  - Listing commission %
  - Title/escrow estimate
  - Home warranty amount
  - TC fee
  - Additional line items (flexible)
- Load Existing dropdown (fetches published slugs from GitHub)
- MLS Listing Upload drop zone
- Property summary card (parsed fields + checkboxes)
- Offer Upload drop zone (accepts multiple PDFs, each creates an offer card)
  - Per-offer: "Add Addendum" zone, "Remove Offer" button
  - "Add Another Offer" button for additional uploads
- Net-to-Seller ranked summary (auto-calculated, real-time updates)
- Strategic Analysis text area (Claude-drafted, editable)
- URL Slug field (auto-generated from address, editable)
- Preview button
- Publish button (disabled until all checkboxes confirmed)

### Cloudflare Pages Functions (`/functions/api/`)

Two endpoints, deployed automatically with the site. Secrets stored in Cloudflare dashboard.

**`POST /api/parse`**
- Receives: PDF file + type flag (`mls_listing` or `purchase_contract`)
- Sends PDF to Claude API with structured extraction prompt
- Returns: JSON with extracted fields
- If a field cannot be extracted, returns `null` for that field

**`POST /api/publish`**
- Receives: generated HTML string + URL slug
- Uses GitHub Contents API to create or update `{slug}/index.html` in the repo
- Returns: live URL on success
- If slug already exists, overwrites (update flow)

**`POST /api/analyze`**
- Receives: structured offer data (all parsed offers + property details)
- Sends to Claude with a prompt to draft a strategic analysis comparing the offers (strengths, weaknesses, net spread, financing risk, recommended negotiation strategy)
- Returns: analysis text as a string

**`GET /api/listings`**
- Fetches directory listing from GitHub repo to populate "Load Existing" dropdown
- Returns: array of existing slugs

### Secrets (configured once in Cloudflare dashboard)

- `ANTHROPIC_API_KEY` for Claude PDF parsing
- `GITHUB_TOKEN` fine-grained PAT scoped to liveaz-offers repo with `contents:write`
- `ADMIN_KEY` a shared secret the admin page sends as an `X-Admin-Key` header on all API requests. Pages Functions reject requests without a valid key. Prevents API cost abuse from anyone who discovers the endpoint URLs. The admin page stores this key in its source (acceptable since the admin page itself is the trust boundary).

### Security

- All Pages Function endpoints validate the `X-Admin-Key` header before processing.
- CORS: `Access-Control-Allow-Origin` restricted to `https://offers.liveazco.com` on all endpoints. Requests from other origins are rejected.

## PDF Parsing

### MLS Listing Extraction

Claude receives the ARMLS listing PDF and returns:

```json
{
  "address": "1960 E Dava Drive",
  "city": "Tempe",
  "state": "AZ",
  "zip": "85283",
  "beds": 4,
  "baths": 3,
  "sqft": 2855,
  "listPrice": 850000,
  "yearBuilt": 1994,
  "subdivision": "Oasis at Anozira",
  "lotSize": "7,200 sqft",
  "pool": true,
  "garageSpaces": 3,
  "features": "Pool & Spa, 3-Car Garage"
}
```

### Purchase Contract Extraction

Claude receives the AAR purchase contract (and any grouped addendums) and returns:

```json
{
  "buyerNames": "Gittings / Jabjiniak",
  "buyerAgentName": "Cassandra Marrujo",
  "buyerAgentBrokerage": "Realty ONE",
  "purchasePrice": 865000,
  "downPaymentAmount": 215000,
  "downPaymentPercent": 24.9,
  "loanType": "Conventional",
  "earnestMoney": 8650,
  "closeOfEscrow": "2026-04-29",
  "concessionsAmount": 0,
  "buyerAgentCompAmount": 25950,
  "buyerAgentCompPercent": 3.0,
  "homeWarranty": false,
  "homeWarrantyAmount": 0,
  "inspectionPeriodDays": 10,
  "preQualAmount": 650000,
  "preQualLender": "Guild Mortgage",
  "offerExpiration": "2026-03-29T20:00:00",
  "specialTerms": null,
  "isUnrepresented": false
}
```

### Addendum Handling

When addendums are grouped with a purchase contract, Claude processes all documents together. Counter offer terms override the original contract where they conflict. The prompt instructs Claude to return the final effective terms (the complete merged object, not just changed fields).

**Example:** A purchase contract lists close of escrow as April 30 and purchase price as $850,000. A counter offer changes the price to $865,000 but does not mention the close date. Claude returns the full object with `purchasePrice: 865000` and `closeOfEscrow: "2026-04-30"` (unchanged).

### Claude API Integration

**Model:** Claude Sonnet (claude-sonnet-4-20250514). Balances extraction accuracy with cost and speed. Sonnet handles structured document extraction reliably; Opus is unnecessary for form parsing.

**Structured output:** Use `tool_use` with a JSON schema definition for each extraction type (MLS listing, purchase contract). This guarantees valid JSON structure and eliminates malformed response handling. The tool definitions mirror the JSON schemas above.

**Request format:** PDF sent as a base64-encoded `document` content block in the messages API. The admin page reads the file as an ArrayBuffer, converts to base64, and sends to the Pages Function. The function forwards to Claude's API.

**File size:** ARMLS listings are typically 1-3 pages (~200KB). AAR purchase contracts are 10-30 pages (~1-5MB). Cloudflare Pages Functions support up to 25MB request bodies, which is more than sufficient. The admin page validates file size client-side and rejects files over 10MB with an error message.

**Token budget:** Max output tokens set to 1,024 (extraction responses are small JSON). Max input will vary by PDF size but AAR contracts are well within Claude's context window.

**Error handling:**
- **Parsing failure** (Claude returns unexpected structure): Show inline error on the offer card with "Could not parse this PDF. Please enter details manually." All fields left blank with checkboxes unchecked.
- **Rate limit (429):** Retry once after 2 seconds. If still rate-limited, show "Service busy, please try again in a moment."
- **Network/server error (5xx):** Show "Something went wrong. Please try again." with a retry button on the affected card.
- **Wrong file type:** If Claude's response indicates the PDF is not an MLS listing or purchase contract, show "This doesn't look like a [listing/purchase contract]. Please check the file."

### Loading States

- **During PDF parse:** Offer card shows a skeleton/shimmer state with "Parsing [filename]..." text. Other cards remain interactive.
- **During publish:** Publish button shows a spinner with "Publishing..." text. Disabled until complete.
- **During page load (Load Existing):** Form shows a loading indicator while fetching from GitHub.

## Net-to-Seller Calculation

Runs client-side in real-time as fields change.

```
Net to Seller = Purchase Price
  - (Listing Commission % x Purchase Price)   // from cost template
  - Buyer Agent Comp                           // use dollar amount if present; else compute from % x purchase price
  - Concessions                                // from offer
  - Title/Escrow Estimate                      // from cost template
  - Home Warranty                              // from offer if specified; else from cost template (not additive)
  - TC Fee                                     // from cost template
  - Additional Template Line Items             // from cost template
```

**Ranking:** Offers auto-sort by net, highest first. Each card shows dollar difference vs. top offer.

**Unrepresented buyers:** If no buyer's agent, the tool flags it and allows adding an additional listing-side commission line item.

**Mortgage payoff:** Optional field. If entered, the generated page includes the interactive payoff calculator where the seller adjusts the number.

## Generated Page Template

The output mirrors the existing Dava Dr page design. A JavaScript template function in the admin page produces:

- Header with Live AZ Co logo, contact info, gold accent bar
- Property banner with address, features, list price
- Ranked offer cards with expandable details (tap to reveal)
- Full comparison table with all offers side-by-side (best/worst values highlighted)
- Financing strength section (pre-qual amounts, headroom %, down payment comparison)
- Interactive mortgage payoff calculator (if payoff provided)
- Strategic analysis section
- Footer with branding and generation timestamp
- "Copy link" button (no-print) for quickly sharing the URL with the seller
- Hidden `<script type="application/json" id="offer-data">` tag with the structured JSON (including cost template used) for round-tripping back to admin

**Design tokens:** Same as Dava Dr page. Slate (#3A3D42), Cream (#FAF7F2), Canyon (#8B6F5C), Gold (#C9953E), Olive (#5C6B4F), Charcoal (#2C2C2C). Fonts: Playfair Display (headings), DM Sans (body).

**Responsive:** Fully responsive with mobile card stacking and swipe hints.

**Print:** Landscape layout with print-optimized styles.

## Updating Existing Pages

- "Load Existing" dropdown at top of admin page lists all published slugs (via `/api/listings`)
- Selecting a slug fetches the page HTML, extracts the embedded JSON from the hidden script tag, and repopulates the form
- **Pre-admin pages** (like the existing Dava Dr page) do not have embedded JSON and cannot be loaded. The dropdown only shows pages that were created through the admin tool. This is acceptable for v1.
- Team makes changes (upload new offer, remove withdrawn offer, edit counter offer terms)
- Re-confirm checkboxes on changed fields
- Re-publish to the same slug (overwrites existing)

### URL Slug Validation

- Auto-generated from address: lowercase, spaces replaced with hyphens, special characters stripped (e.g., "1960 E Dava Dr" becomes `1960-e-dava-dr`)
- Max length: 80 characters
- Allowed characters: lowercase letters, numbers, hyphens
- When publishing to an existing slug that was NOT loaded via "Load Existing," show a confirmation dialog: "A page already exists at this URL. Overwrite it?"

## Key Decisions

1. **Single-file admin page (no framework):** Matches the team's existing Tenet converter pattern. No build tools means any developer can maintain it. The trade-off is no component reuse, but the admin page is a one-off tool, not a growing app.

2. **Pages Functions over standalone Worker:** Same repo, same deployment, zero additional infrastructure. A standalone Worker would require separate wrangler config and deployment pipeline for two tiny endpoints.

3. **Checkbox confirmation gate over trust-and-publish:** Claude will occasionally misread a number from a PDF. Since these pages go directly to sellers making six-figure decisions, a mandatory field-by-field confirmation step is non-negotiable. Checkboxes are the simplest UX that enforces this.

4. **Embedded JSON for round-tripping over database:** No database to maintain. The generated page carries its own data. The trade-off is no centralized listing of all offers across properties, but that's not a current need.

5. **No auth on admin page (v1):** The `_admin` path is not linked publicly. Discoverability is sufficient protection for a team of three. A simple password gate can be added later if needed.

## Out of Scope (v1)

- Authentication/password protection on admin page
- Multi-user simultaneous editing
- Historical tracking of offer changes over time
- Email/SMS notification when a page is published
- Automatic offer ranking commentary beyond Claude's initial draft
- Integration with Lofty CRM
