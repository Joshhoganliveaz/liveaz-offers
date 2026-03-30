# Phase 1: Form & Data Entry - Research

**Researched:** 2026-03-30
**Domain:** Vanilla HTML/CSS/JS single-file form application
**Confidence:** HIGH

## Summary

This phase builds a single-file HTML admin form for entering property details, multiple offers, a persistent cost template, and strategic analysis notes. The technology is vanilla HTML/CSS/JS with zero dependencies, matching the established Tenet converter pattern already used by the team. localStorage handles cost template persistence.

The primary complexity is in the dynamic offer card management (add/remove cards without losing sibling data) and the cost template's dynamic custom line items with localStorage persistence. Both are well-understood vanilla JS patterns. There are no library choices, no build tools, and no framework decisions to make.

**Primary recommendation:** Build as a single `index.html` file with embedded `<style>` and `<script>` blocks. Use semantic form elements, CSS Grid for property detail fields, and DOM manipulation via `insertAdjacentHTML` or `createElement` for dynamic offer cards. Store cost template as a JSON object in localStorage.

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Single scroll layout, top to bottom: Header > Cost Template (collapsed) > Property Details > Offers > Analysis Notes > Generate button
- Centered layout with max-width ~800px
- Cost template starts collapsed by default, saved values persist in localStorage
- Property detail short fields use a grid layout. Address and features stay full-width
- Form starts with one empty offer card pre-loaded
- "Add Offer" button to add more cards (typical 2-4 offers)
- Each offer card has numbered header with subtle left border accent
- Fields grouped by category: Buyer Info | Financial Terms | Timeline | Compensation | Financing | Special Terms
- Visual dividers between field groups inside each card
- Remove button triggers confirmation dialog before deletion
- Adding/removing offers does not affect data in other offer cards
- Cost template fields start blank (no pre-filled defaults). Team enters numbers once and localStorage remembers
- Custom line items: label + fixed dollar amount only (no percentage option)
- Running total at bottom of cost template
- "Reset" link with confirmation to clear saved cost template values
- Light, clean utility style: Background #F9FAFB, Cards #FFFFFF, Borders #E5E7EB, Text #111827, Accent Gold #C9953E
- Text-only header "Offer Comparison Tool" (no logo)
- System font stack (no Google Fonts)
- Generate Page button uses Gold #C9953E as primary CTA color
- Single-file HTML with embedded CSS + JS, no build tools, no framework, no dependencies

### Claude's Discretion
- Exact grid column count for property detail fields
- Input field sizes and spacing
- Offer card border/shadow styling details
- Responsive breakpoints (if any needed for the admin form)
- Textarea height for analysis notes and special terms

### Deferred Ideas (OUT OF SCOPE)
None. Discussion stayed within phase scope.
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| PROP-01 | User can enter property address (street, city, state, zip) | Full-width input fields in property section |
| PROP-02 | User can enter property details (beds, baths, sqft, year built, lot size, pool, garage) | CSS Grid layout for compact field arrangement |
| PROP-03 | User can enter list price | Standard number input with formatting |
| PROP-04 | User can enter notable features as free text | Full-width textarea |
| OFFR-01 | User can add offer with buyer name(s), agent name, brokerage | Buyer Info field group in offer card |
| OFFR-02 | User can enter financial terms: purchase price, down payment, loan type, earnest money | Financial Terms field group |
| OFFR-03 | User can enter timeline terms: COE date, inspection days, expiration | Timeline field group with date inputs |
| OFFR-04 | User can enter compensation terms: concessions, buyer agent comp | Compensation field group |
| OFFR-05 | User can enter financing details: pre-qual amount, lender | Financing field group |
| OFFR-06 | User can enter special terms as free text | Textarea in Special Terms group |
| OFFR-07 | User can flag offer as unrepresented buyer | Checkbox input in Buyer Info group |
| OFFR-08 | User can add multiple offers dynamically | DOM manipulation for offer card creation |
| OFFR-09 | User can remove an offer | Confirmation dialog + DOM removal |
| COST-01 | User can configure listing commission percentage | Number input in cost template section |
| COST-02 | User can configure title/escrow estimate | Number input in cost template section |
| COST-03 | User can configure home warranty amount | Number input in cost template section |
| COST-04 | User can configure TC fee | Number input in cost template section |
| COST-05 | User can add/remove custom line items with label and amount | Dynamic add/remove within cost template |
| COST-06 | Cost template persists in localStorage | JSON serialization to localStorage |
| ANLS-01 | User can enter strategic analysis notes | Full-width textarea in analysis section |
| ANLS-02 | Analysis text appears in generated comparison page | Data available for Phase 3 to read |
</phase_requirements>

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Vanilla HTML5 | N/A | Structure | Zero dependencies per project decision |
| Vanilla CSS3 | N/A | Styling | Embedded in single file, system fonts |
| Vanilla JavaScript | ES6+ | Interactivity | No framework, no build step |
| localStorage API | Web API | Cost template persistence | Built into every browser, no setup |

### Supporting
No external libraries. Everything is built-in browser APIs.

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Vanilla JS | Alpine.js or Petite Vue | Adds a dependency; locked decision is no dependencies |
| localStorage | IndexedDB | Overkill for a small JSON object |
| Inline styles | Tailwind CDN | Adds external dependency; locked decision says no |

**Installation:**
```bash
# No installation needed. Single HTML file.
```

## Architecture Patterns

### Recommended Project Structure
```
liveaz-offers/
├── 1960-e-dava-dr/
│   └── index.html          # Reference output page (existing)
├── index.html               # THE admin form (this phase builds this)
├── docs/
└── .planning/
```

### Pattern 1: Section-Based Single File Layout
**What:** All HTML, CSS, and JS in one file, organized by clear comment blocks.
**When to use:** Always for this project (locked decision).
**Example:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Offer Comparison Tool</title>
  <style>
    /* === RESET & BASE === */
    /* === LAYOUT === */
    /* === HEADER === */
    /* === COST TEMPLATE === */
    /* === PROPERTY DETAILS === */
    /* === OFFER CARDS === */
    /* === ANALYSIS === */
    /* === BUTTONS === */
  </style>
</head>
<body>
  <!-- Header -->
  <!-- Cost Template (collapsible) -->
  <!-- Property Details -->
  <!-- Offers Container -->
  <!-- Analysis Notes -->
  <!-- Generate Button -->
  <script>
    // === STATE ===
    // === COST TEMPLATE ===
    // === OFFER MANAGEMENT ===
    // === HELPERS ===
  </script>
</body>
</html>
```

### Pattern 2: Dynamic Offer Cards via Template Functions
**What:** A JS function that returns an offer card's HTML string, parameterized by offer number. Cards are appended to a container div.
**When to use:** For OFFR-08 (add offers) and OFFR-09 (remove offers).
**Example:**
```javascript
let offerCount = 0;

function createOfferCard() {
  offerCount++;
  const card = document.createElement('div');
  card.className = 'offer-card';
  card.dataset.offerNum = offerCount;
  card.innerHTML = `
    <div class="offer-header">
      <h3>Offer ${offerCount}</h3>
      <button type="button" class="btn-remove" onclick="removeOffer(this)">Remove</button>
    </div>
    <div class="field-group">
      <h4>Buyer Info</h4>
      <!-- fields here -->
    </div>
    <!-- more field groups -->
  `;
  document.getElementById('offers-container').appendChild(card);
}

function removeOffer(btn) {
  const card = btn.closest('.offer-card');
  const num = card.dataset.offerNum;
  if (confirm(`Remove Offer ${num}? This can't be undone.`)) {
    card.remove();
    renumberOffers();
  }
}

function renumberOffers() {
  document.querySelectorAll('.offer-card').forEach((card, i) => {
    card.querySelector('h3').textContent = `Offer ${i + 1}`;
  });
}
```

### Pattern 3: localStorage Cost Template Persistence
**What:** Save cost template values as a JSON object on every change. Load on page init.
**When to use:** For COST-06 (persistence).
**Example:**
```javascript
const STORAGE_KEY = 'liveaz-offer-costs';

function saveCostTemplate() {
  const data = {
    listingCommission: document.getElementById('listing-commission').value,
    titleEscrow: document.getElementById('title-escrow').value,
    homeWarranty: document.getElementById('home-warranty').value,
    tcFee: document.getElementById('tc-fee').value,
    customItems: getCustomItems()
  };
  localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
  updateCostTotal();
}

function loadCostTemplate() {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (!saved) return;
  const data = JSON.parse(saved);
  // populate fields from data...
}

function resetCostTemplate() {
  if (confirm('Reset all cost template values? This clears your saved defaults.')) {
    localStorage.removeItem(STORAGE_KEY);
    // clear all fields and custom items...
    updateCostTotal();
  }
}
```

### Pattern 4: Collapsible Section with CSS Transition
**What:** Click header to toggle section content visibility with smooth animation.
**When to use:** Cost template section (starts collapsed).
**Example:**
```css
.collapsible-content {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.3s ease;
}
.collapsible-content.open {
  max-height: 2000px; /* large enough to fit content */
}
```
```javascript
function toggleSection(header) {
  const content = header.nextElementSibling;
  const chevron = header.querySelector('.chevron');
  content.classList.toggle('open');
  chevron.classList.toggle('rotated');
}
```

### Anti-Patterns to Avoid
- **Using innerHTML for the entire offers container on every change:** Destroys user input in sibling cards. Always append/remove individual cards.
- **Storing offer data in JS state and re-rendering:** Unnecessary complexity. Read directly from DOM when needed (Phase 2/3). The form IS the state.
- **Using form submission/page reload:** This is a client-side-only tool. Use `type="button"` on all buttons except the generate CTA.
- **Forgetting `type="button"` on buttons inside a form:** Default type is "submit" which triggers page reload.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Currency formatting in inputs | Custom formatter | `Intl.NumberFormat` or simple toFixed(2) | Edge cases with cursor position, negative numbers |
| Date input | Custom date picker | Native `<input type="date">` | Browser handles validation, calendar popup |
| Confirmation dialogs | Custom modal | `window.confirm()` | Sufficient for an internal tool, zero code |
| Number validation | Regex parsing | `<input type="number" step="0.01">` with `parseFloat()` | Native validation + step handles decimals |

**Key insight:** This is an internal admin tool for a team of three. Native browser APIs (confirm, date inputs, number inputs) are perfectly appropriate and reduce code significantly.

## Common Pitfalls

### Pitfall 1: Offer Renumbering Breaks Field Names
**What goes wrong:** If offer cards use `name="offer-1-price"` and you renumber after deletion, the names no longer match the visual numbers, or vice versa.
**Why it happens:** Mixing display numbering with data identification.
**How to avoid:** Use a monotonically increasing ID for data attributes (`data-offer-id`) and only update the display number (the h3 text). When collecting data for Phase 2/3, iterate over `.offer-card` elements by DOM order, not by ID.
**Warning signs:** Offer data appears in wrong card after removing a middle offer.

### Pitfall 2: localStorage Quota or Availability
**What goes wrong:** localStorage is unavailable in private browsing on some older browsers, or quota is exceeded.
**Why it happens:** Edge case browser configurations.
**How to avoid:** Wrap localStorage calls in try/catch. The cost template JSON is tiny (< 1KB) so quota is not a real concern, but the try/catch is free insurance.
**Warning signs:** Console errors on page load.

### Pitfall 3: Form State Lost on Accidental Navigation
**What goes wrong:** User fills in 4 offers, accidentally clicks back or refreshes, loses everything.
**Why it happens:** Only cost template persists; offer data lives in the DOM only.
**How to avoid:** Add a `beforeunload` event listener that warns if there is unsaved data (any offer card with populated fields). This is a courtesy, not a persistence mechanism.
**Warning signs:** User complaints about lost work.

### Pitfall 4: Cost Template Running Total Shows NaN
**What goes wrong:** If any cost field is empty, `parseFloat('')` returns NaN, which poisons the sum.
**Why it happens:** Empty string is not zero.
**How to avoid:** Use `parseFloat(value) || 0` pattern for all numeric reads.
**Warning signs:** "Total Estimated Costs: $NaN" displayed.

### Pitfall 5: Dynamic Custom Items Not Saved to localStorage
**What goes wrong:** User adds custom cost items, refreshes, they're gone.
**Why it happens:** Save function only captures the four fixed fields.
**How to avoid:** `saveCostTemplate()` must also serialize the dynamic custom items array. Call save on every custom item add/remove/edit.
**Warning signs:** Custom items disappear on refresh while fixed fields persist.

## Code Examples

### Form Field Patterns for Offer Cards
```html
<!-- Number with dollar formatting hint -->
<label>Purchase Price
  <div class="input-prefix">
    <span class="prefix">$</span>
    <input type="number" name="purchase-price" step="1" min="0" placeholder="0">
  </div>
</label>

<!-- Percentage field -->
<label>Down Payment %
  <div class="input-suffix">
    <input type="number" name="down-pct" step="0.1" min="0" max="100" placeholder="0">
    <span class="suffix">%</span>
  </div>
</label>

<!-- Date field -->
<label>Close of Escrow
  <input type="date" name="coe-date">
</label>

<!-- Select dropdown -->
<label>Loan Type
  <select name="loan-type">
    <option value="">Select...</option>
    <option value="conventional">Conventional</option>
    <option value="fha">FHA</option>
    <option value="va">VA</option>
    <option value="cash">Cash</option>
    <option value="other">Other</option>
  </select>
</label>

<!-- Checkbox -->
<label class="checkbox-label">
  <input type="checkbox" name="unrepresented">
  Unrepresented Buyer
</label>
```

### CSS Grid for Property Details
```css
.property-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px 16px;
}

@media (max-width: 600px) {
  .property-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
```

### Cost Template Total Calculation
```javascript
function updateCostTotal() {
  const listingComm = parseFloat(document.getElementById('listing-commission').value) || 0;
  const titleEscrow = parseFloat(document.getElementById('title-escrow').value) || 0;
  const warranty = parseFloat(document.getElementById('home-warranty').value) || 0;
  const tcFee = parseFloat(document.getElementById('tc-fee').value) || 0;

  let customTotal = 0;
  document.querySelectorAll('.custom-item-amount').forEach(input => {
    customTotal += parseFloat(input.value) || 0;
  });

  // Note: listing commission is a %, not a dollar amount
  // The dollar total here excludes commission (it's applied per-offer in Phase 2)
  const total = titleEscrow + warranty + tcFee + customTotal;

  document.getElementById('cost-total').textContent =
    `Total Fixed Costs: $${total.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
  document.getElementById('commission-note').textContent =
    listingComm ? `+ ${listingComm}% listing commission (calculated per offer)` : '';
}
```

### Beforeunload Guard
```javascript
window.addEventListener('beforeunload', function(e) {
  const hasOfferData = document.querySelectorAll('.offer-card input').length > 0 &&
    Array.from(document.querySelectorAll('.offer-card input'))
      .some(input => input.value.trim() !== '');
  if (hasOfferData) {
    e.preventDefault();
    e.returnValue = '';
  }
});
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| jQuery DOM manipulation | Vanilla JS querySelector/querySelectorAll | Widespread ~2018+ | No jQuery dependency needed |
| Custom CSS resets | `box-sizing: border-box` + minimal reset | Standard practice | Simpler, fewer surprises |
| Custom dropdowns/date pickers | Native HTML5 form elements | Browser support matured | Less code, accessible by default |
| Float-based layouts | CSS Grid + Flexbox | Widely supported since ~2018 | Cleaner, more predictable |

**Deprecated/outdated:**
- jQuery: Unnecessary for this scope; vanilla JS covers everything needed
- CSS float layouts: Grid and Flexbox handle all layout needs

## Open Questions

1. **Cost template running total: should it include commission?**
   - What we know: Listing commission is a percentage, other items are fixed dollar amounts. The total can't include commission without a reference price.
   - What's unclear: Should the total show "fixed costs" only, or should there be a note about commission being separate?
   - Recommendation: Show fixed dollar total plus a note line showing the commission percentage (calculated per-offer in Phase 2). Example pattern included in code examples above.

2. **Should the form file live at project root or in its own directory?**
   - What we know: The output page reference lives in `1960-e-dava-dr/index.html`. The admin form is a separate tool.
   - What's unclear: Where exactly to place the file.
   - Recommendation: Place at project root as `index.html` since this is the primary tool. Output pages can live in address-named subdirectories.

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Manual browser testing (no automated test framework) |
| Config file | none |
| Quick run command | Open `index.html` in browser |
| Full suite command | Manual walkthrough of all form interactions |

### Phase Requirements to Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| PROP-01 | Enter property address fields | manual | Open file, fill address fields | N/A |
| PROP-02 | Enter property details in grid | manual | Fill beds/baths/sqft/etc fields | N/A |
| PROP-03 | Enter list price | manual | Type number in list price field | N/A |
| PROP-04 | Enter notable features | manual | Type in features textarea | N/A |
| OFFR-01 | Add offer with buyer info | manual | Fill buyer name, agent, brokerage | N/A |
| OFFR-02 | Enter financial terms | manual | Fill price, down payment, loan type, earnest | N/A |
| OFFR-03 | Enter timeline terms | manual | Fill COE date, inspection days, expiration | N/A |
| OFFR-04 | Enter compensation terms | manual | Fill concessions, buyer agent comp | N/A |
| OFFR-05 | Enter financing details | manual | Fill pre-qual amount, lender | N/A |
| OFFR-06 | Enter special terms | manual | Type in special terms textarea | N/A |
| OFFR-07 | Flag unrepresented buyer | manual | Check/uncheck checkbox | N/A |
| OFFR-08 | Add multiple offers | manual | Click "Add Offer" multiple times, verify data isolation | N/A |
| OFFR-09 | Remove an offer | manual | Click remove, confirm dialog, verify other cards intact | N/A |
| COST-01 | Configure listing commission % | manual | Enter percentage value | N/A |
| COST-02 | Configure title/escrow | manual | Enter dollar amount | N/A |
| COST-03 | Configure home warranty | manual | Enter dollar amount | N/A |
| COST-04 | Configure TC fee | manual | Enter dollar amount | N/A |
| COST-05 | Add/remove custom line items | manual | Add items, fill label+amount, remove one | N/A |
| COST-06 | Cost template persists | manual | Fill costs, refresh page, verify values restored | N/A |
| ANLS-01 | Enter analysis notes | manual | Type in analysis textarea | N/A |
| ANLS-02 | Analysis available for output | manual-only | Verified in Phase 3 when generate is built | N/A |

### Sampling Rate
- **Per task commit:** Open `index.html` in browser, verify new functionality works
- **Per wave merge:** Full manual walkthrough: fill all fields, add 3 offers, remove middle one, verify cost persistence on refresh
- **Phase gate:** Complete walkthrough covering all 21 requirements before marking phase done

### Wave 0 Gaps
None. This is a single HTML file with no test framework. Validation is manual browser testing, which is appropriate for a no-dependency, no-build-step vanilla HTML project used by a team of three.

## Sources

### Primary (HIGH confidence)
- MDN Web Docs: localStorage API, DOM manipulation, CSS Grid, HTML5 form elements (standard web platform knowledge)
- Existing project reference: `1960-e-dava-dr/index.html` (output page design tokens)
- Existing project reference: `Tenet Equity/tenet-converter.html` (single-file admin tool pattern)

### Secondary (MEDIUM confidence)
- CONTEXT.md locked decisions (direct user input, fully trusted)

### Tertiary (LOW confidence)
- None. All findings are based on well-established web platform APIs.

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - vanilla HTML/CSS/JS is the most stable, well-documented stack possible
- Architecture: HIGH - single-file pattern already proven in team's Tenet converter tool
- Pitfalls: HIGH - common DOM manipulation pitfalls are well-known and documented

**Research date:** 2026-03-30
**Valid until:** Indefinite (vanilla web platform APIs are stable)
