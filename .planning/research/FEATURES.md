# Feature Landscape

**Domain:** Real estate offer comparison admin tool with AI-powered PDF extraction
**Researched:** 2026-03-30

## Table Stakes

Features users expect. Missing = product feels incomplete or unusable for the team.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| PDF upload for listing data | Core input method. Team has PDFs from ARMLS, not structured data | Medium | Claude API with tool_use for structured extraction from 1-3 page MLS PDFs |
| PDF upload for purchase contracts | Core input method. AAR contracts are 10-30 pages with critical terms scattered throughout | High | Most complex extraction task. Financing, contingencies, dates, costs all in different sections |
| Side-by-side offer comparison table | Every competitor (HLM, Jointly, Excel worksheets) has this. Sellers expect columnar layout | Medium | The Dava Dr reference already nails this with section groupings (Buyer Info, Price/Financing, Timeline, Costs, Additional Terms) |
| Net-to-seller calculation | The single most important number for sellers. Price alone is misleading when concessions, commissions, and costs vary | Medium | Must account for: purchase price, seller concessions, buyer agent comp (seller-paid), listing commission, home warranty, title/escrow fees, HOA transfer fees |
| Offer ranking by net proceeds | Sellers and agents fixate on price. Auto-ranking by net forces apples-to-apples comparison | Low | Sort cards and highlight best column. Already in the Dava Dr reference |
| Editable extracted fields | Claude will misread PDF data. Six-figure decisions demand human verification | Medium | Every field must be editable. This is the confirmation checkbox gate from PROJECT.md |
| Branded, professional output | Sellers share these with family, attorneys, financial advisors. Ugly spreadsheets undermine agent credibility | Low | Dava Dr design already exists. Playfair Display + DM Sans, Slate/Cream/Canyon/Gold/Olive tokens |
| Mobile-responsive output page | Sellers view on phones. Non-negotiable in 2026 | Low | Reference page already has responsive breakpoints |
| Print-optimized layout | Sellers print for kitchen-table discussions. Agents print for listing presentations | Low | @page landscape, print-color-adjust already in reference |
| Mortgage payoff input on output page | Seller proceeds = net minus payoff. Every seller's situation is different | Low | Interactive payoff bar already in reference. Recalculates all cards in real-time |

## Differentiators

Features that set this apart from Excel worksheets, HLM, and Jointly. Not expected, but create competitive advantage for Live AZ Co.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| AI-powered PDF extraction (Claude) | No manual data entry. Upload contract, get structured data. HLM and Jointly require manual offer entry. Excel worksheets are entirely manual | High | This is THE differentiator. Competitors make agents type every field. Claude reads the PDF and pre-fills. The confirmation checkbox gate handles accuracy |
| Strategic analysis narrative | Claude drafts a multi-counter strategy analysis comparing all offers. No competitor generates written analysis | Medium | The Dava Dr example shows rich strategic commentary (financing strength analysis, gap coverage comparison, LTV analysis). Agents can edit before publishing |
| Addendum/counter offer merging | After initial offers, counter offers and addendums modify terms. Claude merges revised terms into existing offer cards with "Revised" badges | High | Real-world differentiator. Multiple offer situations involve 2-3 rounds of counters. Competitors require re-entering everything |
| One-click publish to branded URL | From admin tool to live URL in one click. No file management, no FTP, no asking a developer | Medium | GitHub Contents API commits HTML, Cloudflare auto-deploys. The team gets a shareable URL immediately |
| Cost template with localStorage persistence | Common seller costs (title policy, escrow fees, HOA fees) are roughly the same for most transactions. Save once, reuse | Low | Saves 5 minutes per comparison. Configurable per-listing if needed |
| Embedded JSON for round-tripping | Generated page carries its own data. Load Existing dropdown reads data back from published pages | Medium | No database needed. The HTML file IS the data store. GitHub provides version history for free via git commits |
| Expandable offer detail cards | Net summary cards that expand to show financing breakdown on tap/click. Progressive disclosure instead of information overload | Low | Already in reference. Mobile-friendly pattern that respects cognitive load |
| Buyer financing strength indicators | Automated assessment of financing risk (pre-qual vs loan amount gaps, LTV analysis, down payment percentage) with color-coded tags | Medium | Green/Gold/Red tags in reference (Strong/Moderate/Tight). Helps sellers evaluate beyond just price |
| Best/worst value highlighting | Green for strongest terms, red for weakest. Immediate visual scanning across offers | Low | best-val/worst-val classes in reference. Applied per-row based on which offer is strongest on each dimension |
| Load and update existing pages | Team can re-open a published page, add new offers or update terms, and re-publish. Handles the iterative nature of negotiations | Medium | Requires parsing embedded JSON from published HTML. Critical for multi-round counter offers |

## Anti-Features

Features to explicitly NOT build. These add complexity without proportional value for a team of three.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| User authentication / login system | Team of three who coordinate verbally. Admin page lives at `/_admin/` which is not publicly linked. Auth adds complexity, password reset flows, and friction for a tool used 2-4 times per month | Rely on obscurity + API key header validation. If security becomes a concern later, add Cloudflare Access (zero code changes) |
| Multi-user simultaneous editing | Team never edits the same comparison at the same time. They coordinate via text/call. Real-time sync (WebSockets, CRDT) is massive complexity for zero real-world benefit | Last-save-wins is fine. Team of three with verbal coordination |
| Offer change history / audit trail | No regulatory requirement to track changes. The published page is the record. If the team needs to see what changed, they compare the before/after pages | Keep it simple. If audit trail becomes needed, git history of the published HTML files provides it for free |
| Email/SMS notifications on publish | Team texts each other the URL. Adding notification infrastructure (SendGrid, Twilio) for a team of three is over-engineering | Copy URL to clipboard on publish. Team handles distribution |
| Lofty CRM integration | Separate system with its own complexity. The offer comparison is a standalone deliverable, not a CRM workflow. Integration coupling creates maintenance burden | Keep systems decoupled. The URL is the integration point (paste it into Lofty notes) |
| Automatic offer ranking commentary | Claude's strategic analysis handles this better than rules-based ranking logic. Hard-coding "Offer X is best because..." leads to wrong conclusions in edge cases | Claude drafts analysis with full context. Team edits the narrative before publishing |
| Buyer/seller portal with logins | Sellers receive a URL. They do not need accounts, dashboards, or notification preferences. Portal complexity is enormous for marginal value | Static branded page at a clean URL. Sellers bookmark or save the link |
| Database persistence | Cloudflare D1 or Supabase would add infrastructure, migrations, backup concerns. For a tool generating 2-4 pages per month, the HTML files in the git repo ARE the persistence layer | Embedded JSON in generated HTML + GitHub repo as storage. Git provides versioning for free |
| Automated MLS data feeds | ARMLS API access requires broker approval, ongoing fees, and compliance. PDF upload is simpler, works now, and handles the 2-4 listings per month volume | Manual PDF upload. The 30 seconds to upload a PDF is not the bottleneck |
| Offer submission portal for buyer agents | Building a portal where buyer agents submit offers directly adds enormous scope (account creation, form validation, notification, security). Live AZ Co receives offers via email like every other brokerage | Buyer agents email offers. Team uploads PDFs to the admin tool |

## Feature Dependencies

```
PDF Upload (Listing) ─────────────────────────────────────┐
                                                          v
PDF Upload (Contract) ──> Field Extraction ──> Editable Fields ──> Net Calculation ──> Ranking
                              |                      |                                    |
                              v                      v                                    v
                     Confirmation Gate         Cost Template                    Strategic Analysis
                                                                                        |
                                                                                        v
                                                                              Preview ──> Publish
                                                                                            |
                                                                                            v
                                                                              Load Existing (round-trip)
                                                                                            |
                                                                                            v
                                                                              Addendum Upload ──> Merge
```

Key dependency chains:
- Net-to-seller calculation requires: extracted fields + cost template + confirmation
- Strategic analysis requires: all offers extracted + net calculations complete
- Publish requires: preview approval (team confirms output looks right)
- Load Existing requires: published page with embedded JSON
- Addendum merge requires: existing offer card to attach to

## MVP Recommendation

Prioritize in this order:

1. **PDF extraction pipeline** (listing + contract) with editable fields and confirmation checkboxes. This is the core value proposition. Without accurate extraction, nothing else matters.

2. **Net-to-seller calculation** with configurable cost template. The single number sellers care about most.

3. **Side-by-side comparison table** with ranking, best/worst highlighting, and net summary cards. The visual output that replaces Excel spreadsheets.

4. **One-click publish** to Cloudflare Pages via GitHub API. Removes Josh as bottleneck.

5. **Strategic analysis** via Claude. Differentiator that no competitor offers.

Defer to v2:
- **Addendum/counter offer merging**: Complex feature that requires solid extraction foundation first. For v1, team can re-upload the full revised contract and replace the offer card
- **Load Existing for editing**: For v1, team can create a new page if terms change significantly. Add round-tripping once the publish pipeline is proven

## Sources

- HomeLight Listing Management (HLM/Disclosures.io): Side-by-side offer comparison, manual offer entry, seller sharing via custom links
- Jointly: Centralized offer management, auto-generated comparisons, client sharing
- Dotloop: Free Seller's Estimate Net Sheet with up to 3-offer side-by-side comparison
- Multiple Offer Worksheets (Excel/Google Sheets): Industry standard is 5-column comparison
- NAR Multiple Offers guidance: Present all written offers, avoid fixating on price alone
- Existing reference: 1960 E Dava Dr comparison page (in-repo) with 5-offer comparison
- Existing pattern: Tenet Equity converter (single-file HTML admin tool the team already uses)
