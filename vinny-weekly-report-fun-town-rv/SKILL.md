---
name: vinny-weekly-report-fun-town-rv
description: Scrape vinnyreporting.com weekly data for Fun Town RV (all locations) and email a branded weekly performance report every Saturday at 9 AM. Includes Executive Summary, Performance Highlights, Category Matrix (OEM-level only), DAF user metrics, Top 10 Jobs by Fail Count with logos/report links, and Units Inspected by Make with Avg Fails/Unit.
---

## ABSOLUTE RULES — READ BEFORE ANYTHING ELSE

1. ⛔ **NEVER FABRICATE DATA.** ALL data must come from scraping vinnyreporting.com. If scraping fails, STOP and report the error. Never substitute fake data, placeholder numbers, or hallucinated values.

2. ⛔ **FULLY AUTONOMOUS.** Run start-to-finish without asking for user approval. Authenticate, scrape, build, send — one uninterrupted flow. Send exactly ONCE per run.

3. ⛔ **NO SUB-AGENTS.** Do NOT use the Agent tool or spawn sub-agents for any part of this task. Sub-agents burn rate limits and lose context. Do everything yourself in a single sequential session.

4. ⛔ **PROGRAMMATIC DATA TRANSFER ONLY.** When moving data from the vinnyreporting tab to the Gmail tab, you MUST use `JSON.stringify()` on the source tab and `JSON.parse()` on the Gmail tab. NEVER manually type, transcribe, or re-enter data values. Every number in the email must trace back to a JSON.stringify → JSON.parse pipeline with zero human-in-the-loop transcription. If the JSON is too large, split into chunks and concatenate.

5. ⛔ **DO NOT MIX CATEGORY NAME SYSTEMS.** There are TWO naming conventions on vinnyreporting.com:
   - **Category Matrix names** (Steps 6-7): format `Group > SubCategory` — use ONLY in Performance Highlights and Category Matrix tables
   - **Persistent Issues names** (Step 11): longer descriptive names — use ONLY in Executive Summary
   Never rename, merge, abbreviate, or paraphrase any category name. Copy exactly as scraped.

6. ⛔ **HEADER-MATCHING TABLE SELECTORS ONLY.** Never use index-based selectors like `querySelectorAll('table')[7]`. Always find tables by matching header text content (e.g., find a table whose `<thead th>` elements include 'VIN' and 'Job Name').

7. ⛔ **HTML SIZE < 90KB.** Gmail clips emails at ~102KB. Check `window._fullHtml.length` before injecting — must be under 90000 characters. If exceeded, reduce content first.

8. ⛔ **USE THE BUILDER FUNCTION.** The email template provides a complete JavaScript builder function that produces the entire email HTML. Do NOT write your own HTML building code. Execute the builder function EXACTLY as provided. This prevents column-alignment bugs that have occurred in every previous attempt where HTML was built manually.

9. ⛔ **VERBATIM UPDATES.** When copying Trip's weekly updates from Gmail, copy each bullet point WORD-FOR-WORD. Do NOT rephrase, condense, shorten, or paraphrase. If Trip wrote 8 bullets, the report must have 8 bullets with the same wording.

10. ⛔ **DYNAMIC COLUMN DETECTION.** The Job Reports table columns may change between dashboard versions. NEVER hardcode column indices (like `cells[8]` for Fails). Always find columns by matching header text first, then use the discovered indices.

---

## RECIPIENTS

- **To:** jen.berry@funtownrv.com,james.berry@funtownrv.com,patrick.baker@funtownrv.com,jim.fenner@funtownrv.com,chris@bulliontx.com,danielle.beber@funtownrv.com,kendrick.cox@funtownrv.com,jane.mcvoy@funtownrv.com
- **CC:** trip@getvinny.com,rob@getvinny.com,bobbie@getvinny.com,john@getvinny.com

## CONFIG
```
ORG_MATCH    = "Fun Town RV"
LOCATION     = all (no location filter)
DATE_RANGE   = Last 7 Days (for Steps 5-9), Year to Date (for Step 11 only)
EXPECTED_OEM = 7 parent rows: Coachmen, East to West, Forest River, Gulf Stream, Heartland by Jayco, Prime Time, Winnebago
```

---

## REFERENCE FILES — READ JUST-IN-TIME

This skill has two reference files in the `references/` directory. Load each one at the indicated step:

| File | When to read | Contains |
|------|-------------|----------|
| `references/scraping_guide.md` | **Before Step 1** | Auth flow, JS selectors, column mappings, all scraping code (Steps 1-11) |
| `references/email_template.md` | **Before Step 12** | Executive Summary structure, deterministic HTML builder function, verification steps (Steps 12-14) |

⛔ You MUST read these files before executing the corresponding steps. They contain critical JavaScript code, CSS selectors, and column mappings that you cannot improvise.

---

## STEP EXECUTION ORDER

Execute in this exact order. Do NOT skip or reorder steps.

### Phase 1: Authentication & Setup
1. **Step 1** — Authenticate to vinnyreporting.com via magic link (see scraping_guide.md)
2. **Step 2** — Filter to Fun Town RV org (see scraping_guide.md)
3. **Step 3** — Set date range to Last 7 Days (see scraping_guide.md)
4. **Step 4** — Navigate to Dashboards > Category Matrix, ensure DAF+PDI active (see scraping_guide.md)

### Phase 2: Scrape Dashboard Data (Last 7 Days)
5. **Step 5** — Scrape Job Summary Stats from Dashboards > Job Reports tab → handle PAGINATION (load ALL rows, not just first 50) → store as `window._jobStats` (see scraping_guide.md).
6. **Step 5b** — Verify org filter shows "Fun Town RV" (see scraping_guide.md). If wrong, STOP and fix.
7. **Step 6** — Scrape Category Matrix in Fail Count mode → store as `window._failTotals` ordered array (see scraping_guide.md)
8. **Step 7** — Scrape Category Matrix in % Units mode → store as `window._oemData` (with `values` as ordered array) and `window._colNames` (see scraping_guide.md). Verify exactly 7 OEM rows.
9. **Step 7b** — ⛔ COLUMN NAME VERIFICATION (see scraping_guide.md). Verify idx24=Water, idx26=Propane, idx28=RVIA. If wrong, STOP and debug.
10. **Step 8** — Deselect PDI → Scrape DAF User Metrics → Re-enable PDI → store as `window._userMetrics` (see scraping_guide.md)
11. **Step 9** — Scrape Job Reports table → ⛔ DISCOVER COLUMN INDICES BY HEADER TEXT (never hardcode) → store `window._top10` (9b) and `window._makeSummary` (9c) (see scraping_guide.md). Sanity-check: if all top 10 have identical fail counts, the Fails column index is likely wrong.
12. **Step 10** — Get Trip's latest updates from Gmail (see scraping_guide.md)

### Phase 3: Scrape Persistent Issues (Year to Date)
13. **Step 11** — Switch to Year to Date → Scrape Persistent Issues (All Time) → store as `window._persistentIssues` (see scraping_guide.md)
    ⚠️ Do this LAST among scraping steps — it changes the date range filter.

### Phase 4: Pre-Build Checkpoint
14. **Step 11e** — ⛔ DATA COMPLETENESS + ACCURACY CHECKPOINT (see below). MANDATORY — do NOT skip.

### Phase 5: Build Email
15. **Step 12** — Generate Executive Summary from Persistent Issues + Performance data (see email_template.md)
16. **Step 13** — Transfer data to Gmail tab, execute the builder function, run post-build verification (see email_template.md)

### Phase 6: Send
17. **Step 14** — Compose and send via Gmail (see email_template.md). Only after ALL checkpoints pass.

---

## STEP 11e: DATA COMPLETENESS + ACCURACY CHECKPOINT

⛔ **STOP. Before generating the Executive Summary or building ANY HTML, verify ALL data is collected.**

### Part A — Completeness + Sanity Check
Run on the vinnyreporting tab:
```javascript
const checks = {
  jobStats: typeof window._jobStats === 'object' && window._jobStats.uniqueUnits > 0,
  jobCountSanity: window._jobStats && window._jobStats.totalJobs > 0, // verify we got some jobs
  failTotals: Array.isArray(window._failTotals) && window._failTotals.length === 29,
  colNames: Array.isArray(window._colNames) && window._colNames.length === 29,
  oemData: Array.isArray(window._oemData) && window._oemData.length === 7 && Array.isArray(window._oemData[0].values) && window._oemData[0].values.length === 29,
  userMetrics: Array.isArray(window._userMetrics) && window._userMetrics.length >= 1,
  top10: Array.isArray(window._top10) && window._top10.length >= 1,
  makeSummary: Array.isArray(window._makeSummary) && window._makeSummary.length >= 1,
  persistentIssues: Array.isArray(window._persistentIssues) && window._persistentIssues.length >= 1
};
const missing = Object.entries(checks).filter(([k,v]) => !v).map(([k]) => k);
missing.length === 0 ? 'ALL DATA PRESENT' : 'MISSING: ' + missing.join(', ') + ' — GO BACK AND SCRAPE THESE';
```
If ANY item is missing, go back and collect it. Do NOT proceed without complete data.

### Part B — Column Alignment Re-Verification
```javascript
const cn = window._colNames;
const aligned = cn[21]?.includes('Furniture') && cn[24]?.includes('Water') && cn[26]?.includes('Propane') && cn[28]?.includes('RVIA');
aligned ? 'COLUMNS ALIGNED' : 'COLUMN MISMATCH — idx21:'+cn[21]+' idx24:'+cn[24]+' idx26:'+cn[26]+' idx28:'+cn[28];
```
⛔ If columns are not aligned, STOP. The builder function will produce wrong data.

### Part C — Accuracy Verification
Re-read values directly from the live dashboard DOM and compare against stored data:
```javascript
const parentRows = document.querySelectorAll('.matrix-row-parent');
const spotChecks = [];
parentRows.forEach(row => {
  const cells = row.querySelectorAll('td');
  const oem = cells[0].textContent.trim().replace(/^▶/, '').trim();
  const units = cells[1]?.textContent?.trim();
  spotChecks.push({
    oem, units,
    col2: cells[2]?.textContent?.trim(),
    col15: cells[15]?.textContent?.trim(),
    col25: cells[25]?.textContent?.trim()
  });
});
JSON.stringify(spotChecks.slice(0, 3));
```
Compare those 3 OEM rows against corresponding values in `window._oemData`. If ANY value differs: STOP, re-scrape from scratch, re-run checkpoint.

Only proceed to Step 12 after Parts A, B, and C pass.

---

## STEP 14: COMPOSE AND SEND VIA GMAIL

**Subject:** `Vinny Fun Town RV Weekly Report - MM/DD/YYYY` (use today's date)

### Compose flow:
1. Click Compose in Gmail.
2. Use `find` tool to locate To field, then `form_input` to set the To recipients (see RECIPIENTS above).
3. Set the Subject field.
4. Use `find` tool to locate CC field, then `form_input` to set CC recipients.
5. The email_template.md file has the complete injection and send code.

---

## IMPORTANT NOTES
- Gmail tabs may not reach document_idle — use JavaScript tool for all Gmail DOM interactions
- If data is too large for a single JS tool output, scrape in batches via window variables
- If authentication has expired, re-authenticate via magic link flow
- The Ranger Miles signature is auto-populated in compose — insertHTML preserves it
- The Executive Summary should feel like a brief from a sharp analyst — data-driven, not generic
- Logo images in Top 10 Jobs are hosted on prod-images.getvinny.com — some clients may block external images, acceptable
- Report "View" links point to prodapp.getvinny.com — use exact URLs from scraped data
- Units by Make uses Avg Fails / Unit (totalFails / uniqueUnits), NOT per job
- Category Matrix shows parent OEM names only — filter out sub-model rows
- Do NOT include "Interior Structure (Other)" / "Interior > Other" in Top 3 Problem Areas — always footnote instead
- Do NOT imply limited data or early-stage trends — Fun Town RV has months of data
