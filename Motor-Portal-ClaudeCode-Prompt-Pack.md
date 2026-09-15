# Motor Portal — Claude Code Autonomous Build Prompt Pack

This is a ready-to-run sequence of Claude Code prompts that builds the full
Motor Insurance Bulk Policy Portal end-to-end across **four separate
repositories**, in order, with **automatic git commits, pushes, and README
updates baked into every phase** — you should not need to prompt for those
separately.

Reference source: your Motor Portal Application walkthrough (PPTX) and the
Motor Portal ER Diagram & Data Dictionary (DOCX) — a bulk Excel-to-policy
insurance flow, max 200 cases/batch, 3 products, a 6-stage user-facing
pipeline sitting on a 9-state backend lifecycle, and 15 core entities.

---

## 0. How to use this pack

1. **Create an empty parent folder** and four subfolders inside it:
   ```
   MotorPortal/
     MotorPortalAPI/
     MotorPortalWEB/
     MotorPortalDB/
     MotorPortalDOC/
   ```
2. Make sure you have: .NET 8 SDK, Node.js + Angular CLI, PostgreSQL 15+,
   Git, and the [GitHub CLI](https://cli.github.com/) (`gh auth login` done
   once).
3. Open a terminal at `MotorPortal/` and start Claude Code there
   (`claude`). Run it in an auto-accepting mode so it doesn't pause for
   file/command approval on every step, e.g. `claude --dangerously-skip-permissions`
   (only do this in a disposable/dev environment).
4. Paste **Prompt 0** first. Let it finish completely before pasting
   **Prompt 1**, then **Prompt 2**, and so on, in order. Each prompt is
   self-contained and tells Claude Code to keep working, fix its own
   errors, and commit/push before stopping — you don't need to say
   "continue" unless it genuinely gets stuck.
5. If a prompt's session ends or you reopen the folder later, Claude Code
   will re-read the `CLAUDE.md` ground-rules file created in Prompt 0, so
   context isn't lost.

Every prompt below ends the same way on purpose — that repetition is what
keeps the agent autonomous and keeps git/README hygiene consistent across
an 11-step build.

---

## GROUND RULES (baked into every prompt — for your reference only)

- Build the **real, working system** — no stubs, no "TODO: implement
  later", no mock UI screens pretending to be the backend.
- Keep going until the phase compiles, runs, and does what it says. Fix
  compilation/runtime errors yourself without asking permission.
- **Never leave a phase uncommitted.** Stage, commit with a Conventional
  Commit message (`feat:`, `fix:`, `docs:`, `chore:`), and push to
  `origin main` before ending the turn. Commit in small logical chunks
  as you go, not one giant commit at the end.
- **Update `README.md`** in the repo you're working in after every phase:
  what was added, how to run it, and tick off the progress checklist.
- Don't rename the 15 entity names, the product/process lists, the batch
  status values, or the folder structure given below — the frontend,
  backend and DB prompts all assume these exact names.

---

## PROMPT 0 — Workspace, GitHub repos, and ground rules

```
You are a senior full-stack architect bootstrapping a 4-repository system
called "Motor Portal". Work from the current directory, which already
contains four empty folders: MotorPortalAPI, MotorPortalWEB, MotorPortalDB,
MotorPortalDOC.

GOAL FOR THIS PHASE ONLY — repo + workspace bootstrap. Do not write any
application code yet.

1. For EACH of the four folders, do the following:
   a. Run `git init` inside it.
   b. Create a `CLAUDE.md` file at its root containing these ground rules
      verbatim (adapt the "This repo" line to name that specific repo):
        - This repo is part of the 4-repo Motor Portal system
          (MotorPortalAPI, MotorPortalWEB, MotorPortalDB, MotorPortalDOC).
        - Build the real, working system — no stubs, no placeholder logic,
          no "TODO: implement later".
        - Keep going until it compiles and runs. Fix your own errors
          without asking for permission to proceed.
        - Never end a turn with uncommitted changes. Stage, commit with a
          Conventional Commit message, and push to origin main before
          stopping. Commit in small logical chunks, not one mega-commit.
        - Update README.md after every phase of work: what changed, how to
          run/test it, and the progress checklist.
        - Do not rename the 15 core entities, the product/process lists,
          the batch status lifecycle values, or the agreed folder
          structure — other repos and prompts assume these exact names.
   c. Create an initial `README.md` with a title, one-paragraph purpose
      statement, and a "Progress" checklist section (all unchecked) listing
      the phases relevant to that repo (you'll tick these off in later
      prompts).
   d. Create a `.gitignore` appropriate to that repo's stack (Node/Angular
      for WEB, .NET for API, SQL/general for DB, general for DOC).
   e. Commit as `chore: bootstrap repo with ground rules and README`.

2. Using the GitHub CLI (`gh`), create four **separate, same-named** GitHub
   repositories — MotorPortalAPI, MotorPortalWEB, MotorPortalDB,
   MotorPortalDOC — under my authenticated account, private by default.
   Wire each local folder to its matching remote as `origin` and push
   `main`. If `gh` isn't authenticated, stop and tell me exactly what to
   run — don't fabricate credentials.

3. At the parent level (outside the four repos, not its own git repo),
   create a short `WORKSPACE.md` summarizing the 4-repo layout and the
   order the build phases run in, for your own future reference.

4. Confirm at the end with a short status table: repo name → local path →
   remote URL → last commit hash.

Do not proceed to any backend/frontend/database implementation in this
prompt. When done, stop and wait for the next phase.
```

---

## PROMPT 1 — PostgreSQL database: schema, tables, constraints, procedures, seed data

```
Work in MotorPortalDB. Re-read CLAUDE.md first and follow it for the rest
of this session.

GOAL — build the complete PostgreSQL data layer for the Motor Portal
system, as real runnable SQL (not an ORM migration — this repo is
SQL-first; MotorPortalAPI will connect to what you build here via EF Core
"database-first" style mapping in a later phase).

Database name: SGInsuranceDB
Schema name:   SGInsurance
Engine: PostgreSQL 15+, PostgreSQL-native types only (no SQL Server / no
Oracle syntax).

1. FOLDER STRUCTURE
   MotorPortalDB/
     scripts/
       00_create_database.sql
       01_create_schema.sql
       02_tables/            (one .sql file per table, 15 files)
       03_constraints_indexes.sql
       04_functions/         (one .sql file per function/procedure)
       05_views/
       06_seed_data.sql
     migrate.sh               (runs 00→06 in order against a local Postgres)
     rollback.sql
     README.md

2. CREATE THESE 15 TABLES IN THE SGInsurance SCHEMA — do not rename them:
   USER_MASTER, PRODUCT_MASTER, FUNCTION_MASTER, MASTER_POLICY,
   BATCH_MASTER, BATCH_DETAIL, INVALID_RECORDS, PREMIUM_DETAILS,
   GST_DETAILS, PROPOSAL_MASTER, PAYMENT_DETAILS, POLICY_MASTER,
   POLICY_CERTIFICATE, REPORT_LOG, AUDIT_LOG.

   Use exactly these columns and types (translate to PostgreSQL-native
   equivalents, e.g. BIGINT/INTEGER/VARCHAR/NUMERIC/DATE/TIMESTAMP/CHAR
   as given, GENERATED ALWAYS AS IDENTITY for PKs):

   USER_MASTER: USER_ID PK, USERNAME VARCHAR(50) UNIQUE, PASSWORD_HASH
   VARCHAR(255), ROLE VARCHAR(30), STATUS CHAR(1), CREATED_ON TIMESTAMP.

   PRODUCT_MASTER: PRODUCT_ID PK, PRODUCT_CODE VARCHAR(20) UNIQUE,
   PRODUCT_NAME VARCHAR(60), STATUS CHAR(1). Seed: CLASS_E / Motor Class-E,
   CLASS_F / Motor Class-F, EICHER / Eicher Motor.

   FUNCTION_MASTER: FUNCTION_ID PK, FUNCTION_CODE VARCHAR(20) UNIQUE,
   FUNCTION_NAME VARCHAR(60). Seed: EXCEL_UPLOAD / Motor Excel Upload,
   BATCH_SUMMARY / Motor Batch Summary, REPORT / Motor Report,
   SEARCH_PRINT / Search & Print Policy, POLICY_CANCEL / Policy Cancel
   Upload.

   MASTER_POLICY: MASTER_POLICY_ID PK, PRODUCT_ID FK→PRODUCT_MASTER,
   MASTER_POLICY_NO VARCHAR(40) UNIQUE, CUSTOMER_NO VARCHAR(30), CDBG_NO
   VARCHAR(20), CD_BALANCE NUMERIC(15,2).

   BATCH_MASTER: BATCH_ID PK, USER_ID FK→USER_MASTER, PRODUCT_ID
   FK→PRODUCT_MASTER, FUNCTION_ID FK→FUNCTION_MASTER, FILE_NAME
   VARCHAR(255), TOTAL_RECORDS INTEGER, VALID_RECORDS INTEGER,
   INVALID_RECORDS INTEGER, STATUS VARCHAR(25), CREATED_ON TIMESTAMP.
   Add a CHECK constraint TOTAL_RECORDS <= 200.

   BATCH_DETAIL: DETAIL_ID PK, BATCH_ID FK→BATCH_MASTER, MASTER_POLICY_NO
   VARCHAR(40), ENGINE_NO VARCHAR(30), CHASSIS_NO VARCHAR(30), TC_NO
   VARCHAR(30), INVOICE_NO VARCHAR(30), TRANSIT_DATE DATE, MAKE
   VARCHAR(40), MODEL VARCHAR(80), RECORD_STATUS VARCHAR(20).

   INVALID_RECORDS: INVALID_ID PK, BATCH_ID FK→BATCH_MASTER, DETAIL_ID
   FK→BATCH_DETAIL, TRANSIT_DATE DATE, INVOICE_NO VARCHAR(30), ENGINE_NO
   VARCHAR(30), CHASSIS_NO VARCHAR(30), ERROR_REMARKS VARCHAR(255).

   PREMIUM_DETAILS: PREMIUM_ID PK, DETAIL_ID FK→BATCH_DETAIL, BASE_PREMIUM
   NUMERIC(12,2), ADDON_PREMIUM NUMERIC(12,2), DISCOUNT NUMERIC(12,2),
   NET_PREMIUM NUMERIC(12,2).

   GST_DETAILS: GST_ID PK, PREMIUM_ID FK→PREMIUM_DETAILS UNIQUE (1:1),
   GST_RATE NUMERIC(5,2), GST_AMOUNT NUMERIC(12,2), FINAL_PREMIUM
   NUMERIC(12,2).

   PROPOSAL_MASTER: PROPOSAL_ID PK, DETAIL_ID FK→BATCH_DETAIL,
   PROPOSAL_NO VARCHAR(30) UNIQUE, PREMIUM_AMOUNT NUMERIC(12,2), STATUS
   VARCHAR(25), REMARKS VARCHAR(120).

   PAYMENT_DETAILS: PAYMENT_ID PK, PROPOSAL_ID FK→PROPOSAL_MASTER,
   MASTER_POLICY_ID FK→MASTER_POLICY, PF_REF_NO VARCHAR(40), AMOUNT
   NUMERIC(12,2), PAYMENT_STATUS VARCHAR(20), TAGGED_ON TIMESTAMP.

   POLICY_MASTER: POLICY_ID PK, PROPOSAL_ID FK→PROPOSAL_MASTER, PAYMENT_ID
   FK→PAYMENT_DETAILS, POLICY_NO VARCHAR(40) UNIQUE, MAKE VARCHAR(40),
   MODEL VARCHAR(80), ENGINE_NO VARCHAR(30), CHASSIS_NO VARCHAR(30),
   PREMIUM NUMERIC(12,2), STATUS VARCHAR(20), ISSUED_ON TIMESTAMP.

   POLICY_CERTIFICATE: CERT_ID PK, POLICY_ID FK→POLICY_MASTER, CERT_PATH
   VARCHAR(255), GENERATED_ON TIMESTAMP.

   REPORT_LOG: REPORT_ID PK, USER_ID FK→USER_MASTER, REPORT_TYPE
   VARCHAR(40), FROM_DATE DATE, TO_DATE DATE, GENERATED_ON TIMESTAMP.

   AUDIT_LOG: AUDIT_ID PK, USER_ID FK→USER_MASTER, ENTITY_NAME VARCHAR(40),
   ACTION VARCHAR(20), REF_ID VARCHAR(40), TIMESTAMP TIMESTAMP.

3. RELATIONSHIPS & INTEGRITY RULES (implement as FKs with sensible ON
   DELETE behavior):
   - USER_MASTER 1→N BATCH_MASTER, 1→N REPORT_LOG, 1→N AUDIT_LOG.
   - PRODUCT_MASTER 1→N BATCH_MASTER, 1→N MASTER_POLICY.
   - FUNCTION_MASTER 1→N BATCH_MASTER.
   - BATCH_MASTER 1→N BATCH_DETAIL, 1→N INVALID_RECORDS — ON DELETE CASCADE
     (deleting a batch cascades to its details and invalid records).
   - BATCH_DETAIL 1→0..1 INVALID_RECORDS, 1→0..1 PREMIUM_DETAILS, 1→0..1
     PROPOSAL_MASTER.
   - PREMIUM_DETAILS 1→1 GST_DETAILS.
   - PROPOSAL_MASTER 1→0..1 PAYMENT_DETAILS, 1→0..1 POLICY_MASTER.
   - MASTER_POLICY 1→N PAYMENT_DETAILS.
   - PAYMENT_DETAILS 1→0..1 POLICY_MASTER.
   - POLICY_MASTER 1→N POLICY_CERTIFICATE.
   Add indexes on every FK column and on BATCH_DETAIL(ENGINE_NO),
   BATCH_DETAIL(CHASSIS_NO), POLICY_MASTER(POLICY_NO) for search.

4. FUNCTIONS / STORED PROCEDURES (PL/pgSQL) — one file each under
   04_functions/:
   - fn_calculate_net_premium(base, addon, discount) → NET_PREMIUM =
     base + addon - discount.
   - fn_calculate_gst(net_premium, gst_rate) → returns (gst_amount,
     final_premium) where GST_AMOUNT = net_premium * gst_rate/100 and
     FINAL_PREMIUM = net_premium + gst_amount.
   - fn_generate_proposal_no() → unique proposal number in the pattern
     1202304500/01 style (10-digit sequence / 2-digit suffix).
   - fn_generate_policy_no() → unique policy number in the pattern
     3010/A/443668109/00/000.
   - sp_process_batch_validation(batch_id) → walks BATCH_DETAIL rows for
     a batch, applies validation rules (master policy exists, product
     mapping valid, engine/chassis duplication check against
     POLICY_MASTER + BATCH_DETAIL, existing policy check, duplicate
     transaction in batch, mandatory fields, datatype correctness),
     writes rejects into INVALID_RECORDS with ERROR_REMARKS, sets
     RECORD_STATUS on BATCH_DETAIL to VALID/INVALID, and updates
     BATCH_MASTER VALID_RECORDS/INVALID_RECORDS/STATUS.
   - sp_tag_payment(proposal_id) → checks MASTER_POLICY.CD_BALANCE is
     sufficient for PROPOSAL_MASTER.PREMIUM_AMOUNT inside a transaction;
     if insufficient, raises a meaningful exception and does not touch
     balances; if sufficient, deducts the amount, inserts PAYMENT_DETAILS
     with a generated PF_REF_NO and PAYMENT_STATUS = PROCESSED.
   - sp_advance_batch_status(batch_id, new_status) → validates the
     transition follows the lifecycle order below and updates
     BATCH_MASTER.STATUS, else raises an error.
   Batch lifecycle (enforce this exact order, no skipping):
   UPLOADED → VALIDATED → PREMIUM_CALCULATED → GST_CALCULATED →
   PROPOSAL_CREATED → PAYMENT_PENDING → PAYMENT_PROCESSED →
   POLICY_CREATED → PRINTED (with a separate INVALID branch per-record,
   not a batch-level state).

5. VIEWS under 05_views/: a reporting view `vw_policy_issue_report`
   joining BATCH_MASTER, POLICY_MASTER, PROPOSAL_MASTER, MASTER_POLICY,
   PRODUCT_MASTER, PAYMENT_DETAILS, USER_MASTER — used later by the
   Motor Report / Excel export feature (columns: Batch ID, Policy Number,
   Proposal Number, Master Policy, Product, Make, Model, Engine Number,
   Chassis Number, Premium, Payment Status, Issued Date, User).

6. SEED / DUMMY DATA (06_seed_data.sql):
   - One admin user: USERNAME 'admin', PASSWORD_HASH = a bcrypt hash of
     'admin123' (generate a real bcrypt hash, don't fake the format),
     ROLE 'ADMIN', STATUS 'A'.
   - The PRODUCT_MASTER and FUNCTION_MASTER seed rows from step 2.
   - At least 10 MASTER_POLICY rows spread across the 3 products, with
     realistic MASTER_POLICY_NO values (pattern like DL-3010/A/1485551),
     CUSTOMER_NO, CDBG_NO, and CD_BALANCE values (some high, at least one
     deliberately low so the insufficient-balance path can be demoed).
   - 2-3 sample BATCH_MASTER rows in different lifecycle states with
     matching BATCH_DETAIL rows (mix of VALID/INVALID RECORD_STATUS,
     realistic ENGINE_NO/CHASSIS_NO/MAKE/MODEL), so the API/frontend
     teams have real data to build against immediately.

7. Write `migrate.sh` to run files 00→06 in order against a local
   Postgres instance using environment variables for connection details
   (PGHOST/PGPORT/PGUSER/PGPASSWORD/PGDATABASE), and `rollback.sql` to
   drop the schema cleanly for a fresh re-run.

8. ACTUALLY RUN migrate.sh against a local/test Postgres instance
   yourself if one is reachable, fix any SQL errors until it runs clean
   end to end, and paste the final table count / row counts into your
   own verification — don't just assume the SQL is correct.

9. Update README.md with: how to run migrate.sh, the schema diagram in
   words (or a Mermaid ER diagram if you can render one in Markdown), and
   tick off the progress checklist for this phase.

10. Commit your work in logical chunks (e.g. "feat: add 15 core tables",
    "feat: add batch lifecycle functions and procedures", "feat: add seed
    data") and push to origin main. Do not end this turn with anything
    uncommitted.
```

---

## PROMPT 2 — Backend core: .NET 8 Web API scaffolding, EF Core, auth

```
Work in MotorPortalAPI. Re-read CLAUDE.md first.

GOAL — stand up the ASP.NET Core Web API skeleton wired to the
SGInsuranceDB database already built in MotorPortalDB (assume it's
reachable at a local Postgres connection string in appsettings —
parameterize it via environment/user-secrets, don't hardcode a password).

1. Solution/project layout:
   MotorPortal.sln
   MotorPortal.API/            (Controllers/, Middleware/, Extensions/,
                                 Program.cs, appsettings.json,
                                 appsettings.Development.json)
   MotorPortal.Application/    (DTOs/, Interfaces/, Services/, Validators/)
   MotorPortal.Domain/         (Entities/, Enums/, Constants/)
   MotorPortal.Infrastructure/ (Data/, Repositories/, Services/,
                                 Migrations/)

2. Add EF Core with Npgsql provider. Model all 15 entities from the DB
   schema (same names/columns as MotorPortalDB) as EF Core entities in
   MotorPortal.Domain, map them in an `AppDbContext` in
   MotorPortal.Infrastructure using Fluent API (table/schema names must
   match SGInsurance.* exactly — use `HasDefaultSchema("SGInsurance")` on
   the context). Since the schema already exists, configure this as
   database-first: scaffold or hand-write entities/config so
   `dotnet ef database update` is not required to create tables, but keep
   migrations available for any future schema changes made from the API
   side.

3. JWT authentication:
   - AuthController with POST /api/auth/login (validates USERNAME +
     PASSWORD_HASH via bcrypt against USER_MASTER, issues a JWT) and
     POST /api/auth/logout.
   - AuthService, JWT settings in appsettings (issuer/audience/key via
     user-secrets or env var, not committed in plaintext).
   - [Authorize] on all controllers except AuthController and Swagger.
   - Global exception-handling middleware returning a consistent JSON
     error shape (statusCode, message, traceId).
   - Structured logging (Serilog or built-in ILogger with a JSON console
     sink) including a request-id per call.

4. Configure Swagger/OpenAPI with JWT bearer auth support in the UI, CORS
   allowing the Angular dev origin (http://localhost:4200), and a
   GET /api/health endpoint that checks DB connectivity.

5. Repository/service layering: a generic repository plus
   entity-specific repositories where it earns its keep (don't
   over-abstract simple CRUD). DTOs for every entity that crosses the
   API boundary — never return EF entities directly from controllers.

6. Build and run the API yourself against the real database, hit
   /api/health and /api/auth/login (with the seeded admin/admin123) to
   confirm the whole chain works end to end. Fix any errors until it
   does.

7. Update README.md: how to configure the connection string/JWT secret,
   how to run (`dotnet run`), how to open Swagger, and tick off this
   phase's checklist items.

8. Commit in logical chunks ("feat: EF Core entities and AppDbContext",
   "feat: JWT auth", "feat: swagger, cors, health check", "chore: logging
   and global exception handling") and push to origin main. Nothing
   uncommitted at the end of this turn.
```

---

## PROMPT 3 — Backend business logic: Excel upload, validation, pricing, proposal, payment, policy, certificate

```
Work in MotorPortalAPI. Re-read CLAUDE.md first.

GOAL — implement the full bulk-processing pipeline as real, callable API
endpoints, mirroring the batch lifecycle already enforced in
MotorPortalDB's stored procedures. Prefer calling those procedures/
functions from EF Core (via FromSqlRaw / stored-procedure calls) over
re-implementing the same logic twice, but wrap them in proper C# services
so the API has clean seams and testable logic on the C# side too.

1. Excel Upload
   - POST /api/batches/upload (multipart form: ProductId, FunctionId,
     file). Parse the workbook server-side (ClosedXML or EPPlus).
   - Validate: file extension, required columns in order
     (MASTER_POLICY_NO, ENGINE_NO, CHASSIS_NO, TC_NO, INVOICE_NO,
     TRANSIT_DATE, MAKE, MODEL), datatypes, no blank rows, max 200
     records, required fields present. Return a clear structured error
     list if invalid — do not partially insert an invalid file.
   - On success: create BATCH_MASTER (STATUS = UPLOADED), bulk-insert
     BATCH_DETAIL rows, return { batchId, totalRecords }.
   - GET /api/batches/{id}/sample-template — generates and returns a
     downloadable sample .xlsx matching the required columns.

2. Batch Processing orchestration
   - POST /api/batches/{id}/process — runs, in order: validation (call
     sp_process_batch_validation), premium calculation (fn_calculate_net_
     premium per valid BATCH_DETAIL using configurable per-product
     premium rules — put these rules in a small config table or appsettings
     section, not hardcoded in controllers), GST calculation
     (fn_calculate_gst using a configurable GST rate), proposal creation
     (fn_generate_proposal_no + insert PROPOSAL_MASTER, status
     PROPOSAL_CREATED, remarks "Successful Proposal Created"). Advance
     BATCH_MASTER.STATUS after each stage via sp_advance_batch_status.
     Invalid records must not block valid ones — isolate and continue.
   - GET /api/batches/{id}/status — current BATCH_MASTER row plus
     per-stage counts, for a UI to poll and show live progress.

3. Invalid Records
   - GET /api/batches/{id}/invalid-records — list with TRANSIT_DATE,
     INVOICE_NO, ENGINE_NO, CHASSIS_NO, ERROR_REMARKS.
   - DELETE /api/batches/{id}/invalid-records — removes all invalid
     entries for the batch, updates BATCH_MASTER counters, leaves valid
     BATCH_DETAIL rows untouched and eligible to continue processing.

4. Payment Tagging
   - IPaymentService / PaymentService / MockPfService (a real class
     simulating an external PF service call — async, with a small
     artificial delay — architected as if swapping in a real HTTP client
     later is a one-line change).
   - POST /api/batches/{id}/payments — for every PROPOSAL_CREATED case in
     the batch, call sp_tag_payment inside a DB transaction; check
     MASTER_POLICY.CD_BALANCE first and fail that case with a meaningful
     message (not a stack trace) if insufficient, without affecting other
     cases. Advance BATCH_MASTER.STATUS to PAYMENT_PROCESSED once all
     eligible cases are tagged (partial success is reported, not
     silently swallowed).

5. Policy Generation
   - Policy is only created when proposal exists AND payment exists AND
     PAYMENT_STATUS = PROCESSED. Generate POLICY_NO via
     fn_generate_policy_no, insert POLICY_MASTER, advance
     BATCH_MASTER.STATUS to POLICY_CREATED.

6. Policy Certificate + Bulk Print
   - POST /api/policies/{id}/certificate — generates a real PDF
     certificate (QuestPDF or similar) with Policy Number, Master Policy,
     Make, Model, Engine Number, Chassis Number, Premium, Issued Date;
     stores the file under a served static path and records CERT_PATH +
     GENERATED_ON in POLICY_CERTIFICATE.
   - GET /api/policies/{id}/certificate — streams the PDF for
     print/download.
   - POST /api/batches/{id}/bulk-print — generates certificates for every
     policy in the batch, sets BATCH_MASTER.STATUS = PRINTED, writes an
     AUDIT_LOG row with ACTION = PRINT.

7. Wire up global exception handling so every failure path above (bad
   Excel, insufficient CD balance, invalid state transition) returns a
   clear, specific error message and HTTP status — never a bare 500.

8. Test the full pipeline yourself end-to-end against real seeded data
   (upload a small in-memory-generated sample workbook if you don't have
   a physical file, run it through every stage via HTTP calls) and fix
   anything that breaks.

9. Update README.md with the new endpoints (a table: method, path,
   purpose) and tick off this phase's checklist.

10. Commit in logical chunks per feature area (excel upload, batch
    processing/validation, invalid records, payment, policy + certificate)
    and push to origin main. Nothing uncommitted at the end.
```

---

## PROMPT 4 — Backend remaining modules: batch summary, reports, search, policy cancel, CD balance, audit

```
Work in MotorPortalAPI. Re-read CLAUDE.md first.

GOAL — complete the remaining backend surface area needed by the
frontend pages.

1. Batch Summary
   - GET /api/batches?fromDate=&toDate= — list with Batch ID, Total
     Records, Valid Records, Invalid Records, Pending Processing, Payment
     Pending, Payment Processed, Status, Created On, filtered by
     transaction date range.
   - GET /api/batches/summary-counters?fromDate=&toDate= — aggregate
     counters (Total/Valid/Invalid/Pending Batch Processing/Payment
     Pending/Payment Processed) for the dashboard cards.

2. CD Balance
   - GET /api/master-policies — for the dropdown (MasterPolicyNo,
     CustomerNo, CdbgNo).
   - GET /api/master-policies/{id}/cd-balance — returns current
     CD_BALANCE for the "View CD Balance" drawer.

3. Reports
   - POST /api/reports/policy-issue — given a date range, query
     vw_policy_issue_report, write a REPORT_LOG row, and return a
     downloadable .xlsx (Batch ID, Policy Number, Proposal Number, Master
     Policy, Product, Make, Model, Engine Number, Chassis Number,
     Premium, Payment Status, Issued Date, User).

4. Search & Print Policy
   - GET /api/policies/search?engineNo=&chassisNo=&tcNo=&policyNo= —
     returns matching policies (Policy Number, Make, Model, Chassis,
     Engine, Premium, Status, Issued On). At least one search parameter
     required.
   - Reuse the certificate endpoints from Prompt 3 for the Print action.

5. Policy Cancel Upload
   - POST /api/policies/cancel-upload — accepts an Excel file of policies
     to cancel, validates rows against POLICY_MASTER, updates
     POLICY_MASTER.STATUS to CANCELLED for matched valid rows, and
     returns a per-row result list (cancelled vs. rejected with reason).

6. Audit Logging
   - Add an action filter or middleware that writes an AUDIT_LOG row
     (ENTITY_NAME, ACTION, REF_ID, USER_ID, TIMESTAMP) for every
     state-changing endpoint added across Prompts 3–4 (upload, process,
     payment, policy generation, print, cancel). Don't audit read-only
     GETs.

7. Verify every endpoint added in this prompt with a real HTTP call
   against seeded/generated data, and fix anything broken.

8. Update README.md's endpoint table and progress checklist.

9. Commit in logical chunks (batch summary, CD balance, reports, search &
   print, policy cancel, audit logging) and push to origin main. Nothing
   uncommitted at the end. This closes out MotorPortalAPI's functional
   scope — do a final pass confirming Swagger lists every endpoint from
   Prompts 2–4 correctly.
```

---

## PROMPT 5 — Frontend scaffolding: Angular app, layout, auth

```
Work in MotorPortalWEB. Re-read CLAUDE.md first.

GOAL — stand up the Angular application shell, styled to match the
reference Motor Portal screenshots: dark navy body background, an orange
gradient top navbar, white rounded content cards, orange solid-fill
action buttons, clean bordered tables, status badges. Do NOT copy any
proprietary branding — generic "Motor Portal" branding only.

1. Generate a new Angular app (latest stable Angular, standalone
   components + Angular Router, strict mode). Every component MUST have
   separate .ts / .html / .css files — no inline templates or styles
   anywhere.

2. Folder structure under src/app/:
   core/       (guards/, interceptors/, services/, models/)
   shared/     (components/, directives/, pipes/)
   layout/     (header/, sidebar/, footer/)
   features/
     authentication/
     dashboard/
     excel-upload/
     batch-summary/
     batch-processing/
     invalid-records/
     reports/
     policy-search/
     policy-certificate/
     policy-cancel/           (add this — used in Prompt 8)

3. Global theme: a single `_theme.css` (or CSS variables in styles.css)
   defining the navy/orange/white palette and reusing it everywhere —
   don't hardcode hex values per component. Base it on: navy #00305B
   background, orange gradient #EE7B2E→#C1402C for the top navbar, solid
   accent orange #EC6608 for buttons/panels, white cards with soft
   shadows, green for success states, salmon/red for error states.
   Mobile-responsive down to ~375px width throughout.

4. Core services: AuthService (login, logout, token storage per SPA best
   practice — not localStorage for the raw JWT if you can avoid it;
   justify your choice in a code comment), AuthGuard, an HTTP interceptor
   that attaches the bearer token and handles 401 by redirecting to
   login. A typed ApiService/environment.ts pointing at the
   MotorPortalAPI base URL (http://localhost:5xxx by default, override
   via environment files).

5. Login page (features/authentication/login/): Motor Portal
   title/logo, Login ID, Password, Sign In, Forgot Password (can be a
   non-functional link for now), validation messages, a loading
   indicator. On success, store the JWT, redirect to /dashboard, and show
   "Welcome, {username}" in the header.

6. Layout shell: header (Motor Portal brand, Welcome {username}, live
   date/time, Logout) and sidebar (Dashboard, Excel Upload, Batch
   Summary, Reports, Search & Print, Policy Cancel), both behind
   AuthGuard, wrapping a router-outlet.

7. Wire routing for all feature areas listed in step 2 (placeholder
   "coming in next phase" components are fine ONLY for
   dashboard/excel-upload/batch-summary/etc. bodies at this stage — the
   login/layout/auth plumbing itself must be fully real and working, not
   a placeholder).

8. Run the app, log in against the real MotorPortalAPI (from Prompts
   2–4) using the seeded admin/admin123 account, and confirm the
   redirect/guard/interceptor chain genuinely works end to end. Fix
   anything broken.

9. Update README.md: how to configure environment.ts's API base URL, how
   to run (`ng serve`), and tick off this phase's checklist.

10. Commit in logical chunks ("feat: angular app shell and theme",
    "feat: auth service, guard, interceptor, login page", "feat: layout
    header/sidebar and routing skeleton") and push to origin main.
    Nothing uncommitted at the end.
```

---

## PROMPT 6 — Frontend: dashboard, CD balance, excel upload, batch summary, invalid records

```
Work in MotorPortalWEB. Re-read CLAUDE.md first.

GOAL — build out the first real working feature pages, wired to
MotorPortalAPI, matching the reference screenshots' look (two large
orange selection panels with radio buttons for Dashboard; a bordered
"Select a file to upload" row with a Browse button for Excel Upload; an
orange-header data table for Batch Summary).

1. Dashboard (features/dashboard/)
   - Product selection: Motor Class-E / Motor Class-F / Eicher Motor
     (radio group in an orange panel).
   - Process selection: Motor Excel Upload / Motor Batch Summary / Motor
     Report / Search & Print Policy / Policy Cancel Upload (radio group
     in a second orange panel).
   - Both mandatory; Submit navigates to the matching feature route,
     carrying the selected ProductId as route/query state.
   - "View CD Balance" side drawer: Master Policy dropdown (from
     GET /api/master-policies), Customer No + CDBG No auto-filled from
     the selection, Submit, and the live CD_BALANCE from the API.

2. Excel Upload (features/excel-upload/)
   - Shows the selected product context, a file input (.xlsx/.xls) with
     Browse button and selected filename display, Upload button, "Click
     here to download sample Excel file" (calls the sample-template
     endpoint), upload progress indicator, and success/error messaging
     matching the reference ("Record N uploaded successfully. Batch ID :
     XXXXX").
   - After a successful upload, show a "Process current batch…" action
     that calls POST /api/batches/{id}/process and then routes to Batch
     Summary (or shows inline live status — your call, but it must
     reflect real API state, not a client-side simulation).

3. Batch Summary (features/batch-summary/)
   - Transaction From/To Date filters.
   - Counter cards: Total Records, Valid Records, Invalid Records,
     Pending Batch Processing, Payment Pending, Payment Processed —
     from GET /api/batches/summary-counters.
   - Grid: Batch ID, Total Records, Valid Records, Invalid Records,
     Pending Processing, Payment Pending, Payment Processed, Status,
     Created On, Actions (Process Batch, Reason for Invalid, Payment
     Processing, Bulk Print) — each action calls its real endpoint from
     Prompts 3–4.

4. Invalid Records (features/invalid-records/)
   - Table: Transit Date, Invoice Number, Engine Number, Chassis Number,
     Error Remarks, sourced from GET /api/batches/{id}/invalid-records.
   - "Remove All Invalid Entries" button with a confirmation dialog
     before calling DELETE /api/batches/{id}/invalid-records; refresh the
     batch summary counters after.

5. Every page above must handle and display real API error responses
   (e.g. insufficient CD balance, validation failures) in the UI, not
   just in the console.

6. Run the full flow yourself end to end against the real backend:
   dashboard → upload → process → batch summary → invalid records
   cleanup, and fix anything broken.

7. Update README.md and tick off this phase's checklist.

8. Commit in logical chunks (dashboard + CD balance, excel upload, batch
   summary, invalid records) and push to origin main. Nothing
   uncommitted at the end.
```

---

## PROMPT 7 — Frontend: batch processing status, proposal/payment/policy, certificate, bulk print

```
Work in MotorPortalWEB. Re-read CLAUDE.md first.

GOAL — surface the rest of the batch lifecycle (premium → GST → proposal
→ payment → policy → certificate) in the UI, polling real backend state.

1. Batch Processing view (features/batch-processing/)
   - A pipeline visual (six user-facing stages: Excel Upload, Validation,
     Premium Calculation, GST Rate, Proposal Tag, Payment Tag) that polls
     GET /api/batches/{id}/status and reflects genuine per-stage counts
     and the current BATCH_MASTER.STATUS — this must be real polled data,
     not a client-side timer simulation.
   - Trigger actions for "Tag Payments" (POST /api/batches/{id}/payments)
     and show per-case results, including any insufficient-CD-balance
     failures with a clear message, without blocking the cases that
     succeeded.

2. Policy Certificate (features/policy-certificate/)
   - Display Policy Number, Master Policy, Make, Model, Engine Number,
     Chassis Number, Premium, Issued Date for a given policy.
   - Print (opens the PDF from GET /api/policies/{id}/certificate) and
     Download Certificate buttons.

3. Bulk Print — from the Batch Summary grid's "Bulk Print" action, call
   POST /api/batches/{id}/bulk-print and show a summary of certificates
   generated, with links to each.

4. Run the complete lifecycle yourself end-to-end through the UI —
   upload → process → clear invalids → tag payments → view/print
   certificates → bulk print — against the real backend, and fix
   anything broken.

5. Update README.md and tick off this phase's checklist.

6. Commit in logical chunks (batch processing view, certificate view,
   bulk print) and push to origin main. Nothing uncommitted at the end.
```

---

## PROMPT 8 — Frontend: reports, search & print, policy cancel, final UI polish

```
Work in MotorPortalWEB. Re-read CLAUDE.md first.

GOAL — finish the remaining pages and do a full responsive/visual polish
pass across the whole app.

1. Reports (features/reports/)
   - Report Type (Policy Issue Report), Transaction From/To Date, Export
     Report button that calls POST /api/reports/policy-issue and
     downloads the returned .xlsx.

2. Search & Print Policy (features/policy-search/)
   - Search by Engine Number, Chassis Number, TC Number, or Policy
     Number against GET /api/policies/search. Results grid: Policy
     Number, Make, Model, Chassis, Engine, Premium, Status, Issued On,
     each with a Print button reusing the certificate view/endpoint.

3. Policy Cancel Upload (features/policy-cancel/)
   - Excel upload UI (reuse the shared upload component from Excel
     Upload where sensible) posting to
     POST /api/policies/cancel-upload, showing a per-row result list of
     cancelled vs. rejected records with reasons.

4. Full responsive + visual QA pass across every page built in Prompts
   5–8: verify at desktop and ~375px mobile width, fix overflow, wrapping,
   contrast, and spacing issues; confirm every button/action styling is
   consistent with the navy/orange/white theme; confirm loading and empty
   states exist for every data grid (no blank screens while data loads).

5. Update README.md with a full feature list and screenshots section
   placeholder (list the pages; actual screenshots can be added manually
   later), and tick off this phase's checklist — this should complete
   MotorPortalWEB's functional scope.

6. Commit in logical chunks (reports, search & print, policy cancel,
   final polish pass) and push to origin main. Nothing uncommitted at
   the end.
```

---

## PROMPT 9 — Full integration pass and end-to-end smoke test

```
You now have MotorPortalDB, MotorPortalAPI, and MotorPortalWEB fully
built. Re-read each repo's CLAUDE.md.

GOAL — prove the whole system actually works together, end to end, and
fix anything that doesn't.

1. Bring up all three layers locally: Postgres with the MotorPortalDB
   schema/seed data applied, MotorPortalAPI running against it, and
   MotorPortalWEB running against MotorPortalAPI (fix CORS/env/base-URL
   mismatches if any).

2. Walk the ENTIRE real user journey through the running UI, driven by
   real backend calls (no shortcuts): log in as admin/admin123 → pick
   Motor Class-E + Motor Excel Upload on the dashboard → upload a batch
   → process it → clean up invalid records → tag payments (including at
   least one case that deliberately hits an insufficient-CD-balance
   master policy from the seed data, to prove that failure path works) →
   view/print a certificate → bulk print → check it in Batch Summary →
   search & print a policy by engine number → export a Policy Issue
   Report → cancel a policy via Policy Cancel Upload. Fix every bug you
   hit along the way, in whichever repo/layer it actually belongs to.

3. Write a short smoke-test script or checklist (wherever makes sense —
   MotorPortalDOC is fine) capturing these steps so they can be re-run
   after future changes.

4. In each of the three repos, update README.md with a final "Known
   limitations / not yet implemented" section if anything genuinely
   couldn't be completed, and tick off any remaining checklist items now
   proven working end to end.

5. Commit and push in each repo that had fixes, with clear messages
   describing what integration bug was fixed and where. Nothing
   uncommitted anywhere at the end of this phase.
```

---

## PROMPT 10 — MotorPortalDOC: architecture, ER diagram, API reference, setup guide, final READMEs

```
Work in MotorPortalDOC. Re-read CLAUDE.md first.

GOAL — produce the documentation set for the whole system, and put a
final, polished README.md in all four repos.

1. In MotorPortalDOC, create:
   docs/
     architecture.md      — component diagram (Mermaid) of
                             WEB → API → DB, plus the 4-repo layout and
                             why it's split that way.
     er-diagram.md         — a Mermaid ER diagram of the 15 entities and
                             their relationships/cardinalities, matching
                             MotorPortalDB exactly (source of truth: the
                             SQL in MotorPortalDB and the original ER
                             Diagram & Data Dictionary).
     batch-lifecycle.md    — the 6 user-facing stages and the 9-state
                             backend lifecycle (UPLOADED → ... →
                             PRINTED), with the INVALID branch explained.
     api-reference.md      — every endpoint added in Prompts 2–4 (method,
                             path, request/response shape, auth
                             requirement), generated by reading
                             MotorPortalAPI's Swagger/controllers, not
                             invented.
     setup-guide.md        — from-zero instructions: clone all 4 repos,
                             stand up Postgres + run MotorPortalDB's
                             migrate.sh, configure and run MotorPortalAPI,
                             configure and run MotorPortalWEB, seeded
                             login credentials, and the smoke-test
                             checklist from Prompt 9.
     changelog.md          — a dated summary of what each build phase
                             (Prompts 0–9) delivered, in plain language.

2. Write a top-level README.md for MotorPortalDOC indexing all of the
   above.

3. Go back into MotorPortalAPI, MotorPortalWEB, and MotorPortalDB and
   write each a FINAL polished README.md: purpose, tech stack, folder
   structure, how to run locally, how it fits into the 4-repo system
   (link to MotorPortalDOC), and a completed progress checklist — replace
   the bootstrap-era README from Prompt 0 rather than just appending to
   it.

4. Commit and push MotorPortalDOC. Then commit and push the README
   updates in the other three repos from within their own folders.

5. Give me a final summary: the 4 repo URLs, how to run the whole system
   from scratch, and anything you flagged as a known limitation.
```

---

## Notes on the git/GitHub automation

- Every prompt above ends by requiring commits + push before stopping —
  that's intentional so you get a clean, readable commit history per
  repo instead of one giant commit, and so nothing is ever left
  half-done between sessions.
- If you'd rather review diffs before they're pushed, drop `--dangerously-
  skip-permissions` and instead ask Claude Code to stop right before each
  `git push` for your review — say so explicitly in Prompt 0 and it will
  carry that habit through the rest of the pack via CLAUDE.md.
- If your GitHub org enforces branch protection on `main`, tell Claude
  Code in Prompt 0 to work on a `dev` branch and open PRs instead of
  pushing directly — it's a one-line change to the ground rules.
