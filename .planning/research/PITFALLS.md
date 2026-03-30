# Domain Pitfalls

**Domain:** PDF parsing admin tool with Claude API extraction, Cloudflare Pages Functions, GitHub Contents API publishing
**Researched:** 2026-03-30

## Critical Pitfalls

Mistakes that cause rewrites, data corruption, or six-figure seller decisions going wrong.

### Pitfall 1: Claude Misreads Financial Numbers in Purchase Contracts

**What goes wrong:** Claude extracts a purchase price of $485,000 when the contract says $458,000. Digits get transposed, decimal points shift, or handwritten addendum amounts get hallucinated.

**Why it happens:** AAR purchase contracts are 10-30 page scanned PDFs with small fonts, checkboxes, handwritten fields, and fax-quality reproduction. Claude's PDF processing converts each page to an image plus extracted text, but scanned documents produce noisy images.

**Consequences:** Seller evaluates offers based on wrong net-to-seller numbers. Could accept $20K less than the best offer. Legal liability for the team.

**Prevention:**
- The checkbox confirmation gate is the primary defense. Every extracted dollar amount must be human-verified before publishing
- Use `tool_use` with strict schema definitions that force separate fields for each financial term
- In the extraction prompt: "If a value is illegible or ambiguous, return null rather than guessing. Never estimate financial figures"
- Display extracted values alongside the source PDF so the reviewer can visually compare
- Add a "confidence" field to the extraction schema so Claude can flag uncertain values

**Detection:** Extracted values that seem unusual (purchase price below list price by >20%, earnest money over 5% of purchase price, close of escrow less than 15 days out) should trigger visual warnings.

**Confidence:** HIGH (verified via Claude official docs on PDF limitations and vision constraints)

---

### Pitfall 2: Cloudflare Free Plan CPU Time Limit (10ms)

**What goes wrong:** Pages Functions work perfectly in local development (`wrangler pages dev` has no CPU limit), then fail in production with "Worker exceeded CPU time limit" errors.

**Why it happens:** The free Workers plan allows only 10ms of CPU time per request. While waiting on `fetch()` to Claude API does not count as CPU time, constructing the base64 payload, parsing Claude's JSON response, and preparing the GitHub API request all consume CPU.

**Consequences:** Every API endpoint returns 500 errors in production. The tool is completely non-functional.

**Prevention:** Use the Cloudflare Workers Paid plan ($5/month). The paid plan gives 30 seconds default CPU time (configurable up to 5 minutes). This is not optional for this project.

**Detection:** Test with realistic PDFs (10-30 pages) in production environment, not just small test files. Monitor with `wrangler tail --format json`.

**Confidence:** HIGH (verified via [Cloudflare Workers limits documentation](https://developers.cloudflare.com/workers/platform/limits/))

---

### Pitfall 3: GitHub Contents API SHA Conflicts on Update

**What goes wrong:** When updating a previously published offer page, the GitHub Contents API returns 409/422 with "sha wasn't supplied" or "The blob SHA is not matched."

**Why it happens:** PUT to GitHub Contents API requires the `sha` of the current file blob when updating. If the file was modified by another process, the SHA is stale. Creating a new file does not require SHA, but update does.

**Consequences:** "Load Existing -> Edit -> Publish" workflow breaks. Team sees a cryptic error with no recovery path.

**Prevention:**
- Always GET the file to retrieve current SHA immediately before PUT. Never cache SHAs across sessions
- If PUT returns 409/422, re-fetch SHA and retry once
- For new files, omit the `sha` field entirely
- Handle 404 on GET (file deleted externally): fall back to create-new-file flow

**Detection:** Test the full round-trip: publish -> load -> edit -> re-publish. Test after manually editing a file in GitHub's UI.

**Confidence:** HIGH (verified via [GitHub Contents API documentation](https://docs.github.com/en/rest/repos/contents))

---

### Pitfall 4: Embedded JSON Round-Trip Data Corruption

**What goes wrong:** Special characters in offer data (dollar signs, quotes, backslashes, apostrophes in names like "O'Brien") break the embedded JSON or the HTML, causing the page to fail to load or displaying corrupted data.

**Why it happens:** If data contains `</script>`, the browser closes the script tag prematurely. If it contains unescaped quotes, JSON parse fails. This is explicitly called out in the project's CLAUDE.md: "always sanitize for special characters."

**Consequences:** A published page works fine. Weeks later, the team loads it to update an offer and the admin tool crashes with a JSON parse error. They cannot edit the page.

**Prevention:**
- Use `JSON.stringify()` for all data embedding, never manual string interpolation
- Embed JSON in `<script type="application/json" id="page-data">` (not `text/javascript`), then parse with `JSON.parse(document.getElementById('page-data').textContent)`
- Test with adversarial data: names with apostrophes, notes with `</script>`, dollar signs, unicode
- Validate round-tripping: generate page -> extract JSON -> compare to original

**Detection:** Create a test offer with buyer name "O'Brien & Sons </script>" and notes containing backticks and dollar signs.

**Confidence:** HIGH (known class of bug documented in project code conventions)

## Moderate Pitfalls

### Pitfall 5: Base64 Encoding Overhead for Large PDFs

**What goes wrong:** A 10MB PDF becomes ~13.3MB when base64-encoded. Multi-hop path (browser -> Function -> Claude) adds latency and memory pressure (128MB Worker limit).

**Prevention:**
- Process PDFs one at a time (one Function invocation per PDF)
- Client-side validation rejects PDFs over 10MB (per PROJECT.md)
- For most ARMLS listings (1-3 pages, 500KB-2MB) and AAR contracts (10-30 pages, 1-5MB), this is not a practical concern

**Confidence:** HIGH (verified: base64 adds 33%, Claude API limit is 32MB, Workers memory is 128MB)

### Pitfall 6: Claude Extracts Different Fields From Different Contract Formats

**What goes wrong:** Works with standard AAR purchase contracts, then fails with BINSR responses, counter-offers, or brokerage-specific addenda.

**Prevention:**
- Let the user select document type before upload so the correct extraction schema is used
- Include document type context in the extraction prompt
- Design tool_use schema with an "additional_terms" catch-all field for non-standard clauses

**Confidence:** MEDIUM (based on domain knowledge of Arizona real estate document types)

### Pitfall 7: Net-to-Seller Calculation Diverges From Title Company Estimates

**What goes wrong:** Tool calculates one number, title company produces another. Sellers lose trust.

**Prevention:**
- Label as "ESTIMATED" prominently in both admin UI and published page
- Make every cost template line item individually editable and visible
- Allow per-offer cost overrides
- Compare against a real preliminary title report to calibrate defaults

**Confidence:** MEDIUM (domain-specific; mitigation is straightforward)

### Pitfall 8: CORS Misconfiguration Blocks Admin in Production

**What goes wrong:** Admin page at `offers.liveazco.com/_admin/` cannot reach Pages Functions. Works locally (same origin) but fails in production.

**Prevention:**
- Handle `OPTIONS` preflight requests explicitly in middleware
- Include CORS headers on error responses too (browser swallows the error otherwise)
- Local dev note: `wrangler pages dev` runs on localhost, so either bypass CORS locally or set a different allowed origin for dev

**Confidence:** HIGH (standard web development pitfall)

### Pitfall 9: Claude API Latency Creates Poor UX

**What goes wrong:** Each PDF extraction takes 15-30 seconds. Admin uploads MLS listing plus 4 contracts and UI appears frozen for 2+ minutes.

**Prevention:**
- Show per-PDF progress indicators
- Process PDFs sequentially with UI updates between each
- Add a "Cancel" button that aborts the in-flight fetch
- Consider extracting only key pages from long contracts

**Confidence:** HIGH (verified: Claude docs confirm 1,500-3,000 tokens/page)

## Minor Pitfalls

### Pitfall 10: Cloudflare Pages Deploy Timing

**What goes wrong:** After publishing via GitHub API, team clicks the URL immediately and sees the old page.

**Prevention:** Show "Published! Page will be live in 1-2 minutes" with the URL. Do not imply instant availability.

### Pitfall 11: localStorage Cost Template Lost on Browser/Device Change

**What goes wrong:** Suzie configures cost template on laptop. Jacqui opens admin on phone. Jacqui's template is empty.

**Prevention:**
- Ship sensible default values so the tool works without configuration
- Add Export/Import Settings button for JSON file transfer
- Display warning when using defaults: "Using default cost estimates. Review before publishing."

### Pitfall 12: Generated HTML File Naming

**What goes wrong:** Special characters or spaces in file paths break GitHub API calls or URL routing.

**Prevention:** Slugify the address: lowercase, replace spaces with hyphens, strip special characters. Pattern: `{slug}/index.html`.

### Pitfall 13: API Key Exposure in Client-Side Code

**What goes wrong:** Claude API key ends up in client-side JavaScript or in Function error response bodies.

**Prevention:**
- API keys live exclusively in Cloudflare environment variables (set via dashboard)
- Functions access via `context.env.ANTHROPIC_API_KEY`
- Never include API keys in error response bodies
- Admin page authenticates via X-Admin-Key (separate from Claude API key)

## Phase-Specific Warnings

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|---------------|------------|
| Infrastructure (Phase 1) | Free plan CPU limit (Pitfall 2) | Verify paid plan before writing functions |
| Infrastructure (Phase 1) | CORS blocks production (Pitfall 8) | Handle OPTIONS preflight, CORS on errors |
| Infrastructure (Phase 1) | API key exposure (Pitfall 13) | Keys in env vars only, sanitize error responses |
| PDF Extraction (Phase 2) | Claude misreads numbers (Pitfall 1) | Confirmation checkboxes, null-on-ambiguity prompt |
| PDF Extraction (Phase 2) | Long extraction times (Pitfall 9) | Per-PDF progress indicators |
| Calculations (Phase 3) | Net diverges from title co (Pitfall 7) | Label "ESTIMATED", editable line items |
| Publishing (Phase 4) | GitHub SHA conflicts (Pitfall 3) | GET-before-PUT, retry on 409/422 |
| Publishing (Phase 4) | JSON corruption (Pitfall 4) | JSON.stringify, script type="application/json" |
| Publishing (Phase 4) | Deploy timing (Pitfall 10) | Clear messaging about 1-2 minute delay |

## Sources

- [Claude PDF Support Documentation](https://platform.claude.com/docs/en/build-with-claude/pdf-support) (HIGH confidence)
- [Cloudflare Workers Limits](https://developers.cloudflare.com/workers/platform/limits/) (HIGH confidence)
- [GitHub Contents API Documentation](https://docs.github.com/en/rest/repos/contents) (HIGH confidence)
- [@anthropic-ai/sdk edge runtime issue #292](https://github.com/anthropics/anthropic-sdk-typescript/issues/292) (MEDIUM confidence)
