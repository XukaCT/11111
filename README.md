# Autonomous Supermarket

A 60-day supermarket simulation with autonomous inventory decisions, persistent operating history and commercial reporting.

**Subject:** CSE3CWA / CSE5006, Assignment 4  
**Author:** Sang Nguyen 
**Student ID:** 22261623 
**Submitted Run ID:** [ACTUAL RUN ID]

## Purpose and scope

The project models a small autonomous supermarket near Swanston Street and Melbourne Town Hall in Melbourne. It generates customer demand, records valid purchases, tracks stock and expiry, and uses a rule-based Store Brain to place affordable replenishment orders.

The website lets a remote manager inspect one complete 60-day run, its business results, daily operating evidence and integrity checks. It is an educational prototype, not a validated forecast of a real supermarket's performance.

## Technology and prerequisites

| Component | Technology |
|---|---|
| Frontend | React with Vite |
| Backend | Node.js with Express |
| Persistent storage | SQLite |
| Backend SQLite driver | `better-sqlite3` |
| Catalogue input | `backend/products.json` |

Tested environment:

- **Operating system:** [OS AND VERSION USED FOR THE CLEAN-COPY TEST].
- **Node.js:** [EXACT TESTED VERSION FROM `node --version`].
- **npm:** [EXACT TESTED VERSION FROM `npm --version`].
- **Browser:** [BROWSER AND VERSION USED FOR TESTING].

Use the tested Node.js version above because the SQLite driver includes a native dependency. Install dependencies from each folder's `package.json`; include existing lockfiles in the submission.

**Environment variables:** [CONFIRM WHETHER NONE ARE REQUIRED FOR THE DEFAULT LOCAL SETUP]. The intended default backend address is `http://localhost:3000`. The frontend API helper supports `VITE_API_URL`; if overriding the address, follow the base-URL format used by `frontend/src/api.js`.

## Installation and startup

Extract the project ZIP to a local folder. The commands below assume the root contains folders named `backend` and `frontend`; adjust these instructions before submission if the actual names differ.

### Start the backend

Open a terminal in the extracted project root:

```sh
cd backend
npm install
npm start
```

The backend should listen at:

```text
http://localhost:3000
```

Keep this terminal open while using the application.

### Start the frontend

Open a second terminal in the extracted project root:

```sh
cd frontend
npm install
npm run dev
```

Open the local URL printed by Vite.

**Frontend URL used in the final clean-copy test:** [INSERT THE ACTUAL URL].

The frontend connects to the backend and loads the saved Submitted Run. Both processes must remain running; use `Ctrl+C` in each terminal to stop them.

**The marker must not need to run `npm run seed`.** The submitted SQLite database must already contain the completed Submitted Run and its history.

### If startup fails

- **Missing `products.json`:** Confirm the file is in `backend/`, alongside `package.json`, rather than in `src/`. It supplies the catalogue when creating a run.
- **Frontend cannot reach the API:** Confirm the backend is running on the configured address and inspect the displayed error. Check `frontend/src/api.js` and any local API URL override.
- **Port already in use:** Stop the other process using the port, or make a documented port change and update the frontend connection accordingly.
- **SQLite dependency installation fails:** Confirm that Node.js matches the tested version above and inspect the full `npm install` error.
- **Submitted Run is missing:** Check that the supplied `backend/supermarket.db` was included and is the correct file. Do not delete it or regenerate evidence as a routine startup step.

## Database and simulation runs

### Submitted Run

The primary database is:

```text
backend/supermarket.db
```

**Submitted Run ID:** [ACTUAL RUN ID]  
**Completion status:** [CONFIRM COMPLETED, WITH 60/60 DAYS RETAINED]  
**Assurance result:** [ACTUAL RESULT; IDENTIFY ANY UNRESOLVED EXCEPTIONS]

The intended startup workflow automatically selects the Submitted Run. It is also identifiable in **Setup & Runs**.

All four report parts, Daily Reports, Model Declaration and Assurance must refer to this same run when inspecting the submitted evidence. The database is the primary store; JSON is only catalogue input and is not a substitute for historical records.

### Create an additional run

1. Open **Setup & Runs**.
2. Select **Start new 60-day run**.
3. Wait for completion, then select the resulting Run ID.
4. Inspect the new run's report and Assurance results.
5. Select the Submitted Run again to return to the assessed evidence.

A new run must receive a separate identity and preserve existing run history. Starting another run is not intended to replace the Submitted Run automatically.

Model inputs are maintained in `backend/products.json` and `backend/src/config.js`. Changes must apply to a new run rather than alter a run that has already begun. The website's reports should use the selected run's frozen settings, not the latest files.

### Developer-only seed command

For initial preparation in a separate development copy with a fresh database:

```sh
cd backend
npm run seed
```

The intended seed script creates a 60-day run, checks its Assurance results and designates it as the Submitted Run only if those checks pass. Confirm the script's actual output and behaviour before packaging the project.

This command prepares submission evidence; it is not a required marker installation step. Back up existing databases before maintenance and do not delete or overwrite the final submitted database to rerun a seed command.

## Architecture and data flow

```text
products.json + config.js
           |
           v
Create a run and save its configuration snapshot
           |
           v
demand.js -> engine.js <-> storeBrain.js
           |
           v
SQLite: customers, transactions, inventory, orders,
        expiry, cash, daily results and decisions
           |
           v
Report and Assurance services -> Express API -> React website
```

Configuration describes the model. Historical records describe what occurred, while report services derive figures from the selected run's stored history.

| File or folder | Responsibility |
|---|---|
| `backend/src/server.js` | Start Express and attach API routes |
| `backend/src/db.js` | Open SQLite and initialise database protections |
| `backend/src/schema.sql` | Define tables and relationships |
| `backend/src/config.js` | Define model parameters and fixed constraints |
| `backend/src/rng.js` | Supply seeded pseudo-random generation |
| `backend/src/routes/runs.js` | Expose run, report and evidence endpoints |
| `backend/src/services/runs.js` | Validate inputs and create run snapshots |
| `backend/src/simulation/demand.js` | Generate customer activity and requested baskets |
| `backend/src/simulation/engine.js` | Execute the daily cycle and save operating history |
| `backend/src/simulation/storeBrain.js` | Evaluate conditions and decide replenishment |
| `backend/src/services/reports.js` | Query summary, daily and product evidence |
| `backend/src/services/assurance.js` | Reconcile stored inventory, revenue and cash |
| `backend/src/services/declaration.js` | Describe the saved run's model |
| `backend/scripts/seed-submitted-run.js` | Prepare the Submitted Run in development |
| `frontend/src/api.js` | Make backend API requests |
| `frontend/src/components/` | Render setup, declaration, reports, daily evidence and Assurance |
| `frontend/src/components/findings.js` | Build structured findings from report data |

Completed-run configuration and historical records are intended to be protected against editing or deletion through normal application workflows. A modelling or implementation change requires a new run, not revision of old evidence.

## Customer and demand model

The proposed model distinguishes commuter, office-lunch, top-up and household shopping missions. Mission mix, basket behaviour, category preferences, time-of-day patterns and weekday/weekend differences create structured demand, with product popularity weights providing further variation.

Only catalogue products can be purchased. Actual sales are limited to available unexpired stock, and each completed purchase records its items and revenue and updates inventory and cash.

A stored random seed supports repeatable testing with unchanged inputs and code. It is separate from the `npm run seed` database-preparation command, and exact deterministic replay is not an assignment requirement.

**Final implementation check:** [CONFIRM THAT THE MISSION MODEL ABOVE MATCHES THIS SUBMITTED RUN]. Exact assumptions, including customer generation and unavailable-item behaviour, are documented in the website's Model Declaration.

## Inventory and Store Brain

Opening inventory must cost no more than A$40,000. Operating cash starts separately at A$15,000; neither the opening inventory cost nor unused inventory budget changes that opening cash.

The Store Brain uses rules to identify low stock, sold-out products, expiry risk, expired stock and slow-moving inventory. Replenishment considers recent sales, available stock, perishability and cash, with decisions and reasons retained for inspection.

The intended daily cycle is:

1. Receive orders placed at the previous day's close.
2. Remove stock that has reached expiry and record write-offs.
3. Process customers and valid sales.
4. Review closing conditions.
5. On Days 1–59, place justified affordable orders and deduct cash immediately.
6. Save the day's results.

Lead time is one day; newly ordered stock is unavailable until the following morning. No order is placed at the end of Day 60 for Day 61.

Perishable stock is tracked by batch so receipts from different days can expire at different times. The proposed engine allocates sales to the oldest unexpired batches first.

Under cash constraints, the proposed brain prioritises urgent needs and reduces or skips unaffordable orders. The exact thresholds, priorities and first-days demand fallback must be stated in the run's Model Declaration and match the code.

**Pricing policy:** [CONFIRM FIXED PRICES, OR DESCRIBE ACTUAL MARKDOWNS AND THEIR RECORDING].

## Where to find the required evidence

The screen names below follow the planned frontend; confirm the labels in the final build.

| Requirement | Application location |
|---|---|
| Submitted Run and additional runs | **Setup & Runs** |
| Opening resources and sample products | **Setup & Runs** and/or **Model Declaration** |
| Customer, inventory and Store Brain assumptions | **Model Declaration** |
| Part 1: Business Performance | **60-Day Report**, business totals, shoppers, categories and products |
| Part 2: Inventory and Autonomous Operations | **60-Day Report**, inventory health, orders and Store Brain responses |
| Part 3: Commercial Diagnosis and Management Recommendations | **60-Day Report**, three written findings and recommendations |
| Part 4: Commercial Assurance | **Commercial Assurance** |
| Any selected day from 1 to 60 | **Daily Reports**, using the day selector |
| Recorded baskets for a day | **Daily Reports**, transaction evidence control |
| A product's inventory, orders and events | Product trace opened from the supported product links |

The three findings must each include Observation → Evidence → Diagnosis → Business implication → Recommendation. At least one addresses a problem, anomaly, trade-off or limitation; template wording must be reviewed against the actual run.

For a traceability demonstration, start at a reported product or operational issue, open the relevant Daily Report, inspect the Store Brain event and follow its transaction, inventory or order evidence. Use the same Run ID throughout.

## Commercial Assurance

The Assurance view is intended to calculate checks from stored records rather than merely display a hardcoded PASS label.

- **History:** Identify the run, retain all 60 days and show relevant evidence counts.
- **Inventory:** Opening + received − sold − expired/written off = closing, with checks for negative stock, overselling and expired sales.
- **Revenue:** Transaction totals reconcile to each day's revenue, and daily revenue reconciles to the 60-day total.
- **Cash:** Opening cash + revenue − replenishment spending = closing cash, with orders checked against cash available when placed.
- **Exceptions:** Display actual PASS/FAIL results and disclose material discrepancies.

Stockouts, expiry, slow-moving stock and low-cash warnings may be valid simulated business outcomes. They are different from integrity failures such as negative stock or unexplained revenue differences.

## AI collaboration

Perplexity was used to assist with assignment interpretation, backend modularisation, simulation and database design, frontend/API alignment, drafting product data, debugging guidance, and documentation and presentation drafts.

A concrete review issue arose during integration: the AI-assisted replacement backend expected `backend/products.json`, but the file was missing. Running the application produced an `ENOENT` error. I reported the runtime error and requested a catalogue rather than treating the generated backend as complete. This led to a separate catalogue draft and clarified the distinction between including the SQLite Submitted Run and supplying JSON input for new runs.

**Final review outcome:** [STATE WHAT YOU ACTUALLY CHANGED OR CHECKED AFTER THIS ERROR, AND THE ACTUAL RESULT. DO NOT CLAIM A SUCCESSFUL SEED OR RUN WITHOUT TESTING IT.]

The AI-generated catalogue requires review of its pack sizes, costs, prices, shelf lives and opening quantities. Generated findings also require review against stored evidence; AI-written diagnoses are not proof of causation.

I remain responsible for understanding the submitted code, reviewing and validating AI-assisted work, selecting model assumptions and ensuring all report claims match the Submitted Run. Fictional video examples are practice material and must not be represented as simulation evidence.

## Known limitations

- **Uncalibrated demand:** Customer volumes and shopping preferences are modelling assumptions, not a validated forecast based on real store data.
- **Simplified supply:** A fixed one-day lead time does not model supplier delays or delivery uncertainty.
- **Simplified product assumptions:** Catalogue prices and shelf lives are educational inputs, not verified live market or food-safety data.
- **Excluded overhead:** Rent, utilities, insurance and other excluded costs are not modelled; gross profit is not full net profit.
- **Finite horizon:** Results cover 60 simulated days and do not establish long-term business viability.
- **Template analysis:** Structured findings can help explain evidence but need human review for relevance and justified conclusions.
- **Local prototype:** The design is for a locally runnable educational system, not a production retail deployment.
- **Outstanding technical issues:** [LIST ACTUAL UNRESOLVED ISSUES, OR STATE NONE IDENTIFIED AFTER THE DOCUMENTED TESTS].

Passing Assurance supports internal consistency within the checks performed. It does not demonstrate real-world predictive accuracy or prove that a recommendation is optimal.

## Clean-copy verification and packaging

Before submitting, extract a fresh copy of the final ZIP and perform these checks:

- [ ] Run the exact installation/startup commands in this README.
- [ ] Confirm the Submitted Run loads without seeding.
- [ ] Confirm its Run ID and 60/60 retained days.
- [ ] Inspect the Model Declaration and all four acceptance areas.
- [ ] Open a Daily Report, transactions and product/Store Brain evidence.
- [ ] Check actual inventory, revenue and cash Assurance results.
- [ ] Create an additional run and confirm the original history remains intact.
- [ ] Restart the application and confirm saved evidence persists.
- [ ] Confirm that no normal workflow edits or deletes completed history.
- [ ] Replace all README placeholders and remove preparation-only notes.

Include the source, dependency manifests/lockfiles, README, catalogue and SQLite database containing the Submitted Run. Exclude `node_modules`, build output, temporary files and secrets.

Close processes using SQLite before packaging. If `supermarket.db-wal` or `supermarket.db-shm` remains, verify that the database has been safely checkpointed and the main `.db` contains the complete run before omitting these files; do not simply delete a live WAL file.

Submit the source-code ZIP and the required 4–8 minute MP4 directly to the LMS. The website contains the required reports; this README and the video do not replace missing application evidence.
