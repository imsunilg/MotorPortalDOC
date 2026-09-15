# Motor Portal — Batch Lifecycle

Every uploaded batch moves through two parallel views of the same
pipeline: a 6-stage view the operator sees on screen, and a 9-state string
enum enforced strictly, one step at a time, by the database.

## 1. The 6 user-facing stages

Rendered as a tile pipeline on the Batch Processing screen
(`MotorPortalWEB/src/app/features/batch-processing`), each tile's
pending/active/done state is derived by polling `GET /api/batches/{id}/status`
every 2.5 seconds and comparing the real `status` string against this order:

1. **Excel Upload** — operator uploads a `.xlsx` (≤ 200 rows) via
   `POST /api/batches/upload`.
2. **Validation** — every row is checked (master policy exists, required
   fields present); invalid rows are isolated (see §3), valid rows continue.
3. **Premium Calculation** — `fn_calculate_net_premium` computes
   `net = base + addon - discount` per product's configured premium rule.
4. **GST Rate** — `fn_calculate_gst` applies the configured GST rate
   (18% by default, from `appsettings.json`) to produce `gst_amount` and
   `final_premium`.
5. **Proposal Tag** — a `proposal_master` row is created per valid case,
   with a generated, unique proposal number (`fn_generate_proposal_no`).
6. **Payment Tag** — `POST /api/batches/{id}/payments` tags payment per
   proposal against its master policy's CD balance (`sp_tag_payment`);
   on success this also creates the `policy_master` row and, later,
   the certificate.

Certificate generation and bulk print (`POST /api/batches/{id}/bulk-print`)
happen after stage 6 completes and are shown as separate actions rather
than pipeline tiles, but they consume the same `status` value to decide
whether printing is possible.

## 2. The real 9-state backend lifecycle

Enforced by `sp_advance_batch_status(batch_id, new_status)` in
`MotorPortalDB/scripts/04_functions/07_sp_advance_batch_status.sql`, stored
in `batch_master.status` (`varchar(25)`):

```
UPLOADED → VALIDATED → PREMIUM_CALCULATED → GST_CALCULATED →
PROPOSAL_CREATED → PAYMENT_PENDING → PAYMENT_PROCESSED →
POLICY_CREATED → PRINTED
```

The procedure holds the array of these 9 values in strict order and enforces
transitions with no skipping and no going backward:

```sql
v_order := ARRAY['UPLOADED','VALIDATED','PREMIUM_CALCULATED','GST_CALCULATED',
                  'PROPOSAL_CREATED','PAYMENT_PENDING','PAYMENT_PROCESSED',
                  'POLICY_CREATED','PRINTED'];
...
IF v_new_idx <> v_cur_idx + 1 THEN
    RAISE EXCEPTION 'Invalid batch status transition from % to %', v_current, p_new_status;
END IF;
```

This was verified directly during DB testing: attempting the out-of-order
jump `VALIDATED → PRINTED` is rejected by the procedure. Only "advance by
exactly one step" is legal — there is no API path that skips a stage, and
the row is locked (`FOR UPDATE`) while the check runs, so two concurrent
advance attempts on the same batch cannot race each other into an invalid
state.

### Mapping stage → status

| User-facing stage | `batch_master.status` reached | Driven by |
|---|---|---|
| Excel Upload | `UPLOADED` | `POST /api/batches/upload` |
| Validation | `VALIDATED` | `sp_process_batch_validation`, called from `POST /api/batches/{id}/process` |
| Premium Calculation | `PREMIUM_CALCULATED` | `ProcessBatchAsync`, same call |
| GST Rate | `GST_CALCULATED` | `ProcessBatchAsync`, same call |
| Proposal Tag | `PROPOSAL_CREATED` | `ProcessBatchAsync`, same call |
| Payment Tag | `PAYMENT_PENDING` → `PAYMENT_PROCESSED` | `POST /api/batches/{id}/payments` |
| (policy issuance, not a separate operator tile) | `POLICY_CREATED` | same call, once all payable proposals are processed |
| (print, separate action) | `PRINTED` | `POST /api/batches/{id}/bulk-print` |

Note `POST /api/batches/{id}/process` alone advances the batch through four
statuses in one call (`VALIDATED → PREMIUM_CALCULATED → GST_CALCULATED →
PROPOSAL_CREATED`) — the six-tile UI shows this as four sequential tiles
completing together because the underlying HTTP call is a single
orchestrated operation, not four separate round trips.

## 3. The INVALID branch — per-record, isolated, non-blocking

There is **no** `INVALID` value in `batch_master.status` — invalidity is
never a batch-level state. Instead:

- Every row of the uploaded Excel becomes one `batch_detail` row with its
  own `record_status` (`PENDING` initially).
- `sp_process_batch_validation(batch_id)` checks each `batch_detail` row
  independently. A row that fails validation (e.g. its `master_policy_no`
  doesn't exist) gets one `invalid_records` row (with a human-readable
  `error_remarks`, e.g. "Master policy not found") — the `batch_detail`
  row itself is left in place, not deleted.
- `batch_master.valid_records` / `invalid_records` counters are updated to
  reflect the split, and the batch still advances to `VALIDATED` as long as
  the procedure runs to completion — one bad row never prevents the other
  199 from validating.
- Every downstream stage (premium, GST, proposal, payment) only ever
  operates on rows with `record_status = 'VALID'` (or the equivalent
  proposal/payment records derived from them) — invalid rows are simply
  never picked up again, they are not retried or auto-corrected.
- The operator can inspect (`GET /api/batches/{id}/invalid-records`) and
  clear (`DELETE /api/batches/{id}/invalid-records`) the invalid set at any
  time without touching the valid rows or the batch's overall status.
- This was confirmed end-to-end in the final integration pass: a 5-row
  batch with 1 intentionally-bad row (unknown master policy) produced
  4 valid + 1 invalid, the batch reached `PROPOSAL_CREATED` with exactly 4
  proposals, and clearing the invalid record left the valid pipeline
  completely unaffected.

This is also why `docs/er-diagram.md` models `invalid_records` as a sibling
of `batch_detail` under `batch_master`, not as a status the whole batch can
be "stuck" in.
