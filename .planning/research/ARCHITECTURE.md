# Architecture Patterns

**Domain:** Single-file HTML admin tool with serverless API backend
**Project:** Offer Comparison Admin Tool
**Researched:** 2026-03-30

## Recommended Architecture

Three-layer architecture: a single-file HTML admin page (client), Cloudflare Pages Functions (API layer), and external services (Claude API, GitHub API). No database. State lives in localStorage (cost template, draft offers) and in the generated HTML files themselves (embedded JSON for round-tripping).

```
+-----------------------+       +---------------------------+       +------------------+
|   _admin/index.html   | ----> |  /functions/api/          | ----> |  Claude API      |
|   (Single-file HTML)  |       |  extract-listing.ts       |       |  (PDF parsing)   |
|                       |       |  extract-offer.ts         |       +------------------+
|  - PDF upload UI      |       |  generate-analysis.ts     |
|  - Field editing      |       |  publish.ts               | ----> +------------------+
|  - Net-to-seller calc |       |  load.ts                  |       |  GitHub API      |
|  - Preview/Publish    |       |  _middleware.ts            |       |  (Contents API)  |
|  - localStorage       |       +---------------------------+       +------------------+
+-----------------------+
         |
         | generates
         v
+-------------------------+
|  /[address]/index.html  |
|  (Public offer page)    |
|  - Embedded JSON data   |
|  - Dava Dr design       |
+-------------------------+
```

### Component Boundaries

| Component | Responsibility | Communicates With |
|-----------|---------------|-------------------|
| `_admin/index.html` | All UI: PDF upload, field editing, checkbox confirmation, net-to-seller calculation, preview, publish trigger. Cost template management. Load existing pages | Pages Functions via `fetch()` |
| `functions/api/_middleware.ts` | Auth gate: validates `X-Admin-Key` header on every API request. CORS enforcement (origin = offers.liveazco.com) | Runs before all `/api/*` handlers |
| `functions/api/extract-listing.ts` | Receives base64 MLS PDF, sends to Claude with tool_use schema for listing data, returns structured JSON | Claude Messages API |
| `functions/api/extract-offer.ts` | Receives base64 purchase contract PDF, sends to Claude with tool_use schema for offer terms, returns structured JSON | Claude Messages API |
| `functions/api/generate-analysis.ts` | Receives all confirmed offer data, sends to Claude for strategic analysis draft text | Claude Messages API |
| `functions/api/publish.ts` | Receives complete page data (listing, offers, analysis, template), base64-encodes HTML, commits to GitHub repo via Contents API | GitHub REST API |
| `functions/api/load.ts` | Lists existing offer pages from repo and/or fetches a specific page's embedded JSON | GitHub REST API |
| `[address]/index.html` | Static public page. Self-contained HTML with embedded `<script type="application/json" id="page-data">` block carrying all offer data for round-tripping back into admin | None (static) |

### Data Flow

#### Flow 1: Create New Offer Page

```
1. Team uploads MLS listing PDF (FileReader -> base64)
   _admin/index.html  --->  POST /api/extract-listing
                                  |
                                  v
                            Claude API (tool_use with tool_choice forced)
                            Returns: { address, listPrice, beds, baths, sqft, ... }
                                  |
                                  v
                            Response to client

2. Team reviews extracted fields, checks confirmation boxes
   (Pure client-side, no API call)

3. Team uploads purchase contract PDF(s), one at a time
   _admin/index.html  --->  POST /api/extract-offer
                                  |
                                  v
                            Claude API (tool_use)
                            Returns: { buyerName, offerPrice, earnestMoney,
                                       financing, closingDate, contingencies, ... }
                                  |
                                  v
                            Response to client

4. Client auto-calculates net-to-seller per offer
   (Pure client-side using cost template from localStorage)

5. Team requests strategic analysis (optional)
   _admin/index.html  --->  POST /api/generate-analysis
                            Body: { listing, offers[], costTemplate }
                                  |
                                  v
                            Claude API (standard text response)
                            Returns: { analysis: "markdown text" }

6. Team clicks Preview -> client generates full HTML in-memory
   Opens in new tab via Blob URL

7. Team clicks Publish
   _admin/index.html  --->  POST /api/publish
                            Body: { html: base64EncodedHTML, path: "slug/index.html", sha?: "..." }
                                  |
                                  v
                            GitHub Contents API: PUT /repos/OWNER/REPO/contents/[slug]/index.html
                            (base64-encoded HTML, commit message)
                                  |
                                  v
                            Cloudflare Pages auto-deploys from repo (30-120 seconds)
```

#### Flow 2: Load and Update Existing Page

```
1. Team selects from "Load Existing" dropdown
   _admin/index.html  --->  GET /api/load
                                  |
                                  v
                            GitHub Contents API: GET /repos/OWNER/REPO/contents/
                            Returns: [ { name: "1960-e-dava-dr", type: "dir" }, ... ]

2. Client fetches existing page HTML directly from the live site
   _admin/index.html  --->  fetch("https://offers.liveazco.com/1960-e-dava-dr/index.html")
                            Parses embedded JSON from <script id="page-data">
                            Populates admin form fields

3. Team makes changes (edit fields, upload new contract PDF, etc.)

4. Team re-publishes
   _admin/index.html  --->  POST /api/publish
                            Body includes SHA of existing file (for GitHub update)
                            GitHub Contents API: PUT with SHA for update (not create)
```

## Patterns to Follow

### Pattern 1: Embedded JSON for Stateless Round-Tripping
**What:** Every generated page includes a `<script type="application/json" id="page-data">` block containing the complete structured data used to generate it. This eliminates the need for a database.
**When:** Always. Every published page must embed its source data.
**Why:** The admin tool can re-hydrate any existing page by fetching the HTML and parsing the embedded JSON. No separate data store to maintain, back up, or sync.
**Example:**
```html
<!-- In generated page, before closing </body> -->
<script type="application/json" id="page-data">
{
  "version": 1,
  "listing": { "address": "1960 E Dava Dr", "listPrice": 549000 },
  "offers": [ { "buyerName": "Smith", "offerPrice": 540000 } ],
  "costTemplate": { "titlePolicyPct": 0.5, "escrowFee": 1200 },
  "analysis": "The Smith offer presents the strongest...",
  "publishedAt": "2026-03-30T15:30:00Z"
}
</script>
```

### Pattern 2: Claude tool_use for Structured PDF Extraction
**What:** Use Claude's tool_use feature (not free-form text) to extract PDF data into a guaranteed schema. Define tools with JSON Schema input schemas matching the exact fields needed. Force the tool with `tool_choice: { type: "tool", name: "extract_listing_data" }`.
**When:** Every PDF extraction call (listing and offer).
**Why:** tool_use forces Claude to return data matching the schema. No regex parsing of free-form text. Fields that Claude cannot find are returned as null rather than hallucinated.
**Example:**
```typescript
const tools = [{
  name: "extract_listing_data",
  description: "Extract structured data from an ARMLS MLS listing PDF",
  input_schema: {
    type: "object",
    properties: {
      address: { type: "string", description: "Full street address" },
      listPrice: { type: "number", description: "Current list price in dollars" },
      beds: { type: "number" },
      baths: { type: "number" },
      sqft: { type: "number" },
      yearBuilt: { type: "number" },
      mlsNumber: { type: "string" },
      hoaMonthly: { type: ["number", "null"] }
    },
    required: ["address", "listPrice"]
  }
}];

const response = await fetch("https://api.anthropic.com/v1/messages", {
  method: "POST",
  headers: {
    "x-api-key": env.ANTHROPIC_API_KEY,
    "anthropic-version": "2023-06-01",
    "content-type": "application/json"
  },
  body: JSON.stringify({
    model: "claude-sonnet-4-20250514",
    max_tokens: 4096,
    tools,
    tool_choice: { type: "tool", name: "extract_listing_data" },
    messages: [{
      role: "user",
      content: [
        {
          type: "document",
          source: { type: "base64", media_type: "application/pdf", data: pdfBase64 }
        },
        {
          type: "text",
          text: "Extract all listing data from this ARMLS MLS listing sheet. If any value is illegible or ambiguous, return null rather than guessing."
        }
      ]
    }]
  })
});
```

### Pattern 3: Middleware Auth Gate
**What:** A single `_middleware.ts` in `functions/api/` that validates the `X-Admin-Key` header and enforces CORS before any handler runs.
**When:** Every API request.
**Example:**
```typescript
// functions/api/_middleware.ts
export async function onRequest(context) {
  // CORS preflight
  if (context.request.method === "OPTIONS") {
    return new Response(null, {
      headers: {
        "Access-Control-Allow-Origin": "https://offers.liveazco.com",
        "Access-Control-Allow-Methods": "GET, POST, OPTIONS",
        "Access-Control-Allow-Headers": "Content-Type, X-Admin-Key",
        "Access-Control-Max-Age": "86400"
      }
    });
  }

  // Auth check
  const key = context.request.headers.get("X-Admin-Key");
  if (key !== context.env.ADMIN_KEY) {
    return new Response(JSON.stringify({ error: "Unauthorized" }), {
      status: 401,
      headers: {
        "Content-Type": "application/json",
        "Access-Control-Allow-Origin": "https://offers.liveazco.com"
      }
    });
  }

  // Proceed to handler
  const response = await context.next();

  // Add CORS headers to response
  response.headers.set("Access-Control-Allow-Origin", "https://offers.liveazco.com");
  return response;
}
```

### Pattern 4: Client-Side HTML Generation and Net-to-Seller Calculation
**What:** All financial math and HTML template generation runs in the browser. Functions are thin API proxies only.
**When:** Every time an offer field changes or cost template is modified (for calc). On preview/publish (for HTML generation).
**Why:** Instant feedback for calculations. Keeps Functions thin with minimal CPU usage. Template changes deploy instantly since they are in the admin HTML file.
**Key formula:**
```javascript
function calculateNet(offer, listing, costTemplate) {
  const price = offer.offerPrice;
  const listAgentComm = price * (costTemplate.listAgentPct / 100);
  const buyerAgentComp = offer.buyerAgentCompensation || 0;
  const titlePolicy = price * (costTemplate.titlePolicyPct / 100);
  const escrowFee = costTemplate.escrowFee;
  const hoaTransfer = costTemplate.hoaTransferFee || 0;
  const warrantyCredit = offer.homeWarrantyCredit || 0;
  const sellerConcessions = offer.sellerConcessions || 0;

  const totalCosts = listAgentComm + buyerAgentComp + titlePolicy
    + escrowFee + hoaTransfer + warrantyCredit + sellerConcessions;

  return price - totalCosts;
}
```

## Anti-Patterns to Avoid

### Anti-Pattern 1: Storing Data in a Separate Backend
**What:** Using D1, KV, or any database to persist offer data alongside or instead of embedded JSON.
**Why bad:** Adds infrastructure to maintain, introduces sync bugs between DB and generated HTML, requires migration strategy. For a tool generating ~5-20 pages per year, a database is pure overhead.
**Instead:** Embedded JSON in generated pages. localStorage for admin-side drafts and cost templates.

### Anti-Pattern 2: Server-Side HTML Generation
**What:** Generating the offer comparison HTML inside a Pages Function.
**Why bad:** Functions have CPU time limits. Template generation is pure string work that belongs on the client. Debugging template issues requires redeploying functions.
**Instead:** Generate HTML entirely on the client using template literals. Functions only proxy API calls and commit files.

### Anti-Pattern 3: Splitting Admin Into Multiple HTML Pages
**What:** Separate pages for listing upload, offer entry, preview, etc.
**Why bad:** Breaks the single-file pattern the team already uses (Tenet converter). State management across pages requires URL params or additional storage.
**Instead:** Tab-based or step-based UI within a single HTML file. Show/hide sections as the workflow progresses.

### Anti-Pattern 4: Client-Side PDF-to-Text Extraction
**What:** Using pdf.js or similar to extract text client-side before sending to Claude.
**Why bad:** ARMLS PDFs and AAR contracts have complex layouts, tables, and varying formats. Client-side extraction loses layout context that Claude's native PDF vision preserves. Also adds a 300KB+ library.
**Instead:** Send raw base64 PDF directly to Claude via the Pages Function. Claude processes the PDF as both text and images.

### Anti-Pattern 5: Using SDK Libraries in Functions
**What:** Installing @anthropic-ai/sdk or Octokit in Pages Functions.
**Why bad:** Bundle size, edge runtime compatibility issues (documented streaming bugs), dependency management for functions that each make 1-2 fetch calls.
**Instead:** Direct fetch() with proper headers. ~20 lines per API call.

## Scalability Considerations

This is a low-volume internal tool. Scalability is not a primary concern.

| Concern | Current Scale (~20 pages/year) | If Scale Increases |
|---------|-------------------------------|-------------------|
| API costs | ~$0.50-2 per page (3-5 Claude calls) | Acceptable up to hundreds of pages |
| GitHub API rate | Well under 5,000 req/hour limit | Would hit limits only at enterprise scale |
| Pages Function limits | Paid tier: 10M req/month included | More than sufficient |
| File size | Each generated page ~50-100KB | No concern; static HTML scales on CDN |
| Concurrent users | Team of 3, no concurrency concern | Last-write-wins via GitHub SHA |

## Sources

- [Cloudflare Pages Functions: Get Started](https://developers.cloudflare.com/pages/functions/get-started/) - HIGH confidence
- [Cloudflare Pages Functions: Routing](https://developers.cloudflare.com/pages/functions/routing/) - HIGH confidence
- [Claude API: PDF Support](https://platform.claude.com/docs/en/build-with-claude/pdf-support) - HIGH confidence
- [GitHub REST API: Repository Contents](https://docs.github.com/en/rest/repos/contents) - HIGH confidence
- Existing reference: `1960-e-dava-dr/index.html` (Dava Dr design template) - inspected directly
- Existing pattern: `Tenet Equity/tenet-converter.html` (single-file admin tool) - inspected directly
