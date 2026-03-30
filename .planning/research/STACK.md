# Technology Stack

**Project:** LiveAZ Offers - Offer Comparison Admin Tool
**Researched:** 2026-03-30

## Recommended Stack

This is a zero-build, single-file HTML admin tool with Cloudflare Pages Functions as the backend. No npm, no bundler, no framework. CDN-loaded libraries where needed, raw `fetch()` everywhere else.

### Admin Tool (Client-Side)

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Vanilla HTML/CSS/JS | N/A | Admin interface | Matches Tenet converter pattern the team already uses. No build step, any developer can maintain. Single-file = single deployment artifact | HIGH |
| Google Fonts (Playfair Display + DM Sans) | CDN | Typography | Already used in the Dava Dr template. Playfair for headings, DM Sans for body | HIGH |
| FileReader API | Browser native | PDF upload handling | Read PDF files as ArrayBuffer, convert to base64 for Claude API. No library needed | HIGH |

### Backend (Cloudflare Pages Functions)

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Cloudflare Pages Functions | Workers runtime | API proxy layer | Same repo, same deploy. Functions in `/functions/` directory auto-deploy with the site. No separate Worker needed | HIGH |
| `wrangler.toml` | Wrangler v3 | Configuration | Define secrets (ANTHROPIC_API_KEY, GITHUB_TOKEN, ADMIN_KEY), compatibility_date, compatibility_flags | HIGH |

### External APIs

| API | Purpose | Why | Confidence |
|-----|---------|-----|------------|
| Anthropic Messages API (direct `fetch`) | PDF extraction via tool_use | Claude Sonnet processes PDFs natively (base64 document blocks). tool_use with JSON schema forces structured output. No SDK needed in Workers runtime | HIGH |
| GitHub Contents API (direct `fetch`) | Publish HTML files to repo | PUT `/repos/{owner}/{repo}/contents/{path}` creates/updates files with base64-encoded content. Simple REST, no SDK needed | HIGH |

### Claude API Configuration

| Setting | Value | Rationale |
|---------|-------|-----------|
| Model | `claude-sonnet-4-20250514` | Per PROJECT.md constraint. Best cost/quality for structured extraction from PDFs |
| API version header | `anthropic-version: 2023-06-01` | Current stable API version |
| Max request size | 32 MB | PDF limit per Anthropic docs. Client-side 10MB gate means we stay well under |
| Max pages per request | 100 (Sonnet has 200k context) | ARMLS listings are 1-3 pages, AAR contracts are 10-30 pages. Well within limits |
| Token cost per page | ~1,500-3,000 text + image tokens | Each PDF page is converted to image + text. A 30-page contract costs roughly 45k-90k tokens |

### PDF Processing Approach

Use `tool_use` (not structured outputs beta) for extraction. Here is why:

1. **tool_use is GA and works on all models.** Structured outputs beta (`anthropic-beta: structured-outputs-2025-11-13`) only supports Sonnet 4.5 and Opus 4.1 as of March 2026. The PROJECT.md specifies `claude-sonnet-4-20250514` which is Sonnet 4, not 4.5
2. **tool_use forces JSON output matching your schema.** Define a tool like `extract_listing_data` with an `input_schema` describing every field. Claude calls the tool with structured data. You read `tool_use` content blocks from the response
3. **No SDK dependency in Workers.** Direct `fetch()` to `https://api.anthropic.com/v1/messages` with the tool definition in the request body. The Workers runtime handles this perfectly

### GitHub Publishing Approach

| Concern | Approach |
|---------|----------|
| Create new page | PUT with `message`, `content` (base64 HTML), `branch` |
| Update existing page | PUT with same params plus `sha` of existing file (must GET first) |
| File size limit | 100 MB max via Contents API. Generated HTML pages will be 50-100 KB, not a concern |
| Auth | Fine-grained Personal Access Token with `contents: write` scope on the offers repo only |

### No SDK Policy

**Do NOT use `@anthropic-ai/sdk` in Pages Functions.** Reasons:

1. **The SDK is 0.80.0** and designed for Node.js. It works in Workers but adds unnecessary bundle size to a function that makes one `fetch()` call
2. **Known streaming issues in edge runtimes** (documented in [GitHub issue #292](https://github.com/anthropics/anthropic-sdk-typescript/issues/292)). We do not need streaming for extraction anyway (we want the complete tool_use response)
3. **Direct `fetch()` is ~20 lines of code.** Set headers, JSON.stringify the body, parse the response. The SDK adds no value here

**Do NOT use Octokit** for GitHub API calls. Same reasoning: one PUT endpoint, direct `fetch()` is simpler.

## Project Structure

```
liveaz-offers/
  _admin/
    index.html              # Single-file admin tool (all CSS/JS inline)
  functions/
    api/
      extract-listing.ts    # Claude API: extract MLS listing data from PDF
      extract-offer.ts      # Claude API: extract purchase contract data from PDF
      extract-addendum.ts   # Claude API: merge addendum/counter terms into existing offer
      generate-analysis.ts  # Claude API: draft strategic analysis comparing all offers
      publish.ts            # GitHub Contents API: commit HTML to repo
      load.ts               # GitHub Contents API: fetch existing page data
  1960-e-dava-dr/
    index.html              # Existing reference design (template source)
  wrangler.toml             # Pages config: secrets, compatibility
  .dev.vars                 # Local secrets (gitignored)
```

## Secrets & Environment Variables

Managed via Cloudflare dashboard (production) and `.dev.vars` (local development).

| Secret | Purpose |
|--------|---------|
| `ANTHROPIC_API_KEY` | Claude API authentication |
| `GITHUB_TOKEN` | Fine-grained PAT for GitHub Contents API |
| `ADMIN_KEY` | X-Admin-Key header validation value |

### CORS & Security

| Concern | Implementation |
|---------|---------------|
| CORS | Pages Function checks `Origin` header against `offers.liveazco.com`. Returns 403 for other origins |
| Admin auth | Every API function validates `X-Admin-Key` header against `env.ADMIN_KEY` secret. Not encrypted auth, but sufficient for internal team tool |
| GitHub token scope | Fine-grained PAT scoped to single repo with `contents: write` only |

## Cloudflare Workers Runtime Limits (Relevant)

| Limit | Free Plan | Paid Plan | Impact |
|-------|-----------|-----------|--------|
| CPU time/request | 10 ms | 30s default, 5 min max | Free plan is NOT viable. Claude API calls take 5-30 seconds. **Must use Workers Paid ($5/mo)** |
| Memory | 128 MB | 128 MB | Sufficient for base64 PDF handling |
| Subrequests/request | 50 | 10,000 | Fine. We make 1-2 subrequests per function |
| Request body size | 100 MB (free/pro) | 100 MB | Client-side 10 MB gate keeps us well under |
| Script size (compressed) | 3 MB | 10 MB | No concern with direct fetch approach |

**CRITICAL: The free Workers plan has 10ms CPU time limit. This is not enough for any Claude API call. The $5/mo Workers Paid plan is required.**

## Alternatives Considered

| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| Admin UI framework | Vanilla HTML/JS | React/Vue via CDN | Adds complexity for a form-based admin tool. The Tenet converter proves vanilla works for this team |
| Claude SDK | Direct fetch | @anthropic-ai/sdk | Bundle size, edge runtime issues, unnecessary abstraction for 1 endpoint |
| GitHub SDK | Direct fetch | Octokit | Same reasoning. One PUT call does not justify a library |
| Backend | Pages Functions | Standalone Worker | Extra infrastructure. Pages Functions deploy with the site |
| Backend | Pages Functions | Vercel Serverless | Team already uses Cloudflare for this domain. Consistent infra |
| PDF parsing | Claude vision | pdfplumber/pdf.js client-side | Claude handles scanned PDFs, complex layouts, and returns structured data. Client-side parsing cannot match this for real estate contracts |
| Database | None (embedded JSON) | Supabase/D1 | Generated pages carry their own data as embedded JSON. No persistence layer needed |
| Structured output | tool_use (GA) | Structured outputs beta | Model compatibility constraint. tool_use works on all Claude models |

## Local Development

```bash
# Install Wrangler globally (if not already)
npm install -g wrangler

# Create .dev.vars with secrets
echo "ANTHROPIC_API_KEY=sk-ant-..." > .dev.vars
echo "GITHUB_TOKEN=github_pat_..." >> .dev.vars
echo "ADMIN_KEY=your-admin-key" >> .dev.vars

# Run locally with Pages dev server
wrangler pages dev . --compatibility-date=2024-09-23 --compatibility-flags=nodejs_compat
```

No `npm install` needed for the project itself. The admin page is a single HTML file. Functions are TypeScript files that Wrangler compiles.

## Sources

- [Cloudflare Pages Functions docs](https://developers.cloudflare.com/pages/functions/)
- [Cloudflare Workers limits](https://developers.cloudflare.com/workers/platform/limits/)
- [Cloudflare Pages limits](https://developers.cloudflare.com/pages/platform/limits/)
- [Claude API PDF support](https://platform.claude.com/docs/en/build-with-claude/pdf-support)
- [Claude API structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)
- [Claude tool_use structured JSON extraction cookbook](https://github.com/anthropics/anthropic-cookbook/blob/main/tool_use/extracting_structured_json.ipynb)
- [GitHub Contents API](https://docs.github.com/en/rest/repos/contents)
- [@anthropic-ai/sdk on npm](https://www.npmjs.com/package/@anthropic-ai/sdk) (v0.80.0, not recommended for this project)
- [Anthropic SDK edge runtime streaming issue #292](https://github.com/anthropics/anthropic-sdk-typescript/issues/292)
- [Cloudflare Workers secrets](https://developers.cloudflare.com/workers/configuration/secrets/)
