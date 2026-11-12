# Material Radar — Server Component Supply Tracking

A browser-based supply planning prototype that converts inventory, open purchase orders and open demand into a 28-day component shortage tracking view.

**Project owner:** Yu-Hao Tien  
**Status:** Portfolio / beta prototype. Not validated for production planning.  
**Live preview:** https://semi-risk-calvin.warmhero7.chatgpt.site — currently private; access may be restricted.

## Problem and intended users

Production planners and procurement teams need to see which server components may run short, when a shortage starts, and which items need follow-up. Material Radar makes those questions visible without requiring an ERP integration.

## Features

- Demo mode with 24 illustrative CPU, GPU, memory, storage, networking, power, cooling and chassis/board components.
- CSV and XLSX import, workbook sheet selection and explicit column mapping.
- Validation for required values, duplicate line IDs, invalid dates, quantities and inconsistent units.
- Daily projected inventory, first shortage date, peak shortfall and explainable risk scores.
- Demand-growth and receipt-delay scenarios.
- Search, category/risk filters, component details and filtered CSV export.
- Downloadable Excel input template and exported tracking snapshots for comparison.

## Run locally

The application is static HTML, CSS and JavaScript. No build step or backend is required.

```bash
python3 -m http.server 8000 --directory dist
```

Open http://localhost:8000 in a modern browser. The XLSX parser is included locally; no external CDN is required at runtime.

## Try the upload workflow

1. Select **Upload Your Data**.
2. Download the Excel template.
3. Select the workbook in each of the three input panels; choose its corresponding inventory, purchase or demand sheet.
4. Confirm column mappings, baseline and source completeness.
5. Select **Validate & generate tracking**.
6. Review component details and export results or a snapshot.

The template contains sample data only. Replace it with authorized data for a pilot.

## Input contract

| Dataset | Required fields | Optional fields |
| --- | --- | --- |
| inventory | part, quantity, unit | name, category |
| purchase | line_id, part, quantity, available_date, unit | supplier |
| demand | line_id, part, quantity, demand_date, unit | — |

Use one opening inventory row per part and unique line IDs within each transaction dataset. Aggregate locations first. Quantities must be nonnegative; dates must be YYYY-MM-DD or Excel date cells. Inventory is the available opening balance before baseline-day receipts and demand. Use open supply and demand only; avoid counting allocated demand twice. Limits: 5 MB/file and 10,000 data rows/dataset.

Inventory is required. Missing demand produces an Unassessed result. Missing purchase data means no incoming supply is assumed. Other dataset completeness remains the user's responsibility.

## Calculation

Daily balance = previous balance + available receipts − demand.

The horizon includes the baseline date and the next 27 days. Past-due open demand is applied on the baseline date. Overdue purchase lines are excluded until their availability dates are reconfirmed. Scenario delay applies to future receipts; demand growth applies to open demand. Fractional scenario quantities are permitted.

Heuristic score (not a probability): projected shortage +50; first shortage within seven days +15; otherwise below the seven-day average-demand buffer +20. High ≥65, Medium ≥30, Low <30. This version does not infer supplier performance or quality risk from product names.

## Privacy and storage

Imported source files are parsed in the browser and are not sent to an application backend by this implementation. No account, cloud database or automatic persistence is included. Refreshing clears session data. Users must export files to retain results. Hosting services may independently process normal page request metadata. Exported snapshots and reports can contain sensitive information and should be handled accordingly.

## Scope and validation status

This is a deterministic supply/demand prototype, not a trained machine-learning model. CPU/GPU family labels are illustrative and do not establish orderable SKUs or a compatible server BOM. Demo operational data and suppliers are fictional.

Programmatic checks covered inventory arithmetic, shortage dates, scoring, invalid inputs, overdue supply, past-due demand, missing datasets, XLSX round-trip, demo/upload isolation and UI rendering through DOM stubs. Full browser visual and end-to-end QA, production security review, historical backtesting and real-user pilots remain outstanding.

No BOM roll-up, shared-stock allocation, server-build impact calculation, live ERP connection or scheduled refresh is provided. Compare snapshots only within the same project and compatible units; shifting baselines changes the observation window.

## Project contribution and development process

Calvin defined the operational problem, target user workflow, server-component focus and iterative product requirements based on production planning experience. The prototype was developed iteratively with AI-assisted implementation. Operational effectiveness and adoption have not yet been established; these require documented pilots and validation.

## Roadmap

- Browser end-to-end testing and accessibility review.
- Pilot feedback and reproducible comparisons against existing planning workflows.
- BOM and affected-order analysis with explicit stock allocation rules.
- Optional authenticated workspaces and persistent tracking, subject to data-access design.

## Repository layout

- `dist/index.html`: application interface and methodology.
- `dist/app.js`: import, mapping, dashboard and export interactions.
- `dist/engine.js`: validation and daily projection logic.
- `dist/catalog.js`: illustrative demo catalog.
- `dist/style.css`: responsive styling.
- `dist/xlsx.full.min.js`: SheetJS CE 0.20.3 distribution.
- `dist/SHEETJS-LICENSE.txt`: third-party license.

The project's own code has no open-source license selected yet. Third-party code retains its own license. Product names belong to their respective owners; no endorsement is implied.
