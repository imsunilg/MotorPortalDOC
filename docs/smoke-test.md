# Motor Portal — End-to-End Smoke Test Checklist

Manual checklist to re-verify the whole system (DB + API + WEB) after future
changes. Walk it top to bottom through the real running UI in a browser —
no direct API calls except where noted for test-data prep. Expect ~15-20
minutes for a full pass.

## 0. Bring the stack up

- [ ] PostgreSQL 16 service `postgresql-x64-16` is running and
      `motorportal` is reachable:
      `PGPASSWORD=<pwd> psql -h localhost -U postgres -d motorportal -c "SET search_path TO \"SGInsurance\"; SELECT count(*) FROM user_master;"`
      → returns a row count > 0.
- [ ] API: `dotnet run --project MotorPortal.API` (default `http://localhost:5287`).
      `GET /api/health` → `{"status":"healthy","database":"connected",...}`.
- [ ] WEB: `ng serve` (default `http://localhost:4200`).

## a. Login

- [ ] Go to `http://localhost:4200/login`.
- [ ] Enter Login ID `admin`, Password `admin123`, click **Sign In**.
- [ ] Expect redirect to `/dashboard`.

## b. Dashboard — select product + process

- [ ] On Dashboard, select **Motor Class-E** under "Select Product".
- [ ] Select **Motor Excel Upload** under "Select Process".
- [ ] Click **Submit**.
- [ ] Expect navigation to `/excel-upload?productId=1`.

## c. Upload a batch

- [ ] Prepare a small `.xlsx` (5 rows) with header row exactly:
      `MASTER_POLICY_NO | ENGINE_NO | CHASSIS_NO | TC_NO | INVOICE_NO | TRANSIT_DATE | MAKE | MODEL`
      (use the "Download sample Excel file" button on this page as a
      starting template). Use real seeded master policies for CLASS_E
      (e.g. `DL-3010/A/1485551`, `DL-3010/A/1485552`), fresh unique
      engine/chassis numbers, and include one row with an unknown master
      policy number to exercise the invalid-record path.
- [ ] Click **Browse...**, pick the file, then click **Upload**.
- [ ] Expect a green banner: "Record N uploaded successfully. Batch ID : &lt;id&gt;".

## d. Process the batch

- [ ] Click **Process current batch...** on the same page.
- [ ] Expect "Batch processed. Status: PROPOSAL_CREATED" with Valid /
      Invalid / Premium Calculated / Proposals Created counts matching
      the file (e.g. 4 valid, 1 invalid for a 5-row file with 1 bad row).
- [ ] Click **Go to Batch Summary**.

## e. Invalid Records — clear

- [ ] On Batch Summary, find **the row for your batch id** (batch ids are
      not necessarily sorted/last — match the Batch ID column exactly)
      and click **Reason for Invalid** in that row.
- [ ] Expect the invalid-records table to show your bad row with an
      `ERROR_REMARKS` reason (e.g. "Master policy not found").
- [ ] Click **Remove All Invalid Entries**, confirm in the dialog.
- [ ] Expect "Invalid entries removed successfully." and the table to
      become empty.

## f. Tag payments (including insufficient CD-balance case)

- [ ] Before this run, check the CD balance of master policy
      `DL-3010/A/1485553` via psql — it is seeded low. Include a batch
      row against this master policy; the product's configured premium
      (see `appsettings.json` → `PremiumRules`) should exceed its balance.
- [ ] From Batch Summary, click **View Processing** on your batch's row
      (or **Payment Processing** directly from Batch Summary for a
      one-line result).
- [ ] On Batch Processing, click **Tag Payments**.
- [ ] Expect a per-proposal results table: rows against policies with
      sufficient CD balance show **Success**; the row against
      `DL-3010/A/1485553` shows **Failed** with message
      `Insufficient CD balance for master policy <policy no> (balance X, required Y)`.
- [ ] Confirm the failure does **not** block the other rows — succeeded
      count + failed count should equal the valid row count, and
      `policiesCreated` should equal the succeeded count.

## g. View / print a policy certificate

- [ ] Go to **Search & Print Policy**, enter one of the engine numbers
      from a row that succeeded in step (f), click **Search**.
- [ ] In the result row, click **Print** → navigates to
      `/policy-certificate/<policyId>` showing policy number, make,
      model, engine/chassis, premium, issued date.
- [ ] Click **Download Certificate**. Expect a real file download whose
      first bytes are `%PDF` and whose size is a few KB (not empty/0
      bytes, not an HTML error page).

## h. Bulk print the batch

- [ ] Back on Batch Summary, find your batch's row and click **Bulk Print**.
- [ ] Expect "Certificates Generated — Batch &lt;id&gt;" listing one
      policy number per successfully-paid row (failed/insufficient-CD
      rows are excluded).

## i. Confirm batch reaches PRINTED

- [ ] In the same Batch Summary table, your batch's **Status** badge
      should now read `PRINTED`, with `Payment Processed` equal to the
      number of certificates generated and `Payment Pending` equal to
      the number of failed (e.g. insufficient-CD) rows.

## j. Search & Print by engine number

- [ ] Go to **Search & Print Policy**, search by a different engine
      number issued in this run (or an earlier seeded/test policy).
- [ ] Expect a result row with matching engine/chassis/policy number.
- [ ] Click **Print** → confirm it navigates to that policy's
      `/policy-certificate/<policyId>` page with matching details.

## k. Export Policy Issue Report

- [ ] Go to **Reports**. Set From Date and To Date to cover today.
- [ ] Click **Export Report**.
- [ ] Expect a real, non-empty `.xlsx` file download (named like
      `policy-issue-report-<from>-to-<to>.xlsx`) and a "Report downloaded
      successfully." confirmation.

## l. Policy Cancel Upload

- [ ] Prepare a `.xlsx` with a single column header `POLICY_NO` and one
      row containing a policy number issued in this run (or any active
      policy).
- [ ] Go to **Policy Cancel Upload**, browse to the file, click
      **Upload & Cancel**.
- [ ] Expect it to appear under **Cancelled (1)** with 0 rejected.
- [ ] Click **Reset**, browse to the **same file again**, click
      **Upload & Cancel** a second time.
- [ ] Expect **Cancelled (0)** / **Rejected (1)** with reason
      "Policy already cancelled" — confirming idempotent re-upload
      handling.

---

## Known issues found and fixed during the last full pass (2026-09-15)

1. **MotorPortalDB** — `sp_tag_payment` raised
   `Insufficient CD balance for master policy id <numeric id>`, exposing
   an internal surrogate key instead of the human-readable master policy
   number operators actually work with. Fixed to read and report
   `MASTER_POLICY_NO` (plus the balance/required amounts) instead.
   File: `scripts/04_functions/06_sp_tag_payment.sql`.

2. **MotorPortalWEB** — `excel-upload.ts` and `policy-cancel.ts` never
   cleared the native `<input type="file">`'s value after handling a
   selection. Re-selecting the *exact same file* a second time (a very
   plausible flow — e.g. step (l) above, or retrying a failed upload)
   does not fire the browser's `change` event when the file path is
   unchanged, so the component's file signal stays null and the
   upload/cancel button stays disabled with no visible feedback. Fixed
   by clearing `input.value = ''` at the end of `onFileSelected()` in
   both components, so the same file can always be re-selected.
   Files: `src/app/features/excel-upload/excel-upload.ts`,
   `src/app/features/policy-cancel/policy-cancel.ts`.

Re-run steps (f) and (l) above after any change to payment tagging or
file-upload components to confirm these stay fixed.
