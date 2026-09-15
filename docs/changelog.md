# Motor Portal — Changelog

Plain-language, dated summary of what each build phase delivered, across
all four repos. All phases landed in a single build day —
**2026-09-15** — per each repo's actual commit history at the time this
document was written.

## 2026-09-15 — MotorPortalDB: schema + seed data

Built the database from scratch: all 15 core tables (`user_master` through
`audit_log`) with primary/foreign keys, the FK and search indexes, the
PL/pgSQL functions and procedures that carry the system's core business
rules (`fn_calculate_net_premium`, `fn_calculate_gst`,
`fn_generate_proposal_no`, `fn_generate_policy_no`,
`sp_process_batch_validation`, `sp_tag_payment`,
`sp_advance_batch_status`), the `vw_policy_issue_report` reporting view,
and seed data (admin user, 3 products, 5 functions, 10 master policies, 3
sample batches). `migrate.sh` / `rollback.sql` were exercised end-to-end.

## 2026-09-15 — MotorPortalAPI: core scaffolding

Stood up the 4-project .NET 8 solution structure (API / Application /
Domain / Infrastructure), mapped every entity via EF Core Fluent API to the
existing lowercase-snake-case `SGInsurance` schema (database-first, no EF
migrations run against it), added JWT authentication and the login
endpoint, Swagger/OpenAPI, CORS, a real database-round-trip health check,
and global exception-handling middleware with structured Serilog logging.

## 2026-09-15 — MotorPortalAPI: core business logic

Delivered the heart of the pipeline: Excel batch upload (ClosedXML parsing,
structural validation, sample template download), the
validate → premium → GST → proposal orchestration
(`POST /api/batches/{id}/process`, calling the DB-side functions/procedures
rather than reimplementing the math in C#), payment tagging against CD
balance with the mocked PF gateway (`MockPfService` /
`IPfGatewayService`), policy generation, QuestPDF certificate generation,
and bulk print.

## 2026-09-15 — MotorPortalAPI: remaining modules

Added batch summary (`GET /api/batches`, `GET /api/batches/summary-counters`)
and live CD-balance lookup, the Policy Issue Report export
(`POST /api/reports/policy-issue`, streaming a real `.xlsx` built from
`vw_policy_issue_report`), policy search
(`GET /api/policies/search`), and policy cancel upload
(`POST /api/policies/cancel-upload`, per-row cancel/reject with audit
logging). This completed the full documented endpoint surface (see
`docs/api-reference.md`).

## 2026-09-15 — MotorPortalWEB: app shell

Scaffolded the Angular 22 standalone-component app shell and navy/orange
theme, then built authentication (`AuthService`, `authGuard`,
`authInterceptor` — JWT stored in `sessionStorage`, not `localStorage`),
the login page, and the layout shell (header + sidebar navigation around a
routed outlet).

## 2026-09-15 — MotorPortalWEB: dashboard, upload, batch summary, invalid records

Built the Dashboard (mandatory product + process selection, CD Balance
drawer), Excel Upload (upload + process kickoff + sample template
download), Batch Summary (filterable grid, live counters, per-row
process/payment/bulk-print actions), and Invalid Records (per-batch listing
+ bulk clear behind a confirm dialog).

## 2026-09-15 — MotorPortalWEB: batch processing, certificate, bulk print

Built the six-tile live-polling Batch Processing pipeline view (polling
`GET /api/batches/{id}/status` every 2.5s), the Policy Certificate detail
view with authenticated PDF print/download (via `HttpClient` blob fetch +
object URL, since the endpoint requires a Bearer token a plain `<a href>`
can't carry), and the Bulk Print results summary linking generated
certificates back into that view.

## 2026-09-15 — MotorPortalWEB: reports, search, cancel, polish

Built the Reports page (Policy Issue Report export to `.xlsx`), Search &
Print Policy (multi-field search with print-to-certificate), and Policy
Cancel Upload (bulk cancellation with a two-panel cancelled/rejected
result). Closed out with a responsive/visual-polish pass: fixed
low-contrast page titles across 9 stylesheets, one hardcoded hex color,
and two header/page-header overflow risks at narrow (~375px) widths.

## 2026-09-15 — Full cross-repo integration pass

Drove the entire journey end to end through the real running app in a
headless Chromium browser (Playwright), against the real API and
PostgreSQL database, at both 1280px and 375px viewports: login → dashboard
→ Excel upload → batch processing → invalid records → payment tagging
(including a real insufficient-CD-balance case) → certificate view/download
→ bulk print → search & print → report export → policy cancel +
re-upload rejection.

**Two real bugs were found and fixed:**

1. **MotorPortalDB** — `sp_tag_payment` raised
   `Insufficient CD balance for master policy id <numeric id>`, leaking an
   internal surrogate key instead of the human-readable master policy
   number operators actually work with. Fixed in
   `scripts/04_functions/06_sp_tag_payment.sql` to report
   `MASTER_POLICY_NO` plus the balance/required amounts, applied live via
   `CREATE OR REPLACE PROCEDURE` without a destructive migration.
2. **MotorPortalWEB** — `excel-upload.ts` and `policy-cancel.ts` never
   cleared the native `<input type="file">`'s value after handling a
   selection, so re-selecting the exact same file a second time (e.g.
   retrying a failed upload, or the policy-cancel re-upload test) silently
   failed to fire the browser's `change` event. Fixed by resetting
   `input.value = ''` at the end of `onFileSelected()` in both components.

Also fixed during this window, in MotorPortalAPI: a naming inconsistency
between the two batch-summary endpoints (`summary-counters` used
`pendingBatchProcessing` while `/batches` used `pendingProcessing`) —
aligned to `pendingProcessing` on both.

No other bugs were found. The PF (payment facilitator) confirmation remains
a simulated `MockPfService`, which is expected and by design for this
environment.

## 2026-09-15 — Documentation phase (this pass)

Authored `docs/architecture.md`, `docs/er-diagram.md`,
`docs/batch-lifecycle.md`, `docs/api-reference.md`, `docs/setup-guide.md`
and this changelog in MotorPortalDOC, all verified directly against the
real code in MotorPortalDB/API/WEB rather than the original reference
design docs. Replaced MotorPortalDOC's placeholder `README.md` with a full
index, and finalized the READMEs in MotorPortalAPI, MotorPortalWEB and
MotorPortalDB into single polished documents (condensing their
phase-by-phase build logs while preserving load-bearing detail: exact
verification `curl` commands, known-limitations notes, the ER diagram and
function list, and the auth-flow explanation).
