---
phase: 1
slug: form-data-entry
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-30
---

# Phase 1 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Manual browser testing (no automated test framework) |
| **Config file** | none |
| **Quick run command** | Open `index.html` in browser |
| **Full suite command** | Manual walkthrough of all form interactions |
| **Estimated runtime** | ~2 minutes |

---

## Sampling Rate

- **After every task commit:** Open `index.html` in browser, verify new functionality works
- **After every plan wave:** Full manual walkthrough: fill all fields, add 3 offers, remove middle one, verify cost persistence on refresh
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 10 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 01-01-01 | 01 | 1 | PROP-01 | manual | Open file, fill address fields | N/A | ⬜ pending |
| 01-01-02 | 01 | 1 | PROP-02 | manual | Fill beds/baths/sqft/etc fields | N/A | ⬜ pending |
| 01-01-03 | 01 | 1 | PROP-03 | manual | Type number in list price field | N/A | ⬜ pending |
| 01-01-04 | 01 | 1 | PROP-04 | manual | Type in features textarea | N/A | ⬜ pending |
| 01-01-05 | 01 | 1 | OFFR-01 | manual | Fill buyer name, agent, brokerage | N/A | ⬜ pending |
| 01-01-06 | 01 | 1 | OFFR-02 | manual | Fill price, down payment, loan type, earnest | N/A | ⬜ pending |
| 01-01-07 | 01 | 1 | OFFR-03 | manual | Fill COE date, inspection days, expiration | N/A | ⬜ pending |
| 01-01-08 | 01 | 1 | OFFR-04 | manual | Fill concessions, buyer agent comp | N/A | ⬜ pending |
| 01-01-09 | 01 | 1 | OFFR-05 | manual | Fill pre-qual amount, lender | N/A | ⬜ pending |
| 01-01-10 | 01 | 1 | OFFR-06 | manual | Type in special terms textarea | N/A | ⬜ pending |
| 01-01-11 | 01 | 1 | OFFR-07 | manual | Check/uncheck checkbox | N/A | ⬜ pending |
| 01-01-12 | 01 | 1 | OFFR-08 | manual | Click "Add Offer" multiple times, verify data isolation | N/A | ⬜ pending |
| 01-01-13 | 01 | 1 | OFFR-09 | manual | Click remove, confirm dialog, verify other cards intact | N/A | ⬜ pending |
| 01-01-14 | 01 | 1 | COST-01 | manual | Enter percentage value | N/A | ⬜ pending |
| 01-01-15 | 01 | 1 | COST-02 | manual | Enter dollar amount | N/A | ⬜ pending |
| 01-01-16 | 01 | 1 | COST-03 | manual | Enter dollar amount | N/A | ⬜ pending |
| 01-01-17 | 01 | 1 | COST-04 | manual | Enter dollar amount | N/A | ⬜ pending |
| 01-01-18 | 01 | 1 | COST-05 | manual | Add items, fill label+amount, remove one | N/A | ⬜ pending |
| 01-01-19 | 01 | 1 | COST-06 | manual | Fill costs, refresh page, verify values restored | N/A | ⬜ pending |
| 01-01-20 | 01 | 1 | ANLS-01 | manual | Type in analysis textarea | N/A | ⬜ pending |
| 01-01-21 | 01 | 1 | ANLS-02 | manual-only | Verified in Phase 3 when generate is built | N/A | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

Existing infrastructure covers all phase requirements. This is a single HTML file with no test framework. Validation is manual browser testing, appropriate for a zero-dependency vanilla HTML project used by a team of three.

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| All form interactions | PROP-01 through ANLS-02 | Single vanilla HTML file with no build step or test runner | Open index.html, manually exercise each form element per the verification map above |
| Cost template persistence | COST-06 | Requires browser localStorage | Fill cost fields, refresh page, verify values restored |
| Offer card isolation | OFFR-08, OFFR-09 | Requires visual confirmation of DOM state | Add 3 offers with data, remove middle one, verify remaining cards unchanged |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 10s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
