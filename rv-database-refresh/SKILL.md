---
name: rv-database-refresh
description: Refresh the canonical RV make/model/floorplan database at ~/Desktop/RV_Manufacturer_Canonical_List.xlsx. Full workflow: (1) light web-search discovery of net-new OEMs and brands not yet in the file; (2) per-OEM enumeration on official manufacturer websites for new model lines, new floorplans on existing lines, discontinued models, and model-year updates; (3) Sub-Line/Series internal misattribution audit to catch rows assigned to the wrong parent or division. Timestamped backups are created before each batch write. Rows with ambiguous or unverifiable data are flagged with confidence tiers and Needs Verification = Yes. Triggers: "refresh the RV database", "scan for new RV manufacturers", "find new RV models and floorplans", "update the canonical list".
---

Refresh the RV manufacturer canonical database at ~/Desktop/RV_Manufacturer_Canonical_List.xlsx.

1. Open the existing spreadsheet and review current contents (sheets, row counts per manufacturer).

2. For each manufacturer already in the file, visit their official website and check for:
   - New model lines added since last refresh
   - New floorplans added to existing model lines
   - Discontinued models that should be marked as such
   - Model year updates (new MY releases)

3. Search for any NEW RV manufacturers or brands that may have launched or been missed previously. Check industry news, RVIA membership lists, and manufacturer websites.

4. Add any new data using the same column format (Parent Company, Brand/Division, Model Line, Sub-Line/Series, Floorplan, RV Type, RV Class, Model Year, Status, Needs Verification, Notes).

5. Before writing each batch of changes, create a timestamped backup of the spreadsheet (e.g. RV_Manufacturer_Canonical_List_2026-04-15T1430.xlsx) so every batch is recoverable.

6. Apply confidence tiers to new rows:
   - High: data confirmed directly from official manufacturer website
   - Medium: data from reputable dealer or industry source, not yet verified on OEM site
   - Low / Needs Verification = Yes: data from secondary sources only or structurally ambiguous

7. Perform a Sub-Line/Series misattribution audit: scan existing rows for Sub-Line/Series values that appear to belong to a different Brand/Division or Model Line than currently assigned, flag any suspect rows with Needs Verification = Yes and a note explaining the potential misattribution.

8. Update the Verification Log sheet with what was checked, what changed, confidence tiers applied, and the date.

9. Save back to ~/Desktop/RV_Manufacturer_Canonical_List.xlsx.

10. Report a summary of changes: new manufacturers added, new models/floorplans found, discontinued items flagged, misattribution flags set, and updated total counts.

IMPORTANT: Never fabricate data — scrape live from manufacturer websites only. Do NOT use sub-agents for scraping.
