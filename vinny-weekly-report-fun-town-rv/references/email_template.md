# Email Template Guide — Fun Town RV Weekly Report

This file contains the complete email builder function and Executive Summary instructions. Read this BEFORE starting Step 12.

---

## STEP 12: GENERATE EXECUTIVE SUMMARY

Using Persistent Issues data (Step 11) + Performance Highlights data (Steps 6-7), generate an executive summary with 3 subsections:

### Structure

**Top 3 Problem Areas:** Take the top 3 rows from Persistent Issues (highest recurrence + severity), EXCLUDING "Interior Structure" / "Interior > Other". Interior Structure is largely fit-and-finish defects every dealer expects — not insightful.

Add a footnote: _* Interior Structure (Other) continues to appear in X% of units (Y of Z weeks). These are primarily fit-and-finish items expected at the dealer level._

For each of the Top 3, state the category name, Avg % Units, Weeks Present, and one sentence connecting to this week's fail count data.

**Trends: Improving & Worsening:** From Persistent Issues table:
- Worsening: categories with trend-up arrows, list each with Avg % Units
- Improving: categories with trend-down arrows, list each with Avg % Units
- If all stable: "All categories held steady this week."

**Recommended Focus:** 1-2 actionable areas based on high-severity + worsening trend. If nothing worsening, focus on highest-severity persistent issues.

**Tone:** Professional, concise, 8-12 sentences max. Write as Ranger providing data-backed insights. Do NOT imply limited data or early-stage trends — Fun Town RV has months of data.

### Store the Executive Summary

After writing the executive summary text, build it as an HTML string and store as `window._execSummaryHtml` on the vinnyreporting tab:
```javascript
window._execSummaryHtml = `<div style="background:#f8fafc;border-left:4px solid #2d3a3a;padding:16px 20px;margin-bottom:24px;border-radius:0 8px 8px 0;">
  <h2 style="color:#2d3a3a;font-size:18px;margin:0 0 12px;">Executive Summary</h2>
  <h3 style="color:#475569;font-size:14px;margin:12px 0 4px;">Top 3 Problem Areas</h3>
  <p style="font-size:13px;line-height:1.6;color:#334155;margin:0 0 8px;">[YOUR TEXT HERE]</p>
  <p style="font-size:12px;line-height:1.5;color:#64748b;font-style:italic;margin:4px 0 8px;">* Interior Structure footnote...</p>
  <h3 style="color:#475569;font-size:14px;margin:12px 0 4px;">Trends: Improving & Worsening</h3>
  <p style="font-size:13px;line-height:1.6;color:#334155;margin:0 0 8px;">[YOUR TEXT HERE]</p>
  <h3 style="color:#475569;font-size:14px;margin:12px 0 4px;">Recommended Focus</h3>
  <p style="font-size:13px;line-height:1.6;color:#334155;margin:0;">[YOUR TEXT HERE]</p>
</div>`;
```

Also store the Updates HTML:
```javascript
window._updatesHtml = `<ul style="font-size:13px;line-height:1.8;color:#334155;padding-left:20px;margin:8px 0;">
  <li>Bullet point 1 from Trip's email</li>
  <li>Bullet point 2...</li>
</ul>`;
// If no updates: window._updatesHtml = '<p style="font-size:13px;color:#64748b;">No updates this week.</p>';
```

---

## STEP 13: BUILD THE HTML EMAIL

⛔ **THIS STEP USES A DETERMINISTIC BUILDER FUNCTION.** Do NOT write your own HTML building code. Follow the exact steps below.

### 13a. Transfer ALL data to Gmail tab

On the vinnyreporting tab, serialize all data:
```javascript
window._transferPayload = JSON.stringify({
  jobStats: window._jobStats,
  failTotals: window._failTotals,
  colNames: window._colNames,
  oemData: window._oemData,
  persistentIssues: window._persistentIssues,
  userMetrics: window._userMetrics,
  top10: window._top10,
  makeSummary: window._makeSummary,
  execSummaryHtml: window._execSummaryHtml,
  updatesHtml: window._updatesHtml
});
window._transferPayload.length;
```

If payload > 50000 chars, split into chunks:
```javascript
const p = window._transferPayload;
window._chunk1 = p.slice(0, 40000);
window._chunk2 = p.slice(40000);
```

Switch to Gmail tab and parse:
```javascript
// If single chunk:
const d = JSON.parse(PASTE_THE_PAYLOAD_HERE);
// If split:
// const d = JSON.parse(window._c1 + window._c2);

window._jobStats = d.jobStats;
window._failTotals = d.failTotals;
window._colNames = d.colNames;
window._oemData = d.oemData;
window._persistentIssues = d.persistentIssues;
window._userMetrics = d.userMetrics;
window._top10 = d.top10;
window._makeSummary = d.makeSummary;
window._execSummaryHtml = d.execSummaryHtml;
window._updatesHtml = d.updatesHtml;
```

### 13b. ⛔ COLUMN VERIFICATION — MANDATORY

After data is on Gmail tab, IMMEDIATELY verify column alignment:
```javascript
const cn = window._colNames;
const checks = {
  len: cn.length === 29,
  idx0: cn[0].includes('Shore'),
  idx4: cn[4].includes('Tongue') || cn[4].includes('Pin'),
  idx12: cn[12].includes('Other'),
  idx16: cn[16].includes('Jack'),
  idx20: cn[20].includes('Other'),
  idx21: cn[21].includes('Furniture'),
  idx24: cn[24].includes('Water'),
  idx26: cn[26].includes('Propane'),
  idx28: cn[28].includes('RVIA')
};
const fails = Object.entries(checks).filter(([k,v]) => !v);
fails.length === 0 ? 'COLUMN ALIGNMENT VERIFIED' : 'ALIGNMENT FAILED: ' + JSON.stringify(fails) + ' colNames: ' + JSON.stringify(cn);
```

⛔ If this check fails, STOP. Print the full `_colNames` array. The hardcoded TABLE_GROUPS indices will produce wrong data. You must identify where the actual columns are and adjust, or re-scrape.

### 13c. Execute the Email Builder Function

⛔ **Execute this EXACT function on the Gmail tab. Do NOT modify it.** If it's too long for one JS call, split at the marked SPLIT POINT comments, storing each piece and concatenating at the end.

```javascript
(function(){
const D = {
  js: window._jobStats, ft: window._failTotals, cn: window._colNames,
  od: window._oemData, pi: window._persistentIssues, um: window._userMetrics,
  t10: window._top10, ms: window._makeSummary,
  esh: window._execSummaryHtml, uh: window._updatesHtml
};

// Date range from jobStats or compute
const today = new Date();
const weekAgo = new Date(today); weekAgo.setDate(today.getDate() - 7);
const fmt = d => (d.getMonth()+1).toString().padStart(2,'0') + '/' + d.getDate().toString().padStart(2,'0');
const dateRange = fmt(weekAgo) + ' - ' + fmt(today) + '/' + today.getFullYear();

// Helpers
const hc = v => {
  if (!v || v==='—' || v==='\u2014' || v==='') return '#ffffff';
  const n = parseFloat(v); if (isNaN(n) || n===0) return '#ffffff';
  return n<=5?'#dcfce7':n<=15?'#bbf7d0':n<=30?'#fef9c3':n<=50?'#fde68a':n<=70?'#fdba74':n<=85?'#fca5a5':'#f87171';
};
const dv = v => (!v || v==='—' || v==='\u2014' || v==='') ? '&mdash;' : v;
const sh = t => '<h2 style="color:#2d3a3a;font-size:16px;border-bottom:2px solid #e2e8f0;padding-bottom:8px;margin:24px 0 12px;">'+t+'</h2>';
const th = t => '<th style="color:white;font-size:11px;padding:6px 8px;text-align:center;">'+t+'</th>';
const thL = t => '<th style="color:white;font-size:11px;padding:6px 8px;text-align:left;">'+t+'</th>';
const td = (v,a) => '<td style="padding:'+((a||'')===''?'4':'6')+'px '+ ((a||'')===''?'6':'8') +'px;font-size:'+ ((a||'')===''?'11':'12') +'px;text-align:'+((a||'')||'center')+';">'+v+'</td>';
const tdBg = (v,bg) => '<td bgcolor="'+bg+'" style="padding:4px 6px;font-size:11px;text-align:center;">'+v+'</td>';

let html = '';

// ===== SECTION 1: Header Banner =====
html += '<div style="background:#2d3a3a;border-radius:10px;padding:30px 20px;text-align:center;margin-bottom:20px;">';
html += '<h1 style="color:#fff;font-size:24px;margin:0 0 6px;">Fun Town RV &mdash; Weekly Report</h1>';
html += '<p style="color:#94a3b8;font-size:14px;margin:0;">Week of ' + dateRange + '</p></div>';

// ===== SECTION 2: Summary Line =====
html += '<p style="text-align:center;font-size:14px;margin:0 0 20px;"><strong>' + D.js.uniqueUnits + ' unique units inspected</strong> (' + D.js.dafJobs + ' DAFs + ' + D.js.pdiJobs + ' PDIs, totaling ' + D.js.totalJobs + ' jobs)</p>';

// ===== SECTION 3: Executive Summary =====
html += D.esh;

// ===== SECTION 4: Performance Highlights =====
html += sh('Performance Highlights');
const totalUnits = D.od.reduce((s,o) => s + o.units, 0);
const perfData = D.cn.map((name, i) => {
  let weighted = 0;
  D.od.forEach(o => {
    const v = o.values[i];
    const num = (v==='—'||v==='\u2014'||v===''||!v) ? 0 : (parseFloat(v)||0);
    weighted += num * o.units / 100;
  });
  const pct = totalUnits > 0 ? +(weighted / totalUnits * 100).toFixed(1) : 0;
  // KEEP the full "Group > SubCategory" name — do NOT strip the parent prefix.
  // Stripping causes ambiguous entries like three different "Other" rows.
  return { name: name, fails: D.ft[i], pct: pct };
}).filter(d => d.fails > 0).sort((a,b) => b.fails - a.fails).slice(0, 15);

html += '<table style="border-collapse:collapse;width:100%;font-size:12px;"><thead><tr style="background:#2d3a3a;">';
html += thL('Category') + th('Fail Count') + th('% of Units');
html += '</tr></thead><tbody>';
perfData.forEach((d, i) => {
  const bg = i%2===0 ? '#ffffff' : '#f8fafc';
  const dot = d.pct >= 70 ? '#ef4444' : d.pct >= 15 ? '#f59e0b' : '#22c55e';
  html += '<tr style="background:'+bg+';"><td style="padding:6px 8px;font-size:12px;"><span style="color:'+dot+';font-size:16px;">&#9679;</span> '+d.name+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;text-align:center;">'+d.fails+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;text-align:center;">'+d.pct+'%</td></tr>';
});
html += '</tbody></table>';

// ===== SECTION 5: Category Matrix (5 sub-tables) =====
html += sh('Category Matrix (% of Units w/ Fails)');
const TG = [
  {t:'General',s:0,e:4},
  {t:'Exterior',s:5,e:12},
  {t:'Accessories &amp; Underbelly',s:13,e:16},
  {t:'Interior',s:17,e:20},
  {t:'Systems &amp; Safety',s:21,e:28}
];
TG.forEach(g => {
  html += '<h3 style="color:#475569;font-size:14px;margin:16px 0 8px;">'+g.t+'</h3>';
  html += '<table style="border-collapse:collapse;width:100%;font-size:12px;"><thead><tr style="background:#2d3a3a;">';
  html += thL('OEM') + th('Units');
  for (let i = g.s; i <= g.e; i++) {
    let n = D.cn[i]; if (n && n.includes(' > ')) n = n.split(' > ')[1]; html += th(n||'');
  }
  html += '</tr></thead><tbody>';
  D.od.forEach((o, idx) => {
    const bg = idx%2===0 ? '#ffffff' : '#f8fafc';
    html += '<tr style="background:'+bg+';"><td style="padding:4px 6px;font-size:11px;font-weight:600;">'+o.oem+'</td>';
    html += '<td style="padding:4px 6px;font-size:11px;text-align:center;">'+o.units+'</td>';
    for (let i = g.s; i <= g.e; i++) {
      html += tdBg(dv(o.values[i]), hc(o.values[i]));
    }
    html += '</tr>';
  });
  html += '</tbody></table>';
});

// ===== SECTION 6: Persistent Issues =====
html += sh('Persistent Issues (Year to Date)');
html += '<table style="border-collapse:collapse;width:100%;font-size:12px;"><thead><tr style="background:#2d3a3a;">';
html += thL('Category') + th('Weeks Present') + th('Avg % Units') + th('Trend');
html += '</tr></thead><tbody>';
D.pi.forEach((p, i) => {
  const bg = i%2===0 ? '#ffffff' : '#f8fafc';
  const tc = p.trendClass || p.trend || '';
  const arrow = tc.includes('up') ? '<span style="color:#ef4444;font-weight:bold;">&#8593;</span>'
    : tc.includes('down') ? '<span style="color:#22c55e;font-weight:bold;">&#8595;</span>'
    : '<span style="color:#94a3b8;">&#8594;</span>';
  html += '<tr style="background:'+bg+';"><td style="padding:6px 8px;font-size:12px;">'+p.category+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;text-align:center;">'+p.weeksPresent+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;text-align:center;">'+p.avgPctUnits+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;text-align:center;">'+arrow+'</td></tr>';
});
html += '</tbody></table>';

// ===== SPLIT POINT 1 — if too long, store html so far as window._partA and start new block =====

// ===== SECTION 7: DAF User Metrics =====
html += sh('Checkin (DAF) User Metrics');
html += '<table style="border-collapse:collapse;width:100%;font-size:12px;"><thead><tr style="background:#2d3a3a;">';
html += thL('Inspector') + thL('Location') + th('Units') + th('Avg Fails/Unit');
html += '</tr></thead><tbody>';
D.um.forEach((u, i) => {
  const bg = i%2===0 ? '#ffffff' : '#f8fafc';
  html += '<tr style="background:'+bg+';"><td style="padding:6px 8px;font-size:12px;">'+u.user+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;">'+(u.location||'')+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;text-align:center;">'+(u.units||'')+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;text-align:center;">'+(u.avgPerUnit||'')+'</td></tr>';
});
html += '</tbody></table>';

// ===== SECTION 8: Top 10 Jobs =====
html += sh('Top 10 Jobs by Fail Count');
html += '<table style="border-collapse:collapse;width:100%;font-size:12px;"><thead><tr style="background:#2d3a3a;">';
html += th('') + thL('Make') + thL('Model') + th('Stock #') + thL('Job') + th('Fails') + th('Status') + th('Report');
html += '</tr></thead><tbody>';
D.t10.forEach((j, i) => {
  const bg = i%2===0 ? '#ffffff' : '#f8fafc';
  const logoHtml = j.logo ? '<img src="'+j.logo+'" style="width:24px;height:24px;vertical-align:middle;" alt="" />' : '';
  const failBadge = '<span style="background:#FEE2E2;color:#991B1B;padding:2px 8px;border-radius:12px;font-size:11px;font-weight:600;">'+j.fails+'</span>';
  const st = j.status || '';
  const stColor = st.includes('Finalize') ? 'background:#D1FAE5;color:#065F46'
    : st.includes('Complete') ? 'background:#DBEAFE;color:#1E40AF'
    : 'background:#FEF3C7;color:#92400E';
  const stBadge = '<span style="'+stColor+';padding:2px 8px;border-radius:12px;font-size:11px;">'+st+'</span>';
  const model = (j.model||'') + (j.trim ? ' '+j.trim : '');
  const rpt = j.reportLink ? '<a href="'+j.reportLink+'" style="color:#4a7c7e;font-weight:bold;">View</a>' : '';
  html += '<tr style="background:'+bg+';">';
  html += '<td style="padding:4px 6px;text-align:center;">'+logoHtml+'</td>';
  html += '<td style="padding:4px 6px;font-size:11px;">'+j.make+'</td>';
  html += '<td style="padding:4px 6px;font-size:11px;">'+model+'</td>';
  html += '<td style="padding:4px 6px;font-size:11px;text-align:center;">'+(j.stock||'')+'</td>';
  html += '<td style="padding:4px 6px;font-size:11px;">'+(j.jobName||'')+'</td>';
  html += '<td style="padding:4px 6px;text-align:center;">'+failBadge+'</td>';
  html += '<td style="padding:4px 6px;text-align:center;">'+stBadge+'</td>';
  html += '<td style="padding:4px 6px;text-align:center;">'+rpt+'</td></tr>';
});
html += '</tbody></table>';
html += '<p style="font-size:11px;color:#64748b;margin:4px 0 0;text-align:center;">Showing top 10 of '+D.t10.length+' jobs by fail count. Full job-level detail at <a href="https://www.vinnyreporting.com/" style="color:#2d3a3a;">vinnyreporting.com</a></p>';

// ===== SECTION 9: Units by Make =====
const totalMakeUnits = D.ms.reduce((s,m) => s + m.units, 0);
const totalMakeJobs = D.ms.reduce((s,m) => s + m.totalJobs, 0);
html += sh('Units Inspected by Make (' + totalMakeUnits + ' units | ' + totalMakeJobs + ' jobs)');
html += '<table style="border-collapse:collapse;width:100%;font-size:12px;"><thead><tr style="background:#2d3a3a;">';
html += thL('Make / Brand') + th('Units') + th('DAFs') + th('PDIs') + th('Total Jobs') + th('Total Fails') + th('Avg Fails / Unit');
html += '</tr></thead><tbody>';
D.ms.forEach((m, i) => {
  const bg = i%2===0 ? '#ffffff' : '#f8fafc';
  html += '<tr style="background:'+bg+';"><td style="padding:6px 8px;font-size:12px;font-weight:600;">'+m.make+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;text-align:center;">'+m.units+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;text-align:center;">'+m.dafs+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;text-align:center;">'+m.pdis+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;text-align:center;">'+m.totalJobs+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;text-align:center;">'+m.totalFails+'</td>';
  html += '<td style="padding:6px 8px;font-size:12px;text-align:center;">'+m.avgFailsPerUnit+'</td></tr>';
});
html += '</tbody></table>';

// ===== SECTION 10: Updates & Priorities =====
html += sh('Updates &amp; Priorities');
html += D.uh;

// ===== SECTION 11: Footer =====
html += '<hr style="border:none;border-top:1px solid #e2e8f0;margin:24px 0 12px;">';
html += '<p style="font-size:12px;color:#94a3b8;text-align:center;">Generated by Ranger | <a href="https://www.vinnyreporting.com/">vinnyreporting.com</a></p>';

window._fullHtml = html;
'Built ' + html.length + ' chars with ' + TG.length + ' matrix tables';
})();
```

⛔ If the function is too long for a single `javascript_tool` call, split at the SPLIT POINT comment. Store the first half's `html` as `window._partA`, then in the next call start with `let html = window._partA;` and continue from the split point.

### 13d. Post-Build Verification

⛔ **MANDATORY — do NOT skip.**

**Part A — Section Order Check:**
```javascript
const html = window._fullHtml;
const markers = [
  {n:'Header',t:'Weekly Report'},{n:'Summary',t:'unique units inspected'},
  {n:'ExecSum',t:'Executive Summary'},{n:'PerfHL',t:'Performance Highlights'},
  {n:'CatMatrix',t:'Shore'},{n:'PersIssues',t:'Persistent Issues'},
  {n:'DAFUsers',t:'Checkin'},{n:'Top10',t:'Top 10'},{n:'UnitsMake',t:'Units Inspected by Make'},
  {n:'Updates',t:'Updates'},{n:'Footer',t:'Generated by Ranger'}
];
const pos = markers.map(m => ({n:m.n, p:html.indexOf(m.t)}));
const miss = pos.filter(p => p.p===-1).map(p=>p.n);
const ooo = pos.filter((p,i) => i>0 && p.p<=pos[i-1].p).map(p=>p.n);
JSON.stringify({ok:miss.length===0&&ooo.length===0, missing:miss, outOfOrder:ooo, size:html.length});
```

**Part B — Systems & Safety Data Spot-Check:**
```javascript
const tmp = document.createElement('div'); tmp.innerHTML = window._fullHtml;
const allTables = tmp.querySelectorAll('table');
// Find Systems & Safety table (last category matrix table, contains "Furniture" header)
let ssTable = null;
allTables.forEach(t => {
  const ths = Array.from(t.querySelectorAll('th')).map(h=>h.textContent.trim());
  if (ths.includes('Furniture') || ths.includes('RVIA')) ssTable = t;
});
if (!ssTable) { 'ERROR: Systems & Safety table not found'; } else {
  const headers = Array.from(ssTable.querySelectorAll('thead th')).map(h=>h.textContent.trim());
  const rows = ssTable.querySelectorAll('tbody tr');
  const sample = Array.from(rows).slice(0,3).map(r => {
    const cells = r.querySelectorAll('td');
    return {oem:cells[0]?.textContent?.trim(), units:cells[1]?.textContent?.trim(),
      firstVal:cells[2]?.textContent?.trim(), lastVal:cells[cells.length-1]?.textContent?.trim()};
  });
  JSON.stringify({headers:headers, sampleRows:sample});
}
```

Then switch to vinnyreporting tab and compare:
```javascript
// Get the same OEMs from stored data for comparison
const check = window._oemData.slice(0,3).map(o => ({
  oem: o.oem, units: o.units,
  furniture: o.values[21], appliances: o.values[22], electrical: o.values[23],
  water: o.values[24], hvac: o.values[25], propane: o.values[26],
  safety: o.values[27], rvia: o.values[28]
}));
JSON.stringify(check);
```

⛔ Compare the Systems & Safety headers and values from the built HTML against the source data. If ANY value doesn't match: STOP, identify the bug, fix, rebuild. The most common bug is values shifting — check that Water values in the HTML match Water values in the source, not RVIA or another column.

**Part C — HTML Size Check:**
```javascript
window._fullHtml.length < 90000 ? 'SIZE OK: ' + window._fullHtml.length : 'TOO LARGE: ' + window._fullHtml.length + ' - must reduce';
```

Only proceed to Step 14 after Parts A, B, and C all pass.

---

## STEP 14: INJECT AND SEND

### Inject HTML into Gmail compose body:
```javascript
if (!window._rangerPolicy) {
  window._rangerPolicy = trustedTypes.createPolicy('ranger', { createHTML: (s) => s });
}
const body = document.querySelector('div[aria-label="Message Body"]');
body.focus();
const range = document.createRange();
range.selectNodeContents(body);
const sel = window.getSelection();
sel.removeAllRanges();
sel.addRange(range);
sel.collapseToStart();
document.execCommand('insertHTML', false, window._rangerPolicy.createHTML(window._fullHtml));
```

### Send:
```javascript
const sendBtn = document.querySelector('div[aria-label="Send"][role="button"]')
  || document.querySelector('div[data-tooltip="Send"]');
if (sendBtn) sendBtn.click();
else throw new Error('Send button not found');
```
Click ONCE. Wait 3 seconds and verify "Message sent" appears or compose window closes.

---

## IMPORTANT NOTES
- Gmail tabs may not reach document_idle — use JavaScript tool for all Gmail DOM interactions
- If data is too large for a single JS tool output, scrape in batches via window variables
- If authentication has expired, re-authenticate via magic link flow
- The Ranger Miles signature is auto-populated in compose — insertHTML preserves it
- Logo images in Top 10 Jobs use original URLs as scraped — do NOT rewrite hostnames
- Report "View" links point to prodapp.getvinny.com — use exact URLs from scraped data
- Units by Make: Avg Fails/Unit = totalFails / uniqueUnits (NOT per job)
- Category Matrix shows parent OEM names only — filter out sub-model rows
- Do NOT include "Interior Structure (Other)" / "Interior > Other" in Top 3 Problem Areas — always footnote instead
- Do NOT imply limited data or early-stage trends — Fun Town RV has months of data
