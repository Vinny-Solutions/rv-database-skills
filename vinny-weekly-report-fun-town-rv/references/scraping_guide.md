# Scraping Guide — Fun Town RV Weekly Report

This file contains all JavaScript selectors, column mappings, and scraping code for vinnyreporting.com. Read this BEFORE starting Step 5.

---

## STEP 1: AUTHENTICATE TO VINNYREPORTING.COM
1. Open a browser tab and navigate to https://www.vinnyreporting.com/
2. If not logged in, click the login/sign-in link.
3. Enter email: ranger@getvinny.com
4. Click "Send Magic Link" (or equivalent button).
5. Open Gmail for ranger@getvinny.com (https://mail.google.com) in another tab.
6. Find the most recent magic link email from Vinny and extract the authentication URL.
7. Navigate to that URL to complete authentication.
8. If a "Sign In to Vinny Reports" confirmation button appears, click it.
9. Navigate back to https://www.vinnyreporting.com/ — you should now be logged in.

## STEP 2: FILTER TO CUSTOMER ORG
Select **Fun Town RV** from the organization dropdown on the dashboard. Do NOT filter by location — this report covers ALL locations.

## STEP 3: SET DATE RANGE TO LAST 7 DAYS
Click the "Last 7 Days" quick-select button, or set the date range inputs:
```javascript
const end = new Date();
const start = new Date();
start.setDate(start.getDate() - 7);
const fmt = d => d.toISOString().slice(0, 10);
const inputs = document.querySelectorAll('input[type="date"]');
const nativeSet = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value').set;
nativeSet.call(inputs[0], fmt(start));
inputs[0].dispatchEvent(new Event('input', { bubbles: true }));
inputs[0].dispatchEvent(new Event('change', { bubbles: true }));
nativeSet.call(inputs[1], fmt(end));
inputs[1].dispatchEvent(new Event('input', { bubbles: true }));
inputs[1].dispatchEvent(new Event('change', { bubbles: true }));
```
Wait 3-5 seconds for data to reload.

## STEP 4: NAVIGATE TO DASHBOARDS > CATEGORY MATRIX
Click the "Dashboards" tab, then click the "Category Matrix" button. Make sure both DAF and PDI job type buttons are active/checked.

## STEP 5: SCRAPE JOB SUMMARY STATS

⛔ **The Reports tab table is PAGINATED** — it only shows 50 rows at a time. You MUST load ALL pages before counting jobs. Fun Town RV typically has 200-400 jobs per week.

Navigate to the Dashboards tab, then click the "Job Reports" sub-tab. This is where the full job list lives.

### 5a. Wait for table to fully load
The table may show a loading spinner or progressively load rows. Wait until the spinner stops and the row count stabilizes:
```javascript
// Check if table is still loading
const spinner = document.querySelector('.loading-spinner, .spinner, [class*="loading"]');
const isLoading = spinner && spinner.offsetParent !== null;
isLoading ? 'STILL LOADING — wait 3 seconds and check again' : 'LOADED';
```

### 5b. Load ALL rows — handle pagination
The Job Reports table paginates. Look for a "Show All" button, page size selector, or "Next" button. Try these approaches IN ORDER:

**Approach 1: Look for a page size selector and set to max**
```javascript
const pageSizeSelect = document.querySelector('select[class*="page"], select[class*="size"], select[class*="entries"]');
if (pageSizeSelect) {
  const maxOption = Array.from(pageSizeSelect.options).reduce((a,b) => parseInt(a.value) > parseInt(b.value) ? a : b);
  const nativeSet = Object.getOwnPropertyDescriptor(window.HTMLSelectElement.prototype, 'value').set;
  nativeSet.call(pageSizeSelect, maxOption.value);
  pageSizeSelect.dispatchEvent(new Event('change', { bubbles: true }));
  'Set page size to ' + maxOption.value;
} else { 'No page size selector found — try approach 2'; }
```
Wait 3 seconds after changing page size.

**Approach 2: Click "Next" until all pages loaded**
If no page size selector exists, look for next/pagination buttons and collect rows from all pages:
```javascript
// Collect all job data across pages
const allJobData = [];
const collectPage = () => {
  const allTables = document.querySelectorAll('table');
  const table = Array.from(allTables).find(t => {
    const ths = Array.from(t.querySelectorAll('thead th')).map(th => th.textContent.trim());
    return ths.includes('VIN') && (ths.includes('Job Name') || ths.includes('Job'));
  });
  if (!table) return 0;
  const headers = Array.from(table.querySelectorAll('thead th')).map(th => th.textContent.trim());
  const rows = table.querySelectorAll('tbody tr');
  return rows.length;
};
'Current page has ' + collectPage() + ' rows';
```
If the table has pagination controls (a "Next" button or page size selector), there may be more rows. Click "Next" and collect again. Repeat until no more pages.

### 5c. Count jobs from the FULL dataset
After all rows are visible/collected, count:
```javascript
const allTables = document.querySelectorAll('table');
const table = Array.from(allTables).find(t => {
  const ths = Array.from(t.querySelectorAll('thead th')).map(th => th.textContent.trim());
  return ths.includes('VIN') && (ths.includes('Job Name') || ths.includes('Job'));
});
if (!table) throw new Error('Could not find Job Reports table');
const headers = Array.from(table.querySelectorAll('thead th')).map(th => th.textContent.trim());
const jobNameIdx = headers.findIndex(h => h === 'Job Name' || h === 'Job');
const vinIdx = headers.indexOf('VIN');
const rows = table.querySelectorAll('tbody tr');
const uniqueVins = new Set();
let dafCount = 0, pdiCount = 0;
rows.forEach(row => {
  const cells = row.querySelectorAll('td');
  const vin = cells[vinIdx]?.textContent?.trim();
  const jobName = cells[jobNameIdx]?.textContent?.trim();
  if (vin) uniqueVins.add(vin);
  if (jobName && jobName.includes('Dealer Acceptance')) dafCount++;
  else if (jobName && jobName.includes('Pre-Delivery')) pdiCount++;
});
const totalRows = rows.length;
JSON.stringify({uniqueUnits: uniqueVins.size, dafJobs: dafCount, pdiJobs: pdiCount, totalJobs: dafCount + pdiCount, tableRows: totalRows});
```
Store result as `window._jobStats`.

⛔ **SANITY CHECK:** If totalJobs is 0, something is wrong — likely the org filter is incorrect or the table didn't load. STOP and debug. Also verify pagination was handled — if a "Next" button exists and wasn't clicked, rows may be missing.

## STEP 5b: VERIFY ORGANIZATION FILTER
⛔ **Before scraping ANY dashboard data**, verify the org filter:
```javascript
const orgEl = document.querySelector('.multiselect__single') || document.querySelector('[class*="dropdown"] [class*="selected"]');
const orgText = orgEl ? orgEl.textContent.trim() : 'UNKNOWN';
orgText; // MUST show "Fun Town RV" — if not, STOP and fix the org filter
```
With Fun Town RV selected, the Category Matrix should show exactly **7 parent OEM rows**: Coachmen, East to West, Forest River, Gulf Stream, Heartland by Jayco, Prime Time, Winnebago.

## STEP 6: SCRAPE CATEGORY MATRIX — FAIL COUNT MODE

### Button selection
There are TWO groups of `.matrix-metric` buttons. The matrix toggle group is the LAST set. Always match by button TEXT and pick the LAST matching occurrence:
```javascript
const allBtns = Array.from(document.querySelectorAll('button.matrix-metric'));
const failCountBtns = allBtns.filter(b => b.textContent.trim() === 'Fail Count');
const matrixFailCountBtn = failCountBtns[failCountBtns.length - 1];
matrixFailCountBtn.click();
```
Wait 2 seconds, then verify:
```javascript
const btns = Array.from(document.querySelectorAll('button.matrix-metric'));
const fcBtns = btns.filter(b => b.textContent.trim() === 'Fail Count');
const isActive = fcBtns[fcBtns.length - 1]?.classList?.contains('active');
isActive; // Must be true — if false, click again and wait 2 more seconds
```

### Column Header Mapping (TWO-ROW thead with colspan/rowspan)
```javascript
const matrixTable = document.getElementById('matrix-table-wrapper')?.querySelector('table');
const theadRows = matrixTable.querySelectorAll('thead tr');
const row0 = theadRows[0]; const row1 = theadRows[1];
const columnHeaders = []; let colIdx = 0;
const subHeaders = Array.from(row1.querySelectorAll('th')); let subIdx = 0;
row0.querySelectorAll('th').forEach(th => {
  const text = th.textContent.trim();
  const colspan = parseInt(th.getAttribute('colspan') || '1');
  const rowspan = parseInt(th.getAttribute('rowspan') || '1');
  if (rowspan === 2) { columnHeaders.push({ group: null, name: text, colIdx }); colIdx++; }
  else { for (let i = 0; i < colspan; i++) { const sub = subHeaders[subIdx]?.textContent?.trim() || '';
    columnHeaders.push({ group: text, name: sub, colIdx }); colIdx++; subIdx++; } }
});
```

**Expected 31 columns:** 0: OEM/Model, 1: Units, 2: Shore Cord, 3: Spare Tire, 4: Keys, 5: Locks, 6: Tongue/Pin, 7-14: Exterior (Decals, Trim/Skirts, Compartments, Hitch & Bumper, Windows, Slide Outs, Doors/Steps, Other), 15-16: Accessories (Awnings, Other), 17-18: Underbelly (Tires, Jacks), 19-22: Interior (Flooring, Int. Windows, Walls/Trim, Other), 23: Furniture, 24: Appliances, 25: Electrical, 26: Water, 27: HVAC, 28: Propane, 29: Safety, 30: RVIA

### Scrape parent OEM rows — fail counts as ordered array
```javascript
const rows = document.querySelectorAll('.matrix-row-parent');
// Sum fail counts per column index (0-28 = columns 2-30)
const failTotals = new Array(29).fill(0);
rows.forEach(row => {
  const cells = row.querySelectorAll('td');
  if (cells.length < 31) return;
  for (let c = 2; c < 31; c++) {
    const val = parseInt(cells[c]?.textContent?.trim()) || 0;
    failTotals[c - 2] += val;
  }
});
window._failTotals = failTotals;
JSON.stringify(failTotals);
```
Store as `window._failTotals`. Index 0 = column 2 fail total, index 1 = column 3, etc. Pair with `window._colNames` (built in Step 7) to get category names for Performance Highlights.

## STEP 7: SCRAPE CATEGORY MATRIX — % UNITS MODE

### Switch BOTH metric groups
```javascript
// Matrix toggle: LAST "% Units" button
const allBtns = Array.from(document.querySelectorAll('button.matrix-metric'));
const pctBtns = allBtns.filter(b => b.textContent.trim() === '% Units');
const matrixPctBtn = pctBtns[pctBtns.length - 1];
matrixPctBtn.click();
```
Wait 2 seconds, verify active state. Then ALSO switch the trendline group:
```javascript
const allBtns2 = Array.from(document.querySelectorAll('button.matrix-metric'));
const pctWFBtns = allBtns2.filter(b => b.textContent.trim() === '% Units w/ Fails');
if (pctWFBtns.length > 0 && !pctWFBtns[0].classList.contains('active')) {
  pctWFBtns[0].click();
}
```
Wait 2 seconds. **Both groups must be in percentage mode** for correct data across all 31 columns.

### Scrape ALL parent OEM rows — INDEX-BASED ARRAYS
⛔ Use `.matrix-row-parent` to get ONLY parent rows. Do NOT scrape all `tbody tr`.

⛔ **CRITICAL: Store cell values as an ORDERED ARRAY (indices 0-28 = columns 2-30), NOT as a named object.** This eliminates category-name matching bugs. Each array position corresponds to a fixed column index.

```javascript
const parentRows = document.querySelectorAll('.matrix-row-parent');
const oemData = [];
parentRows.forEach(row => {
  const cells = row.querySelectorAll('td');
  if (cells.length < 31) return;
  const oem = cells[0].textContent.trim().replace(/^▶/, '').trim();
  const units = parseInt(cells[1].textContent.trim()) || 0;
  // Store columns 2-30 as an ordered array: index 0 = column 2, index 1 = column 3, etc.
  const values = [];
  for (let c = 2; c < 31; c++) {
    values.push(cells[c]?.textContent?.trim() || '');
  }
  oemData.push({ oem, units, values });
});
JSON.stringify(oemData);
```
Store as `window._oemData`. Verify count === 7. If more, org filter is wrong.

**Also store the column headers as an ordered array** for use in email table headers:
```javascript
// columnHeaders was built in Step 6 — extract display names for columns 2-30
const colNames = [];
for (let c = 2; c < columnHeaders.length && c < 31; c++) {
  const h = columnHeaders[c];
  colNames.push(h.group ? h.group + ' > ' + h.name : h.name);
}
window._colNames = colNames;
JSON.stringify(colNames);
```
Store as `window._colNames`. This is the authoritative list of what each array position means.

### ⛔ Step 7b: COLUMN NAME VERIFICATION — MANDATORY
Execute this IMMEDIATELY after storing `_colNames`. Do NOT skip this step.
```javascript
const cn = window._colNames;
const checks = {
  len: cn.length,
  idx0: cn[0],    // expect contains "Shore"
  idx4: cn[4],    // expect contains "Tongue" or "Pin"
  idx12: cn[12],  // expect contains "Other"
  idx16: cn[16],  // expect contains "Jack"
  idx20: cn[20],  // expect contains "Other"
  idx21: cn[21],  // expect contains "Furniture"
  idx24: cn[24],  // expect contains "Water"
  idx26: cn[26],  // expect contains "Propane"
  idx28: cn[28]   // expect contains "RVIA"
};
JSON.stringify(checks);
```
⛔ **STOP CONDITIONS:** If `len` ≠ 29, or idx24 does NOT contain "Water", or idx26 does NOT contain "Propane", or idx28 does NOT contain "RVIA": **STOP IMMEDIATELY.** Print the full `_colNames` array. The email builder function uses hardcoded index ranges (Systems & Safety = indices 21-28). If columns are misaligned here, the entire email will have wrong data in the Category Matrix. Debug the column header parsing and fix before proceeding.

**If column count is wrong:** The two-row thead parsing may have produced extra or missing entries. Count the actual `<td>` cells in a parent row (`document.querySelector('.matrix-row-parent').querySelectorAll('td').length`) and compare to `columnHeaders.length`. They must match.

## STEP 8: SCRAPE CHECKIN (DAF) USER METRICS

⛔ **IMPORTANT:** This section shows DAF inspector metrics ONLY. You MUST deselect PDI before scraping. After deselecting PDI, wait for the page to fully reload — the user list and numbers should change to reflect DAF-only data. If the same users appear with the same numbers after toggling PDI off, it may mean all users only perform DAFs (which is valid). But VERIFY by checking that the unit counts in the user table match the DAF count in `_jobStats`, NOT the total count.

Deselect PDI so only DAF data shows:
```javascript
const jobTypeBtns = document.querySelectorAll('.job-type-toggle button, button');
const pdiBtn = Array.from(jobTypeBtns).find(b => b.textContent.trim() === 'PDI' && b.className.includes('active'));
if (pdiBtn) pdiBtn.click();
```
Wait 3 seconds. Navigate to "Trend by User" dashboard tab. Scrape Users by Location table:
```javascript
const tables = document.querySelectorAll('table');
const userTable = Array.from(tables).find(t => {
  const ths = Array.from(t.querySelectorAll('thead th')).map(th => th.textContent.trim());
  return ths.includes('User') && ths.includes('Units') && ths.includes('Avg / Unit');
});
const headers = Array.from(userTable.querySelectorAll('thead th')).map(th => th.textContent.trim());
const rows = userTable.querySelectorAll('tbody tr');
let currentLocation = '';
const userData = Array.from(rows).map(row => {
  const cells = row.querySelectorAll('td');
  const loc = cells[0]?.textContent?.trim();
  if (loc) currentLocation = loc;
  return {
    location: loc || currentLocation,
    user: cells[1]?.textContent?.trim(),
    units: cells[2]?.textContent?.trim(),
    fails: cells[3]?.textContent?.trim(),
    avgPerUnit: cells[4]?.textContent?.trim()
  };
});
```
Store as `window._userMetrics`. After scraping, re-enable PDI:
```javascript
const pdiBtn2 = Array.from(document.querySelectorAll('button')).find(b => b.textContent.trim() === 'PDI' && !b.className.includes('active'));
if (pdiBtn2) pdiBtn2.click();
```
Wait 3 seconds.

## STEP 9: SCRAPE JOB REPORTS TABLE

Navigate to Dashboards > "Job Reports" tab.

⛔ **CRITICAL: Column indices are NOT fixed.** The table may have different columns depending on the dashboard version. You MUST find columns by HEADER TEXT, not by hardcoded index.

### 9a. Discover column indices first
```javascript
const allTables = document.querySelectorAll('table');
const jrTable = Array.from(allTables).find(t => {
  const ths = Array.from(t.querySelectorAll('thead th')).map(th => th.textContent.trim());
  return ths.includes('VIN') && (ths.includes('Make') || ths.includes('Stock'));
});
if (!jrTable) throw new Error('Could not find Job Reports table');
const headers = Array.from(jrTable.querySelectorAll('thead th')).map(th => th.textContent.trim());
JSON.stringify({headers: headers, count: headers.length});
```

⛔ **STOP AND EXAMINE THE HEADERS.** Map each column you need by finding its index in the headers array:
- VIN: `headers.indexOf('VIN')`
- Stock: `headers.indexOf('Stock')`  
- Make: `headers.indexOf('Make')`
- Model: `headers.indexOf('Model')`
- Trim: `headers.indexOf('Trim')`
- Location: `headers.indexOf('Location')`
- Job Name: `headers.findIndex(h => h === 'Job Name' || h === 'Job')`
- Fails: `headers.findIndex(h => h === 'Fails' || h === 'Fail Count' || h.includes('Fail'))`
- Status: `headers.indexOf('Status')`
- Report: `headers.findIndex(h => h === 'Report' || h === 'View')`

If "Fails" column is NOT found in the headers, you MUST use fail counts from the Category Matrix (Step 6's `_failTotals`). In that case, compute per-job fails from the OEM-level fail totals and the `_makeSummary` data. Do NOT fabricate fail counts.

### 9b. Top 10 by Fails — dynamic column scraping
```javascript
const headers = Array.from(jrTable.querySelectorAll('thead th')).map(th => th.textContent.trim());
const idx = {
  vin: headers.indexOf('VIN'),
  stock: headers.indexOf('Stock'),
  make: headers.indexOf('Make'),
  model: headers.indexOf('Model'),
  trim: headers.indexOf('Trim'),
  loc: headers.indexOf('Location'),
  job: headers.findIndex(h => h === 'Job Name' || h === 'Job'),
  fails: headers.findIndex(h => h === 'Fails' || h === 'Fail Count' || h.includes('Fail')),
  status: headers.indexOf('Status'),
};
// Find the column with a link (Report/View)
const rptIdx = headers.findIndex(h => h === 'Report' || h === 'View');
// Find Logo column (first column with an img)
const logoIdx = 0; // Usually first column

const jrRows = jrTable.querySelectorAll('tbody tr');
const allJobs = [];
jrRows.forEach(row => {
  const cells = row.querySelectorAll('td');
  if (cells.length < 5) return;
  const img = cells[logoIdx]?.querySelector('img');
  let logo = img ? img.src : '';
  const link = rptIdx >= 0 ? cells[rptIdx]?.querySelector('a') : null;
  let rptLink = link?.href || '';
  if (rptLink) rptLink = rptLink.replace(/https?:\/\/[^/]+/, 'https://prodapp.getvinny.com');
  const fails = idx.fails >= 0 ? (parseInt(cells[idx.fails]?.textContent?.trim() || '0') || 0) : 0;
  const statusEl = idx.status >= 0 ? cells[idx.status]?.querySelector('span') : null;
  allJobs.push({
    logo,
    vin: idx.vin >= 0 ? (cells[idx.vin]?.getAttribute('title') || cells[idx.vin]?.textContent?.trim() || '') : '',
    stock: idx.stock >= 0 ? (cells[idx.stock]?.textContent?.trim() || '') : '',
    make: idx.make >= 0 ? (cells[idx.make]?.textContent?.trim() || '') : '',
    model: idx.model >= 0 ? (cells[idx.model]?.textContent?.trim() || '') : '',
    trim: idx.trim >= 0 ? (cells[idx.trim]?.textContent?.trim() || '') : '',
    jobName: idx.job >= 0 ? (cells[idx.job]?.textContent?.trim() || '') : '',
    fails,
    status: statusEl?.textContent?.trim() || (idx.status >= 0 ? cells[idx.status]?.textContent?.trim() || '' : ''),
    reportLink: rptLink
  });
});
allJobs.sort((a, b) => b.fails - a.fails);
window._top10 = allJobs.slice(0, 10);
window._allJobs = allJobs; // Keep full list for Step 9c
JSON.stringify({totalJobs: allJobs.length, top10Fails: window._top10.map(j => j.fails), failsColFound: idx.fails >= 0, failsColIdx: idx.fails});
```

⛔ **SANITY CHECK after scraping Top 10:**
- If `failsColFound` is false: the Fails column was not in the table headers. You CANNOT use fail counts. Set fails to 0 and note in the email that fail counts are from the Category Matrix aggregate, not per-job.
- If ALL top 10 have the EXACT same fail count: this is HIGHLY suspicious. Verify by examining 2-3 rows visually (take a screenshot and check). If fails are all identical AND suspiciously round, the column index is likely wrong.
- If `totalJobs` is 0: something went wrong with scraping — go back and verify org filter and table loading (see Step 5).

Extract in two batches of 5 if output is truncated.

### 9c. Units Inspected by Make
```javascript
const allJobs = window._allJobs;
const makeStats = {};
const uniqueVins = new Set();
allJobs.forEach(j => {
  uniqueVins.add(j.vin);
  if (!makeStats[j.make]) makeStats[j.make] = { units: new Set(), dafs: 0, pdis: 0, totalFails: 0 };
  makeStats[j.make].units.add(j.vin);
  if (j.jobName.includes('Dealer Acceptance')) makeStats[j.make].dafs++;
  else if (j.jobName.includes('Pre-Delivery')) makeStats[j.make].pdis++;
  makeStats[j.make].totalFails += j.fails;
});
const makeSummary = Object.entries(makeStats).map(([make, s]) => ({
  make, units: s.units.size, dafs: s.dafs, pdis: s.pdis,
  totalJobs: s.dafs + s.pdis, totalFails: s.totalFails,
  avgFailsPerUnit: (s.totalFails / s.units.size).toFixed(1)
})).sort((a, b) => b.units - a.units);
```
Store as `window._makeSummary`. If many rows (300+), scrape in batches using `window._jrBatch1`, `_jrBatch2`, etc.

## STEP 10: GET LATEST UPDATES FROM TRIP
Search Gmail for the most recent email from trip@getvinny.com with subject containing "Weekly Updates" or "FTRV". 

⛔ **Copy bullet points VERBATIM** — do NOT rephrase, condense, shorten, or paraphrase Trip's updates. Each bullet point from the original email should appear exactly as written (word-for-word) in the report's Updates & Priorities section. The only acceptable changes are fixing obvious typos or adding HTML formatting tags. If Trip wrote 8 bullet points, the report must have 8 bullet points with the same wording.

If no updates email found, use: "No updates this week."

## STEP 11: SCRAPE TRENDLINE — PERSISTENT ISSUES TABLE

⚠️ Do this LAST among scraping steps — it changes the date range filter.

### 11a. Change date range to Year to Date
```javascript
const ytdBtn = Array.from(document.querySelectorAll('button')).find(b => b.textContent.trim() === 'Year to Date');
if (ytdBtn) ytdBtn.click();
```
Wait 5 seconds.

### 11b. Navigate to Trendline and configure
Click "Trendline" dashboard tab, then:
```javascript
// Recurring Categories source toggle (second button)
document.querySelectorAll('.trendline-source-toggle button')[1].click();
// % Units w/ Fails mode (third button)
document.querySelectorAll('.trendline-mode-toggle button')[2].click();
```

### 11c. Set Persistent Issues table to "All"
```javascript
const piSelect = document.getElementById('pi-week-window');
const nativeSet = Object.getOwnPropertyDescriptor(window.HTMLSelectElement.prototype, 'value').set;
nativeSet.call(piSelect, '0');
piSelect.dispatchEvent(new Event('change', { bubbles: true }));
```
Wait 2 seconds.

### 11d. Scrape the Persistent Issues table
```javascript
const table = document.getElementById('persistent-issues-table');
const rows = table.querySelectorAll('tbody tr');
const persistentIssues = Array.from(rows).map(row => {
  const cells = row.querySelectorAll('td');
  return {
    category: cells[0]?.textContent?.trim(),
    weeksPresent: cells[1]?.textContent?.trim(),
    avgPctUnits: cells[2]?.textContent?.trim(),
    trend: cells[3]?.textContent?.trim(),
    trendClass: cells[3]?.className?.trim()
  };
});
JSON.stringify(persistentIssues);
```
Store as `window._persistentIssues`.

**Data fields:**
- Weeks Present: e.g., "15/15" means appeared every week
- Avg % Units: average percentage of units affected
- Trend: arrow direction (up = worsening, down = improving, right = stable)
- trendClass: CSS class for color coding
