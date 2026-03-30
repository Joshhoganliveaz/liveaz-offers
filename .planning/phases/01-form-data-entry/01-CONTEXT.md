# Phase 1: Form & Data Entry - Context

**Gathered:** 2026-03-30
**Status:** Ready for planning

<domain>
## Phase Boundary

Complete input UI for property details, multiple offers, configurable cost template, and strategic analysis notes. All data entry is manual (no PDF parsing). Cost template persists in localStorage. No calculations or output generation in this phase.

</domain>

<decisions>
## Implementation Decisions

### Form Layout & Flow
- Single scroll layout, top to bottom: Header > Cost Template (collapsed) > Property Details > Offers > Analysis Notes > Generate button
- Centered layout with max-width ~800px
- Cost template section starts collapsed by default (click to expand). Saved values persist in localStorage so most sessions the team won't touch it
- Property detail short fields (beds, baths, sqft, year built, lot size, pool, garage) use a grid layout (2x2 or 3-column). Address and features stay full-width

### Offer Card Interaction
- Form starts with one empty offer card pre-loaded
- Team clicks "Add Offer" button to add more cards
- Typical scenario: 2-4 offers per listing
- Each offer card has numbered header with subtle left border accent (e.g., "Offer 1", "Offer 2") for visual distinction
- Fields grouped by category within each card: Buyer Info | Financial Terms | Timeline | Compensation | Financing | Special Terms
- Visual dividers between field groups inside each card
- Remove button on each offer card triggers a confirmation dialog ("Remove Offer 2? This can't be undone.") before deletion
- Adding/removing offers does not affect data in other offer cards

### Cost Template UX
- No pre-filled defaults. Fields start blank on first use. Team enters their numbers once and localStorage remembers them
- Custom line items: label + fixed dollar amount (no percentage option)
- Shows a running total ("Total Estimated Costs: $X") at bottom of cost template section
- "Reset" link with confirmation to clear all saved cost template values from localStorage
- Fields: listing commission %, title/escrow estimate, home warranty amount, TC fee, plus dynamic custom line items with add/remove

### Visual Style & Branding
- Light, clean utility style (not branded like the output page)
- Background: #F9FAFB (light gray), Cards: #FFFFFF, Borders: #E5E7EB, Text: #111827, Accent: Gold (#C9953E)
- Text-only header: "Offer Comparison Tool" (no logo, this is an internal admin tool)
- System font stack (no Google Fonts dependency for the admin page)
- Generate Page button uses Gold (#C9953E) as the primary CTA color

### Claude's Discretion
- Exact grid column count for property detail fields
- Input field sizes and spacing
- Offer card border/shadow styling details
- Responsive breakpoints (if any needed for the admin form)
- Textarea height for analysis notes and special terms

</decisions>

<specifics>
## Specific Ideas

- Matches the Tenet converter single-file HTML pattern the team already uses
- The admin form is deliberately distinct from the generated output page. Admin = light utility style; output = branded Dava Dr design
- Single-file HTML with embedded CSS + JS, no build tools, no framework, no dependencies

</specifics>

<code_context>
## Existing Code Insights

### Reusable Assets
- `1960-e-dava-dr/index.html`: Reference for the OUTPUT page design tokens (Slate, Cream, Canyon, Gold, Olive, Playfair Display + DM Sans). The admin form does NOT use these tokens, but the Generate button in Phase 3 will produce pages matching this design
- `~/Projects/Tenet Equity/tenet-converter.html`: Reference for the single-file admin tool pattern (embedded CSS/JS, dark theme, file handling, export buttons)

### Established Patterns
- Single-file HTML with no external dependencies (Tenet converter pattern)
- localStorage for persisting user preferences
- Collapsible sections with CSS transitions (used in Tenet converter's privacy infographic)

### Integration Points
- Phase 2 will add real-time net-to-seller calculations reading from the same form fields
- Phase 3 will add a template function that reads all form data and generates the branded output page

</code_context>

<deferred>
## Deferred Ideas

None. Discussion stayed within phase scope.

</deferred>

---

*Phase: 01-form-data-entry*
*Context gathered: 2026-03-30*
